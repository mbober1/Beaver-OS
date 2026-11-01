---
name: bitbake-image-build
description: 'Build BeaverOS BitBake images such as lab-image-minimal and lab-image-minimal-update (OTA bundle) in the Yocto build directory. USE FOR: running bitbake, building bitbake image targets, validating metadata with bitbake -p, checking enabled layers, locating build artifacts. DO NOT USE FOR: flashing or testing the board (use luckfox-ssh or rsync-to-board), editing recipe contents without a build request.'
argument-hint: 'Image target, e.g. lab-image-minimal or lab-image-minimal-update'
---

# Build BitBake Image

## When to Use

- The user asks to build an image or recipe with BitBake.
- The user asks to validate Yocto metadata after a recipe, layer, or config change.
- The user asks to produce the OTA update bundle.

## Procedure

1. Run from the workspace root. Initialize the build directory once with `./setup-environment build` only if `build/` does not exist. Rerunning it in an existing build directory appends duplicate settings to `build/conf/local.conf`.
2. In each new shell, source the OE-Core environment from the workspace root:
   ```sh
   source sources/oe-core/oe-init-build-env build
   ```
3. Build the requested target:
   ```sh
   bitbake lab-image-minimal
   ```
   For the OTA bundle, use `bitbake lab-image-minimal-update`.
4. Locate the output directory when the user needs artifacts:
   ```sh
   bitbake -e lab-image-minimal | grep '^DEPLOY_DIR_IMAGE='
   ```

## Decision Points

- Build failure: read the first error in the log, identify the failing recipe, and fix it in the owning layer (see the yocto-recipes instructions). Use `bitbake -k <target>` to continue past independent failures and collect all errors.

## Pitfalls

- `TMPDIR` is set to `/mnt/storage/beaveros`. If a build fails with disk or path errors, confirm the storage mount is available (see `.devcontainer/devcontainer.json`).
- `default.xml` checks out OE-Core at `sources/oe-core`, but `.github/workflows/build-image.yml` sources `sources/poky`. Verify this path when diagnosing CI failures.
- Do not commit generated files under `build/`.

## Completion Checks

- The target build exits 0, or the failing recipe and error are reported to the user.
- Artifact paths are reported from `DEPLOY_DIR_IMAGE` when the user asks for output files.
