# GETDiffusion | Diffusion Transformer for Genomic Motif Prediction

> **University of Michigan · BIOINF 593 course research · Fall 2025**  
> **Team project / 课程小组研究** · This is **not** an official GET Foundation release, a peer-reviewed publication, or an independently authored implementation.

**中文概述：** 本项目基于开源 [GET (General Expression Transformer)](https://github.com/GET-Foundation/get_model)，探索以带时间步条件的 Diffusion Transformer（DiT）编码器替换原始 Transformer，在单细胞多组学数据上进行掩码区域–motif 特征预测。我们在 PBMC 10k multiome 数据集、留出 `cd8_tem_1` 细胞类型的实验中，与原始 GET 编码器进行了对照。

**English summary:** An academic team experiment extending GET with timestep-conditioned DiT blocks and adaptive layer normalization (adaLN) for masked region-by-motif prediction on PBMC 10k multiome data. Code, configurations, comparison artifacts, limitations, and the team contribution breakdown are documented below.

## 快速导航 / At a glance

- [模型设计](#模型设计--architecture)
- [实验结果](#实验结果--results)
- [代码导航](#代码导航--repository-guide)
- [如何尝试复现](#复现说明--reproducibility)
- [个人贡献与团队分工](#个人贡献与团队分工--contributions)
- [限制与后续工作](#限制与后续工作--limitations)
- [来源与致谢](#来源与致谢--credits)

## 研究问题 / Research question

原始 GET 使用 Transformer 编码区域–motif 特征。我们研究：**扩散时间步条件化与 adaLN 调制能否改善掩码 motif 特征重建？** 这个项目的已报告指标是 **motif-level masked prediction**；不能直接等同于已经验证了下游基因表达、变异效应或零样本扰动预测能力。

## 模型设计 / Architecture

| Component | GET baseline | GETDiffusion |
| --- | --- | --- |
| Input | Region-by-motif matrix `(B, 900, 283)` | Same |
| Region embedding | `283 → 768` | Same |
| Masking | 50% | 50% |
| Encoder | 12 standard Transformer blocks | 12 DiT-style blocks |
| Attention | 12 heads | 12 heads |
| Conditioning | None | Sinusoidal timestep embedding → MLP → `768` |
| Normalization | Standard LayerNorm | Timestep-conditioned adaLN modulation and residual gates |
| Prediction head | `768 → 283` | Same |
| Loss | Masked-position MSE | Masked-position MSE |
| Approx. parameters | 85.5M | 128.8M |

核心区别在于 DiT 模块利用时间步嵌入生成调制参数，对注意力和 MLP 子层进行自适应调制。更详细的模型配置见 [`get_model/config/model/GETDiffusion.yaml`](get_model/config/model/GETDiffusion.yaml)，实现见 [`get_model/model/diffusion.py`](get_model/model/diffusion.py)。

## 实验结果 / Results

以下数据来自 **2025 年团队课程手稿 Table 2**，不是独立复现实验或公开 benchmark 榜单。评估背景为 PBMC 10k multiome、留出 `cd8_tem_1` 细胞类型、20 epochs；数据仅支持这一实验设置下的对比。

| Metric | GETDiffusion | Baseline GET |
| --- | ---: | ---: |
| Pearson | **0.7960** | 0.7800 |
| Spearman | **0.5890** | 0.3817 |
| R² | **0.6313** | 0.6083 |
| MSE ↓ | **0.00125** | 0.00238 |
| MAE ↓ | **0.0155** | 0.0307 |
| Mean per-motif Pearson | **0.407** | 0.074 |

课程手稿还报告 283 个 motif 中有 282 个在 per-motif Pearson 指标上优于 baseline。这是该实验设置的观察结果，不应解释为跨数据集或下游任务的已证实改进。

**关于结果口径：** 仓库 [`results/`](results/) 内的 JSON/CSV 为实验期间导出的原始文件；其中部分指标的精度和样本计数与手稿汇总不完全一致。例如 [`diffusion_vs_transformer_results.json`](results/diffusion_vs_transformer_results.json) 与 [`comprehensive_summary.csv`](results/comprehensive_summary.csv) 记录了不相等的 `n_samples`。在复核 checkpoint、预测粒度及评估集合之前，不应将其呈现为严格相同样本数量上的独立验证。上表以 **课程手稿** 为唯一口径。

实验图表：[architecture comparison](results/architecture_comparison.pdf) · [checkpoint comparison](results/compare_checkpoints.pdf) · [per-motif results](results/per_motif_comparison.csv) · [parameter comparison](results/model_parameter_comparison.md)

## 代码导航 / Repository guide

| Path | Purpose |
| --- | --- |
| [`get_model/model/diffusion.py`](get_model/model/diffusion.py) | GETDiffusion / DiT-style model implementation |
| [`get_model/config/model/GETDiffusion.yaml`](get_model/config/model/GETDiffusion.yaml) | Model architecture and diffusion parameters |
| [`get_model/config/finetune_GETDiffusion.yaml`](get_model/config/finetune_GETDiffusion.yaml) | Finetuning configuration |
| [`scripts/run_diffusion-transformer.py`](scripts/run_diffusion-transformer.py) | Training experiment entry point |
| [`scripts/run_diffusion-transformer.sbatch`](scripts/run_diffusion-transformer.sbatch) | Slurm/HPC launch example |
| [`notebooks/pretrain_GETDiffusion.ipynb`](notebooks/pretrain_GETDiffusion.ipynb) | Exploration and pretraining notebook |
| [`results/`](results/) | Analysis exports, figures and parameter comparisons |
| [`docs/REPRODUCIBILITY.md`](docs/REPRODUCIBILITY.md) | Honest setup limitations and reproduction checklist |
| [`docs/PROJECT_NOTES.md`](docs/PROJECT_NOTES.md) | Experimental scope and interpretation |

**原始 GET 的教程和安装文档：** [GET Foundation repository](https://github.com/GET-Foundation/get_model). Those upstream instructions describe GET and do **not** automatically reproduce this course experiment.

## 复现说明 / Reproducibility

当前分支保留研究时期的代码与结果，**并非开箱即用的独立可复现软件包**。训练脚本包含研究成员的本地路径与 U-M Great Lakes 集群路径；运行时需自行提供数据、替换路径并配置 CUDA/PyTorch 等依赖。公开结果不包含保证可用的模型 checkpoint 或完整训练数据打包。

请先读 [reproducibility notes](docs/REPRODUCIBILITY.md)，再尝试运行；避免直接执行含旧路径的脚本并期待得到论文指标。

## 个人贡献与团队分工 / Contributions

本项目为多人合作研究。**不是 Kenny 独立编写全部模型代码。**

| Team member | Contributions reported in the course manuscript |
| --- | --- |
| **Kenny Chen (K.C.)** | 参与模型设计、设计实验、参与研究手稿撰写 |
| **Sumeed Manzoor (S.M.)** | 数据准备、模型架构代码实现、模型评估、图表制作及手稿撰写 |
| **Geordan Bolden (G.B.)** | 模型方向与架构指导、项目构思 |

论文作者贡献声明指出全体作者参与项目构思。以上表述依据课程手稿，意在清楚区分设计、实验工作、代码实现和写作责任。此个人 Fork 仅用于**展示与说明团队研究成果**，不表示 Fork 所有源码由仓库所有者创作。

## 限制与后续工作 / Limitations

- 单一 PBMC 10k 数据集、单一留出细胞类型；泛化结论有限。
- 总训练轮数 20；无充分收敛性研究。
- 未进行可拆分证明时间步条件/adaLN 各自作用的 ablation。
- GETDiffusion 参数量约增加 50%，计算开销更大。
- 未直接验证下游基因表达、variant effects、perturbation 等任务。
- 原始实验日志/导出统计口径需要进一步对齐；请不要超范围引用指标。

## 来源与致谢 / Credits

- **Base model:** [GET Foundation — General Expression Transformer](https://github.com/GET-Foundation/get_model); related peer-reviewed article: [A foundation model of transcription across human cell types, Nature](https://doi.org/10.1038/s41586-024-08391-z).
- **Course team implementation:** [Yoyomanzoor/get_multimodel · yoyo](https://github.com/Yoyomanzoor/get_multimodel/tree/yoyo), preserved here for project documentation.
- **Project:** University of Michigan BIOINF 593, Fall 2025; acknowledgments to course/research mentors and Great Lakes HPC support, as in the manuscript.
- **Publication status:** University course research manuscript (December 2025), **not published in Oxford University Press or a peer-reviewed journal**. Do not use a GitHub URL as a DOI.
- **Licensing:** Source code originated from upstream GET; follow the upstream repository's license and attribution requirements. This README does not grant additional rights.

> **For recruiters / 招聘说明：** 可通过此 README 快速了解研究目标、模型变化、实验与个人贡献；团队主要代码实现不应被误认为 Kenny 的个人独立实现。
