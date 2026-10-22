---
name: Yocto Recipes
description: "Use when creating or changing Yocto/OpenEmbedded BitBake recipes or recipe append files. Covers layer ownership, metadata, and validation in BeaverOS."
applyTo: ["sources/**/*.bb", "sources/**/*.bbappend"]
---

# Yocto Recipe Guidelines

- Put distro features, application packages, and image composition in `sources/meta-beaver/`; put board- and machine-specific recipes in `sources/meta-beaver-bsp/`.
- Prefer a `.bbappend` in the owning BeaverOS layer to modifying an upstream layer's recipe. Keep recipe files and patches in the recipe's layer, and reference local files through `SRC_URI` using the established layer conventions.
- Follow BitBake metadata conventions and existing nearby recipes. Declare license metadata and checksums accurately; do not invent values to satisfy parsing.
- Keep machine-specific behavior out of distro-wide recipes. Use existing machine overrides and classes where appropriate rather than duplicating recipes.
- Do not commit generated build configuration or build output from `build/` as recipe changes.
- Validate metadata changes with `bitbake -p` from the configured build environment. Then build the affected recipe or relevant image when practical, and report any validation that could not be run.