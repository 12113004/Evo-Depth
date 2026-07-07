# Evo-Depth fork reproduction notes

This fork is used as an independent Evo-Depth reproduction workspace.
The upstream/default README from the fork point is preserved at:

- [`doc/README.upstream.md`](doc/README.upstream.md)

## Reproduction summary

Repository state used for the reproduction:

- Fork: `https://github.com/12113004/Evo-Depth.git`
- Upstream: `https://github.com/MINT-SJTU/Evo-Depth`
- Baseline commit before local reproduction fixes: `a15aeec`
- Date: 2026-07-07

Downloaded checkpoints:

- `MINT-SJTU/EVO-Depth-MetaWorld`
- `MINT-SJTU/EVO-Depth-Arena`
- `MINT-SJTU/EVO-Depth-LIBERO`

Large downloaded weights, benchmark clones, logs, videos, and local orchestration artifacts are intentionally ignored by git. See [`.gitignore`](.gitignore).

## Local changes for reproducible evaluation

The fork contains small runtime fixes needed for stable local evaluation:

- allow disabling FlashAttention with `EVO_DEPTH_USE_FLASH_ATTN=0`, avoiding a local flash-attn compile;
- gate verbose flow-matching action debug prints behind `EVO_DEPTH_DEBUG_ACTION=1`;
- disable websocket ping timeout for long synchronous simulator steps;
- fix LIBERO offscreen render environment variables before importing `OffScreenRenderEnv`;
- add LIBERO client flags:
  - `--no_video`
  - `--task_indices`, e.g. `1-4` or `1,3,5-7`.

All local build/test commands were run with low compile/thread parallelism, e.g.:

```bash
export MAX_JOBS=1
export CMAKE_BUILD_PARALLEL_LEVEL=1
export MAKEFLAGS=-j1
export OMP_NUM_THREADS=1
export MKL_NUM_THREADS=1
export OPENBLAS_NUM_THREADS=1
export NUMEXPR_NUM_THREADS=1
```

## Main reproduction result

The project and websocket inference path were brought up successfully, and LIBERO was evaluated completely.
However, local results did **not** reproduce the paper's reported LIBERO score.

### LIBERO, seed=23

`seed=23` is the default seed in `LIBERO-evaluation/libero_client.py`, so it is the default baseline seed for this fork.

Full 40-task / 400-episode result:

| Suite | Success | Rate |
| --- | ---: | ---: |
| `libero_spatial` | 77/100 | 77% |
| `libero_goal` | 81/100 | 81% |
| `libero_object` | 91/100 | 91% |
| `libero_10` | 61/100 | 61% |

Mean over the four suites: **77.5%**.

### Default success-rate regression baseline

For future quick success-rate regression tests in this fork, use:

- suite: `libero_object`
- seed: `23`
- baseline success rate: **91%** (`91/100`)

This is the default local baseline unless a future commit intentionally changes the evaluation environment, checkpoint, or action/server path.

## Other checks

- MetaWorld partial official-like run stopped at 28/50 tasks because the observed mean was about `27.5%`, far below the paper target.
- VLA-Arena `safety_static_obstacles` smoke test ran successfully, but the full paper VLA-Arena benchmark was not completed.
- LIBERO-Plus assets were downloaded and unpacked, but full LIBERO-Plus evaluation was not completed.

## Evidence locations

The raw local logs are ignored by git and remain machine-local. Main aggregate files from this reproduction were:

- `.omx/logs/libero_seed23_full_aggregate.txt`
- `.omx/logs/libero_combined_012700_plus_102200_aggregate.txt`
- `.omx/reports/evo_depth_reproduction_status_20260707.md`

Because `.omx/` is ignored, copy these files elsewhere before cleaning the workspace if you need to preserve machine-local evidence.
