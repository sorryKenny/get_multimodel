# GETDiffusion research notes (BIOINF 593, Fall 2025)

## Question and context

Could a timestep-conditioned Diffusion Transformer encoder improve GET's masked genomic region-by-motif reconstruction compared with its standard Transformer encoder?

This project uses the GET codebase as a foundation. The cited *Nature* paper and upstream GET implementation belong to the GET Foundation authors. The DiT adaptation described here was performed as a University of Michigan course team project.

## Core method

- Input: region-by-motif tensor with `900 × 283` features per example.
- Region embedding: `283 → 768`; masking rate `50%`; CLS token introduced before encoder.
- Backbone: 12 DiT-style blocks, 12 attention heads, `768` hidden size.
- Timestep: sinusoidal 256-dimensional embedding projected to `768`.
- adaLN: timestep-dependent shift, scale and gate controls for attention and MLP residual sublayers.
- Prediction: project encoder outputs `768 → 283`, optimizing masked-feature MSE.
- Experimental setting in course manuscript: PBMC 10k multiome, held-out `cd8_tem_1`, 20 epochs, batch 8, AdamW with learning rate 1e-4, 1000 diffusion timesteps, beta endpoints 0.0001–0.02.

## Reported metrics (manuscript Table 2)

| Metric | DiT | Baseline |
| --- | ---: | ---: |
| Pearson | 0.7960 | 0.7800 |
| Spearman | 0.5890 | 0.3817 |
| R² | 0.6313 | 0.6083 |
| MSE | 0.00125 | 0.00238 |
| MAE | 0.0155 | 0.0307 |

Per-motif mean Pearson: 0.407 vs 0.074 (reported in manuscript).

**Interpretation limits:** These were presented as course-project results, not as externally validated production metrics. This branch includes exported experimental results with some differing aggregation counts; comparisons should not be generalized before independent reproducibility checks.

## Reported author contributions

- Kenny Chen: model-design contribution, experimental design, manuscript contribution.
- Sumeed Manzoor: data preparation, architecture implementation, model evaluation, figure generation, manuscript contribution.
- Geordan Bolden: model direction and architecture guidance.
- All authors: project conception.

## Limitations highlighted in manuscript

1. One PBMC 10k dataset and one held-out cell type.
2. 20 epochs only; convergence may be incomplete.
3. No dedicated component ablations.
4. Larger model (~129M vs 85M parameters).
5. No direct downstream validation for gene expression prediction, variant effects, or perturbation response.

## Sharing status

The team manuscript is **not** a peer-reviewed publication and its original template displayed incorrect publisher/DOI metadata. A corrected, coauthor-approved PDF may be linked here in the future; this repository does not automatically treat the course manuscript as approved for public release.
