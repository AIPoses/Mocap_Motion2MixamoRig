# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

## [0.1.5] - 2026-10-15

### Added

- Blender 4.2+ Extension that launches `m2mr` in an external Python environment, streams progress, and imports the generated GLB.
- `m2mr run --output-dir`, `--progress-jsonl`, and `--no-preview` for add-on job control (existing CLI flags unchanged).
- README translations in 17 languages, under `docs/i18n/`.
- Python, Blender, and GitHub stars badges, and a star history chart in the English README.

### Changed

- Python 3.11+ is now required, matching `gvhmr>=1.6`.
- Translated READMEs, the poster, and the demo video moved into `docs/`.

## [0.1.0] - 2026-09-04

### Added

- Retarget single-person motion videos to Mixamo-rigged characters.
- Recover and transfer a static 3D pose from a single-person image.
- Export reusable human skeleton motion as `skeleton_motion.npz`.
- Export per-bone Mixamo rotations as `mixamo_rotations.npz`.
- Export animated, skinned Mixamo characters as `.glb`.
- Generate human skeleton, Mixamo skeleton, character, and comparison previews.
- Reuse previously extracted skeleton motion with `--skeleton`.
- Validate dependencies and user-provided assets with `m2mr doctor`.
- Select CPU, CUDA, or Apple Silicon MPS for motion extraction.

[Unreleased]: https://github.com/MaxLiu22/Motion2MixamoRig/compare/v0.1.5...HEAD
[0.1.5]: https://github.com/MaxLiu22/Motion2MixamoRig/compare/v0.1.0...v0.1.5
[0.1.0]: https://github.com/MaxLiu22/Motion2MixamoRig/releases/tag/v0.1.0
