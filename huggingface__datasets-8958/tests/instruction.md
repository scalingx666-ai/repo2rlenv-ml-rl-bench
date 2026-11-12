# Issue

**Title:** fix(mesh): accept .gltf in the Mesh feature and the meshfolder builder

## Description

`Mesh` (and the `meshfolder` builder behind it) only accepts `.glb`, `.ply` and `.stl`, although `trimesh` — which does the decoding — also reads glTF. So a folder of `.gltf` models produces no `Mesh` column at all, and `Mesh().decode_example({"path": "model.gltf", ...})` raises.

Measured on `main` (62e0b421), by inferring the module for a folder of data files:

| folder contents | before | after |
|---|---|---|
| 1 `.gltf`, no textures | `DataFilesNotFoundError` | `meshfolder` |
| 2 `.gltf` + 1 `.png` | `imagefolder` | `meshfolder` |
| 3 `.gltf` + 2 `.png` | `imagefolder` | `meshfolder` |
| 1 `.gltf` + 4 `.png` | `imagefolder` | `imagefolder` |

```python
>>> from datasets import Mesh
>>> Mesh().decode_example({"path": "model.gltf", "bytes": None})
ValueError: A mesh path should have a .glb, .ply, or .stl extension.   # before
```

## Changes

- `src/datasets/packaged_modules/meshfolder/meshfolder.py`: add `.gltf` to `MESH_EXTENSIONS`, which feeds both the lowercase and uppercase `_EXTENSION_TO_MODULE` maps.
- `src/datasets/features/mesh.py`: accept `gltf` in `_infer_mesh_file_type`, which gates both the path and the embedded-bytes decode paths; update the two error messages and the docstring.
- `docs/source/about_dataset_features.mdx`: the supported-extension sentence.

`trimesh.load(..., file_type="gltf")` handles a `.gltf` with embedded (data-URI) buffers as well as one whose buffers sit next to it, so the decode path needs nothing else.

## What this does not change

Format inference still picks the most common extension (`.parquet` / `.lance` / … preferred on a tie). A repo where the glTF models are outnumbered by their own textures — the 684 `.gltf` / 4,063 `.glb` / 5,532 image case in #8956 — therefore still infers `imagefolder`. Changing that means deciding that a model extension outranks a texture extension, which is a policy call for the maintainers rather than something to fold into an extension-list fix. Happy to send that separately if you want it.

## Tests

- `tests/features/test_mesh.py::test_mesh_feature_decodes_gltf` — decodes a `.gltf` from a path and from embedded bytes.
- `tests/packaged_modules/test_meshfolder.py::test_gltf_file_is_inferred_as_meshfolder` — a `.gltf` data file infers `meshfolder`.
- `tests/packaged_modules/test_meshfolder.py::test_meshfolder_config_and_extensions` — expected extension list updated.

```
$ pytest tests/features/test_mesh.py tests/packaged_modules/test_meshfolder.py -q
25 passed
```

With `src/` reverted, the same command gives `3 failed, 22 passed` — the three assertions above are the ones that fail.

`ruff check` and `ruff format --check` (repo pin v0.11.8, line-length 119) pass on all changed files.

Not tested: loading a `.gltf` that lives on the Hub (only local paths and embedded bytes were exercised), and the dataset-viewer side, which is a separate service.

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `62e0b421bb99`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.