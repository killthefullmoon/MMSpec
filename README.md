# MMSpec: Benchmarking Speculative Decoding for Vision-Language Models

<p align="center">
  <strong>If you like our project, please give us a star ⭐ on GitHub for the latest update.</strong>
</p>

Code for the paper "[MMSpec: Benchmarking Speculative Decoding for Vision-Language Models](https://killthefullmoon.github.io/projects/MMSpec/)".

For more details, please refer to the project page with **dataset exploration and visualization tools**: [MMSpec Project Page](https://killthefullmoon.github.io/projects/MMSpec/).

[[🌐 Project Page](https://killthefullmoon.github.io/projects/MMSpec/)] [[📖 Paper](https://arxiv.org/abs/2603.14989)] [[🤗 Checkpoints](https://huggingface.co/collections/Cloudriver/multimodal-speculative-decoding)]

<p align="center">
  <img src="https://killthefullmoon.github.io/projects/MMSpec/static/images/mmspec/spec_radar_qwen.png" alt="MMSpec radar comparison" width="78%">
</p>

## About The Project

MMSpec is a benchmark for studying speculative decoding in vision-language models (VLMs). It is designed for fair third-party comparison under a unified evaluation protocol and introduces ViSkip, a plug-and-play vision-aware strategy that skips speculative drafting when the next token depends heavily on visual evidence.

The benchmark contains 600 multimodal samples from 6 task categories, covers 10 representative lossless speculative decoding methods, and reports both Mean Accepted Tokens (MAT) and Walltime Speedup Ratio.

## Highlights

- First benchmark dedicated to speculative decoding for VLMs.
- Unified evaluation setup for both training-based and training-free methods.
- Covers Qwen2.5-VL-7B-Instruct and LLaVA-1.5-7B as the main evaluation targets.
- Includes ViSkip variants for vision-aware latency reduction on top of existing methods.

## Outlines

- [Benchmark At A Glance](#benchmark-at-a-glance)
- [Methods Covered](#methods-covered)
- [Repository Structure](#repository-structure)
- [Installation](#installation)
- [Evaluation](#evaluation)
- [Training](#training)
- [Key Findings](#key-findings)
- [Citation](#citation)

## Benchmark At A Glance

MMSpec is built around workload diversity, balanced topic coverage, multi-turn support, and method-agnostic measurement.

| Category | Source | Avg. output length |
| --- | --- | ---: |
| General VQA | GQA | 46.98 tokens |
| Text VQA | TextVQA | 63.15 tokens |
| Image Captioning | COCO | 191.90 tokens |
| Chart VQA | CharXiv | 68.56 tokens |
| Complex Reasoning | MMMU-Pro | 285.60 tokens |
| Multi-turn Conversation | ConvBench, MM-MT-Bench | 747.65 tokens |

Dataset splits are stored under [`dataset/MMSpec/`](./dataset/MMSpec):

- `testmini`: quick sanity-check subset
- `test`: full benchmark split
- `caption_marker_test`: generated image annotations organized by MMSpec topic folders

Each split contains `mmspec.jsonl` and an `images/` directory. A typical sample includes `id`, `image`, `turns`, `category`, and `topic`.

## Methods Covered

MMSpec unifies 10 representative lossless speculative decoding families:

- ViSpec
- MSD
- EAGLE-1 / EAGLE-2 / EAGLE-3
- Medusa
- SAM Decoding
- Lookahead
- Recycling
- PLD

This repository additionally provides runnable evaluation entrypoints for:

- baseline autoregressive decoding
- ViSkip-enhanced variants: `vispec_vskip`, `msd_vskip`, `sam_vskip`

<p align="center">
  <img src="https://killthefullmoon.github.io/projects/MMSpec/static/images/mmspec/methods.png" alt="MMSpec methods overview" width="78%">
</p>

## Repository Structure

- [`dataset/`](./dataset): benchmark data used by MMSpec.
- [`evaluation/`](./evaluation): Python entrypoints for benchmark execution.
- [`method/`](./method): speculative decoding implementations, including ViSpec and ViSkip variants.
- [`scripts/`](./scripts): ready-to-run evaluation scripts grouped by model.
- [`train/`](./train): training code and launch scripts for EAGLE, EAGLE3, Medusa, and MSD.

The current evaluation scripts are organized by target model:

- [`scripts/Qwen2.5-VL-7B`](./scripts/Qwen2.5-VL-7B)
- [`scripts/LLaVA-1.5-7B`](./scripts/LLaVA-1.5-7B)

## Installation

MMSpec requires Python 3.10+ and `transformers==4.51.3`.

```bash
pip install -r requirements.txt
```

If you plan to train EAGLE3 or MSD with DeepSpeed, install the corresponding runtime separately.

## Evaluation

Run all commands from the project root.

### Quick Start

Evaluate the Qwen model on `testmini`:

```bash
bash scripts/Qwen2.5-VL-7B/eval_baseline_mmspec.sh testmini
bash scripts/Qwen2.5-VL-7B/eval_vispec_mmspec.sh testmini
bash scripts/Qwen2.5-VL-7B/eval_vispec_vskip_mmspec.sh testmini
```

Evaluate the LLaVA model on the full `test` split:

```bash
bash scripts/LLaVA-1.5-7B/eval_baseline_mmspec.sh test
bash scripts/LLaVA-1.5-7B/eval_msd_mmspec.sh test
bash scripts/LLaVA-1.5-7B/eval_sam_vskip_mmspec.sh test
```

All evaluation scripts accept `testmini` or `test` as the first argument and write results to:

```text
results/<model_name>/mmspec_<split>/
```

### Available Methods

For both model folders, the following entrypoints are available:

- `eval_baseline_mmspec.sh`
- `eval_eagle_mmspec.sh`
- `eval_eagle2_mmspec.sh`
- `eval_eagle3_mmspec.sh`
- `eval_lookahead_mmspec.sh`
- `eval_medusa_mmspec.sh`
- `eval_msd_mmspec.sh`
- `eval_msd_vskip_mmspec.sh`
- `eval_pld_mmspec.sh`
- `eval_recycling_mmspec.sh`
- `eval_sam_mmspec.sh`
- `eval_sam_vskip_mmspec.sh`
- `eval_vispec_mmspec.sh`
- `eval_vispec_vskip_mmspec.sh`

Some scripts expose optional checkpoint overrides through environment variables or a second positional argument. The default model and checkpoint paths are defined directly inside each script.

## Training

Training utilities live in [`train/`](./train). The repository currently includes launch scripts and code for:

- EAGLE stage 1 / stage 2
- EAGLE3 stage 1 / stage 2
- Medusa
- MSD

See [`train/README.md`](./train/README.md) for the available launch scripts and training entrypoints.

## Key Findings

From the MMSpec benchmark and project page:

- Training-free methods usually provide limited gains in multimodal decoding and can even regress latency.
- Training-based methods that ignore visual information still underperform in VLM inference.
- Throughput speedup alone is not enough; stable end-to-end latency matters in practice.
- Vision-aware control, as used in ViSkip, becomes increasingly important as batch size grows.

## Citation

If you find MMSpec useful, please cite:

```bibtex
@article{shen2025mmspec,
  title={MMSpec: Benchmarking Speculative Decoding for Vision-Language Models},
  author={Hui Shen and Xin Wang and Ping Zhang and Yunta Hsieh and Qi Han and Zhongwei Wan and Ziheng Zhang and Jingxuan Zhang and Jing Xiong and Ziyuan Liu and Yifan Zhang and Hangrui Cao and Chenyang Zhao and Mi Zhang},
  year={2025},
  note={Preprint}
}
```

## Acknowledgements

This repository builds on prior speculative decoding systems including EAGLE and Medusa, and consolidates them into a unified VLM benchmarking framework.
