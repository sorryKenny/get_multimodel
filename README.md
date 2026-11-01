# GETDiffusion | Diffusion Transformer for Genomic Motif Prediction

> **University of Michigan · BIOINF 593 Course Research · Fall 2025**  
> **Team project.** This is **not** an official GET Foundation release, a peer-reviewed publication, or an independently authored implementation.

## Overview

GETDiffusion is a course research project built on the open-source [GET (General Expression Transformer)](https://github.com/GET-Foundation/get_model). Our team explored replacing GET's standard Transformer encoder with timestep-conditioned Diffusion Transformer (DiT) blocks and adaptive layer normalization (adaLN) for **masked region-by-motif prediction** from single-cell multiomic features.

We compared the modified encoder against a standard GET baseline on the **PBMC 10k multiome dataset**, with **`cd8_tem_1` held out**. This repository preserves the team code, configurations, experimental artifacts, documented limitations, and individual contributions.

**Quick links:** [Architecture](#architecture) · [Results](#experimental-results) · [Repository Guide](#repository-guide) · [Reproducibility](#reproducibility) · [Contributions](#team-contributions) · [Limitations](#limitations) · [Credits](#credits-and-provenance)

## Research Question

Can diffusion-timestep conditioning and adaLN modulation improve masked region-by-motif feature reconstruction relative to GET's standard Transformer encoder?

The reported evaluation concerns **masked motif-feature prediction**. It does not independently establish improvements in downstream gene expression inference, variant-effect prediction, perturbation modeling, or zero-shot transfer.

## Architecture

| Component | Standard GET baseline | GETDiffusion |
| --- | --- | --- |
| Input | Region-by-motif features, `(B, 900, 283)` | Same |
| Region embedding | `283 → 768` | Same |
| Masking ratio | 50% | 50% |
| Encoder | 12 Transformer blocks | 12 DiT-style blocks |
| Attention heads | 12 | 12 |
| Conditioning | None | Sinusoidal timestep embedding → MLP → 768 |
| Normalization | Standard LayerNorm | Timestep-conditioned adaLN and residual gating |
| Prediction head | `768 → 283` | Same |
| Objective | Masked-position MSE | Masked-position MSE |
| Approximate parameters | 85.5M | 128.8M |

The main architectural change is the timestep-conditioned DiT encoder. A timestep embedder produces conditioning vectors that dynamically modulate the attention and MLP sublayers through adaLN shifts, scales, and gates. The model retains the region embedding and masked motif prediction task used in the comparison.

- [Model implementation](get_model/model/diffusion.py)
- [GETDiffusion model configuration](get_model/config/model/GETDiffusion.yaml)
- [Detailed model parameter comparison](results/model_parameter_comparison.md)

## Experimental Results

The following values are **reported in Table 2 of the team's December 2025 course manuscript**. They are not independently rerun benchmark results. The described experimental setting uses PBMC 10k multiome, a held-out `cd8_tem_1` cell type, and 20 training epochs.

| Metric | GETDiffusion | Standard GET |
| --- | ---: | ---: |
| Pearson correlation ↑ | **0.7960** | 0.7800 |
| Spearman correlation ↑ | **0.5890** | 0.3817 |
| R² ↑ | **0.6313** | 0.6083 |
| MSE ↓ | **0.00125** | 0.00238 |
| MAE ↓ | **0.0155** | 0.0307 |
| Mean per-motif Pearson ↑ | **0.407** | 0.074 |

The course manuscript also reports improved per-motif Pearson correlation for **282 of 283 motifs** relative to the baseline. These findings are confined to the reported experimental setup and should not be interpreted as demonstrated generalization to other datasets or tasks.

### Interpreting the saved result files

The [`results/`](results/) directory contains experimental exports. Some values, precision, and sample counts differ from the manuscript summary. In particular, [`diffusion_vs_transformer_results.json`](results/diffusion_vs_transformer_results.json) and [`comprehensive_summary.csv`](results/comprehensive_summary.csv) show unequal `n_samples` between the model outputs. The evaluation aggregation and compared subsets need further reconciliation before claiming a strictly same-sample independent comparison.

**The table above deliberately uses the course manuscript as its single reporting source** rather than blending values from potentially incompatible exports.

Available artifacts: [Architecture comparison](results/architecture_comparison.pdf) · [Checkpoint comparison](results/compare_checkpoints.pdf) · [Per-motif comparison](results/per_motif_comparison.csv) · [Parameter comparison](results/model_parameter_comparison.md).

## Repository Guide

| Location | Description |
| --- | --- |
| [`get_model/model/diffusion.py`](get_model/model/diffusion.py) | Diffusion Transformer / GETDiffusion model implementation |
| [`get_model/config/model/GETDiffusion.yaml`](get_model/config/model/GETDiffusion.yaml) | Model architecture and diffusion settings |
| [`get_model/config/finetune_GETDiffusion.yaml`](get_model/config/finetune_GETDiffusion.yaml) | Finetuning configuration |
| [`scripts/run_diffusion-transformer.py`](scripts/run_diffusion-transformer.py) | Experiment training entry point |
| [`scripts/run_diffusion-transformer.sbatch`](scripts/run_diffusion-transformer.sbatch) | Slurm / HPC job script |
| [`notebooks/pretrain_GETDiffusion.ipynb`](notebooks/pretrain_GETDiffusion.ipynb) | Pretraining and exploratory notebook |
| [`results/`](results/) | Exported metrics, comparisons, figures, and analysis |
| [`docs/REPRODUCIBILITY.md`](docs/REPRODUCIBILITY.md) | Environment requirements and reproducibility caveats |
| [`docs/PROJECT_NOTES.md`](docs/PROJECT_NOTES.md) | Research design, detailed settings, and limitations |

For the **original GET model** and its general installation tutorials, see the [GET Foundation repository](https://github.com/GET-Foundation/get_model). Its upstream commands and documentation do **not** automatically reproduce this course experiment.

## Reproducibility

This branch preserves the historical course code and outputs; it is **not a tested, one-command reproducible research package**.

The training entry points contain researcher-specific local paths and University of Michigan Great Lakes HPC paths. A new user would need to obtain accessible datasets and checkpoints, configure a compatible PyTorch/CUDA environment, adjust absolute paths, and adapt Slurm settings. The available repository does not guarantee that all training data or pretrained checkpoints are bundled.

Please read the [reproducibility guide](docs/REPRODUCIBILITY.md) before attempting to run the scripts. Do not assume the exported scores can be recreated from a clean clone without additional setup and validation.

## Team Contributions

**GETDiffusion was a collaborative course research project. The entire model codebase was not independently implemented by Kenny Chen.**

| Contributor | Contribution stated in the course manuscript |
| --- | --- |
| **Kenny Chen (K.C.)** | Contributed to model design, experimental design, and manuscript writing |
| **Sumeed Manzoor (S.M.)** | Data preparation, architecture implementation, evaluation, figures, and manuscript writing |
| **Geordan Bolden (G.B.)** | Guidance on model direction and architecture; project conception |

The manuscript credits all authors with the conception of the project. This portfolio fork is intended to document **the team research and Kenny's specific contributions**, not to claim individual authorship of all inherited or team-written source code.

## Limitations

- **Limited evaluation scope:** One dataset (PBMC 10k) and one held-out cell type; generalization remains unverified.
- **Training duration:** 20 epochs; convergence was not exhaustively studied.
- **Missing ablations:** No definitive isolation of the effects of timestep conditioning and adaLN.
- **Added complexity:** Approximately 50% more parameters than the standard Transformer baseline, with additional computational cost.
- **No downstream validation:** No direct demonstration of improvements in gene expression prediction, variant effects, or perturbation response.
- **Reporting consistency:** Some saved exports use differing sample counts or aggregation conventions; conclusions must remain tied to the documented evaluation.

## Credits and Provenance

- **Upstream implementation:** [GET Foundation — General Expression Transformer](https://github.com/GET-Foundation/get_model). The upstream peer-reviewed paper is [*A foundation model of transcription across human cell types* (Nature)](https://doi.org/10.1038/s41586-024-08391-z).
- **Team research branch:** [Yoyomanzoor/get_multimodel, `yoyo`](https://github.com/Yoyomanzoor/get_multimodel/tree/yoyo), preserved in this fork for project documentation.
- **Academic context:** University of Michigan, BIOINF 593, Fall 2025. The original manuscript acknowledges project mentors and computing support.
- **Publication status:** The GETDiffusion work was a **December 2025 course research manuscript**, not a peer-reviewed Oxford University Press or other journal publication. A GitHub repository URL is not a DOI.
- **Licensing:** This fork inherits material from GET. Follow upstream licenses and attribution requirements; this project summary does not grant new rights.

