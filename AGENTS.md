# BeaverOS Agent Notes

## Build Workflow

From the workspace root, initialize a fresh build directory once, then source the OE-Core environment in each new shell:

```sh
./setup-environment build
source sources/oe-core/oe-init-build-env build
bitbake-layers show-layers
bitbake -p
bitbake lab-image-minimal
```

Build the OTA bundle with `bitbake lab-image-minimal-update`. The repository README documents checkout and image deployment: [README.md](README.md).

## Where Changes Belong

- `sources/meta-beaver/` owns BeaverOS distro configuration, features, and image composition. See [meta-beaver/README.md](sources/meta-beaver/README.md).
- `sources/meta-beaver-bsp/` owns board machine configuration and BSP recipes. See [meta-beaver-bsp/README.md](sources/meta-beaver-bsp/README.md).
- `default.xml` pins the OE/Yocto layer revisions and checkout paths; keep layer assumptions consistent with it.
- `build/conf/` is generated, machine-local build configuration. Avoid treating it as distributable project metadata.

For metadata changes, use `bitbake -p` and inspect enabled layers with `bitbake-layers show-layers` before attempting a full image build. No separate repository-wide test suite is documented; BitBake parsing and the affected recipe or image build are the available checks.

## Pitfalls

- Run `setup-environment` from the workspace root. It appends settings to `build/conf/local.conf`; rerunning it in the same build directory can duplicate configuration.
- The setup script selects `rock-5b-plus` by default. Check the relevant machine configuration and local build settings before changing targets.
- `default.xml` checks out OE-Core at `sources/oe-core`, while [.github/workflows/build-image.yml](.github/workflows/build-image.yml) currently sources `sources/poky`. Verify this path mismatch when diagnosing CI setup failures.
- Builds need substantial disk space. `setup-environment` sets `TMPDIR` to `/mnt/storage/beaveros`; confirm the devcontainer storage mount is available before a full build. See [.devcontainer/devcontainer.json](.devcontainer/devcontainer.json).