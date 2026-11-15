---
name: ota-swupdate
description: 'Build a BeaverOS OTA update bundle (lab-image-minimal-update), transfer the .swu to the target board, apply it with SWUpdate, and verify the A/B slot switch after reboot. USE FOR: OTA updates, software update, firmware update, swupdate, swupdate-client, .swu bundle, lab-image-minimal-update, A/B slot switch, rootfsA/rootfsB verification, checking the active boot partition. DO NOT USE FOR: initial flashing of a .wic image, plain image builds without OTA (use bitbake-image-build), kernel or U-Boot development, or editing recipes (see the yocto-recipes instructions).'
argument-hint: 'Image target, default lab-image-minimal-update; optional target host, default radxa-zero-3e-custom.lan'
---

# BeaverOS OTA Update with SWUpdate

End-to-end procedure to ship a running BeaverOS board to the other rootfs slot:
build the bundle, copy it over, trigger SWUpdate, and confirm the switch.

## When to Use

- The user asks to update the board over the air / via OTA.
- The user asks to build or deploy `lab-image-minimal-update`.
- The user asks to verify which A/B slot the board booted from.

Do not use for the first installation: that is a full `.wic` flash.

## Prerequisites

1. OTA must be enabled in the build configuration. `build/conf/local.conf` must
   include `features/ota.inc` (it sets `DISTRO_FEATURES:append = " ota"`, which
   `lab-image-minimal-update` requires via `REQUIRED_DISTRO_FEATURES`).
2. The board must already boot from the A/B layout, i.e. have
   `PARTLABEL=rootfsA` and `PARTLABEL=rootfsB` partitions. A board installed from
   a non-OTA `.wic` cannot be updated this way.
3. `MACHINE` in `local.conf` must match the board being updated.

## Procedure

### 1. Build the update bundle

Build `lab-image-minimal-update` following
[bitbake-image-build](../bitbake-image-build/SKILL.md) (source the OE-Core
environment from the workspace root, then run `bitbake lab-image-minimal-update`).
The build pulls in `lab-image-minimal.rootfs.tar.zst` and packs the
`sw-description`, `update.sh`, and rootfs archive into a `.swu` cpio bundle.

### 2. Locate the artifact

```sh
bitbake -e lab-image-minimal-update | grep '^DEPLOY_DIR_IMAGE='
```

The bundle is the stable symlink in that directory:

```text
${DEPLOY_DIR_IMAGE}/lab-image-minimal-update-<MACHINE>.rootfs.swu
```

Use the symlink, not the timestamped file, so the path stays valid between
builds. Confirm it exists before copying:

```sh
ls -l "${DEPLOY_DIR_IMAGE}"/lab-image-minimal-update-*.rootfs.swu
```

### 3. Preflight the board

Use the SSH conventions from [target-ssh](../target-ssh/SKILL.md). Replace
`<target-host>` with the board host, e.g. `radxa-zero-3e-custom.lan`.

```sh
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking no" \
    root@<target-host> 'findfs PARTLABEL=rootfsA; findfs PARTLABEL=rootfsB; \
    findmnt -no SOURCE /; systemctl is-active swupdate.service'
```

Stop if either `findfs` fails: the board is not A/B-provisioned. Note which
slot is current (`/proc/cmdline` `root=` resolves to rootfsA or rootfsB).

### 4. Transfer the bundle

Copy to `/tmp` on the board:

```sh
rsync -avz --progress \
    -e "ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no" \
    "${DEPLOY_DIR_IMAGE}/lab-image-minimal-update-<MACHINE>.rootfs.swu" \
    root@<target-host>:/tmp/
```

`scp` works the same way if rsync is unavailable on the target.

### 5. Start SWUpdate

The image ships `swupdate.service`, which runs the daemon with
`-H custom-board:1.0 -f /etc/swupdate.cfg -p 'reboot'`. Feed the bundle to that
daemon over its IPC socket with the client tool:

```sh
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking no" \
    root@<target-host> 'systemctl start swupdate.service; \
    swupdate-client -v /tmp/lab-image-minimal-update-<MACHINE>.rootfs.swu'
```

`sw-description` installs the rootfs archive into `/dev/update` (the inactive
slot). `update.sh` `preinst` formats that slot, and `postinst` clears the GPT
`LegacyBIOSBootable` flag on the current partition and sets it on the new one.

Fallback if `swupdate-client` is missing, run standalone (stop the daemon first
so the two processes do not compete for the control socket):

```sh
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking no" \
    root@<target-host> 'systemctl stop swupdate.service; \
    swupdate -v -H custom-board:1.0 -f /etc/swupdate.cfg -i /tmp/lab-image-minimal-update-<MACHINE>.rootfs.swu'
```

Because the service passes `-p reboot`, the board reboots automatically after a
successful update. With the standalone fallback, reboot manually.

### 6. Verify the switch

After the board comes back:

```sh
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking no" \
    root@<target-host> 'cat /proc/cmdline; findmnt -no SOURCE /; \
    sfdisk --part-attrs /dev/mmcblk0 3; sfdisk --part-attrs /dev/mmcblk0 4; \
    journalctl -b -u swupdate --no-pager | tail -20'
```

Check all of the following:

- `/proc/cmdline` `root=` resolves to the slot that was inactive before the
  update (rootfsA <-> rootfsB).
- `findmnt /` shows that partition, mounted read-write.
- `sfdisk --part-attrs` reports `LegacyBIOSBootable` only on the newly active
  partition.
- `journalctl -b -u swupdate` shows the install completed without errors.

## Decision Points

- `bitbake lab-image-minimal-update` fails on `REQUIRED_DISTRO_FEATURES`:
  add `include features/ota.inc` to `build/conf/local.conf` and rebuild.
- `findfs PARTLABEL=rootfsA/B` fails on the board: the board was flashed with a
  single-slot image. Flash the A/B `.wic` first; OTA is not applicable.
- `update.sh` reports it cannot determine `root=` or the update device: check
  `/proc/cmdline` and that the partition table is GPT.
- The board reboots into the same slot: inspect the GPT bootable attribute and
  U-Boot's boot selection (see Pitfalls).

## Pitfalls

- `sfdisk --activate` is MBR-only and silently does nothing on GPT. Slot
  switching uses `--part-attrs ... LegacyBIOSBootable`; do not "fix" `update.sh`
  by switching to `--activate`.
- `CONFIG_ENV_IS_IN_MMC` stores U-Boot's environment on disk. A saved `bootcmd`
  can persist across updates; a full `.wic` reflash or
  `env default -a; saveenv` is needed if boot selection misbehaves.
- Partition numbers vary with layout. Always resolve slots by
  `PARTLABEL=rootfsA`/`rootfsB`, never by a hardcoded `mmcblk0pN`.
- The web UI (`galaktyczny-updater`) serves on port 8080 from `/www`; it is an
  alternative to the CLI but not required for this procedure.
- The overlay `/data` partition is outside the A/B slots and is not touched by
  the update; persistent data survives the switch.

## Completion Checks

- The `.swu` was built and copied from `DEPLOY_DIR_IMAGE`.
- SWUpdate reported success and the board rebooted.
- The board is running from the opposite slot with the bootable attribute set
  on it, and `swupdate` logs are clean for the current boot.
