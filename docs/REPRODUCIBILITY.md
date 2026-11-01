# Reproducibility and setup notes

This repository preserves **research code**, rather than a guaranteed one-command reproduction pipeline.

## Provenance

- Course experiment: BIOINF 593, Fall 2025, University of Michigan.
- GET upstream: https://github.com/GET-Foundation/get_model
- Team experiment branch: https://github.com/Yoyomanzoor/get_multimodel/tree/yoyo
- Target dataset: PBMC 10k multiome (see upstream GET data instructions).

## Environment

- PyTorch, CUDA-capable training environment, and the dependencies declared in the inherited GET environment / packaging files.
- Scripts were run on University of Michigan Great Lakes HPC; Slurm scripts reflect that infrastructure.
- Dataset files, checkpoint references, and intermediate artifacts are **not guaranteed** to be available within this GitHub repository.
- Current run scripts include absolute paths such as `/home/smanzoor/...` and `/scratch/bioinf593f25_class_root/...`. These must be adjusted by anyone attempting a run.

## What a new user would need to do

1. Start from a compatible Python/PyTorch environment; follow GET upstream installation guidance and inspect this branch's `env.yml` / `pyproject.toml`.
2. Obtain the legally accessible PBMC10k multiome dataset; do not assume cluster-internal data are bundled.
3. Replace `codebase_dir`, `scratch_dir`, `data_path`, `checkpoint_path`, and `output_dir` in experiment scripts with paths valid for your environment.
4. Review [`GETDiffusion.yaml`](../get_model/config/model/GETDiffusion.yaml) and configuration overrides, especially masking, timesteps, training duration, and leave-out cell type.
5. If using Slurm, adapt `scripts/*.sbatch` to your own account, partition, resources and environment.
6. Run a small smoke test first. The authors have **not** verified a clean-clone full experiment reproduction across machines.
7. For evaluation, record exact checkpoint, subset/cell type, element-vs-cell-vs-motif aggregation, sample count and score implementation before comparing to course-manuscript metrics.

## Checkpoints and metrics caveat

- Manuscript Table 2 reports an overall comparison for held-out `cd8_tem_1`.
- The exported results under `results/` use differing precision and count fields; `diffusion_vs_transformer_results.json` reports different sample counts for the two models.
- The current README intentionally identifies which values are **manuscript-reported** rather than representing them as independently confirmed reproducible benchmarks.

## Important scope boundaries

- DiT-style timestep conditioning is tested for masked region-by-motif predictions; this repository does not establish performance for downstream clinical or broader transcriptomics tasks.
- Comparison is based on one dataset, one held-out cell type and 20 training epochs.
- See [project notes](PROJECT_NOTES.md) for further caveats.
