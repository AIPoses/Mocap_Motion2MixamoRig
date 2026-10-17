# Release checklist

## Repository

- [ ] Working tree is clean
- [ ] Version numbers are consistent: `version` in `pyproject.toml` and in
      `blender_extension/motion2mixamorig_blender/blender_manifest.toml` both
      equal the tag without the `v` (tag `v0.1.6` → `0.1.6`). The add-on zip
      name and the version Blender shows come from the manifest.
- [ ] No gated models, weights, Mixamo assets, or user inputs are tracked
- [ ] README instructions match the current CLI
- [ ] CHANGELOG is updated
- [ ] Release notes are prepared (`.github/RELEASE_NOTES_v<version>.md`)

## Validation

- [ ] Package imports successfully
- [ ] CLI help works
- [ ] `m2mr doctor` runs and reports environment status
- [ ] Video pipeline smoke test completed
- [ ] Image pipeline smoke test completed
- [ ] Generated NPZ files load successfully
- [ ] Generated GLB imports into Blender
- [ ] Blender add-on smoke test ([`blender-extension.md`](blender-extension.md))

## Publishing

- [ ] Release preparation commit is pushed
- [ ] Annotated tag is created
- [ ] Tag is pushed
- [ ] Add-on zip is built: `python scripts/package_blender_extension.py`
      → `dist/motion2mixamorig-<version>.zip`
- [ ] GitHub Release is created from the tag, with the release notes and the
      add-on zip attached
- [ ] Release URL opens successfully, and
      `https://github.com/MaxLiu22/Motion2MixamoRig/releases/latest` points to it
