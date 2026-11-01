---
name: rsync-to-board
description: 'Transfer files and directories from the host to the Luckfox Lyra Plus board (root@luckfox-lyra-plus.lan) using rsync over SSH. USE FOR: copying firmware, kernel modules (.ko), binaries, DTBs, or config files to the board; syncing a local directory to a remote path on the board; resuming or incremental copies. DO NOT USE FOR: running remote commands (use luckfox-ssh), building on the host, or copying from the board to the host unless explicitly asked.'
---

# Transfer Files to Luckfox Lyra Plus with rsync

## Copy Command

Use this base command. It disables host key checking and does not write a known_hosts file, matching the luckfox-ssh skill:

```bash
rsync -avz --progress -e "ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no" <local-src> root@luckfox-lyra-plus.lan:<remote-dest>
```

- A trailing slash on `<local-src>` copies the directory contents. No slash copies the directory itself.
- Add `--dry-run` first when syncing a directory, to preview changes.
- Add `--delete` only when the user asks for the remote directory to mirror the local one. It removes remote files that are not present locally.
- Use `-z` for compression on slow links. Omit it for already-compressed files if speed matters.

## Examples

Copy a kernel module to the board's home directory:

```bash
rsync -avz --progress -e "ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no" rk3506_rproc/rk3506_rproc.ko root@luckfox-lyra-plus.lan:/root/
```

Sync a local directory to a remote path, previewing first:

```bash
rsync -avz --dry-run -e "ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no" ./out/ root@luckfox-lyra-plus.lan:/root/out/
```

## Notes

- Run one rsync command per terminal call and wait for the output before the next.
- Use `run_in_terminal` in sync mode.
- If the board is unreachable, check that it is powered and on the network, and that `luckfox-lyra-plus.lan` resolves (mDNS). Do not change the host key options to work around it.
- If a password is requested, ask the user to type it into the terminal. Never pass passwords through chat or tool arguments.
- Do not write credentials into this file.
