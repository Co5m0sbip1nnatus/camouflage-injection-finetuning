# Fine-Tuning a Prompt-Injection Detector Against Camouflaged Attacks

A controlled experiment on the **Camouflage Detection Gap** (Pai 2026, [arXiv:2605.22001](https://arxiv.org/abs/2605.22001)): few-shot injection detectors catch overt attacks but miss ones camouflaged in a document's own vocabulary. This repo asks what nobody had tried in the original paper: does actually fine-tuning the detector close the gap?

**Answer: partially, at a cost.** QLoRA fine-tuning improves camouflage detection but degrades static detection (catastrophic forgetting). A four-way hyperparameter sweep fails to remove the trade-off. An OR-ensemble of the few-shot and fine-tuned detectors routes around it entirely.

| Detector | Static | Camouflage | FP | F1 |
|---|---|---|---|---|
| Few-shot (control) | 100% | 52.5% | 12.5% | 0.836 |
| QLoRA fine-tuned (best single) | 90 to 95% | 60 to 65% | 12.5 to 15% | 0.844 |
| **Ensemble (OR)** | **100%** | **82.5%** | 22.5% | **0.901** |

Full write-up: [blog post](BLOG_URL_HERE)

## What's in the notebook

`exp1_modular.ipynb` is a single self-contained notebook, executable top to bottom:

1. **Data**: 200 tasks x 5 domains from the paper's released task bank; benign / static / LLM-generated camouflage samples; split by task_id (no near-duplicate leakage); fixed 120-sample test set shared by every method
2. **Baseline**: few-shot detector on Llama 3.1 8B (the control arm)
3. **Fine-tuning**: 4-bit QLoRA (r=16, 0.08% trainable params) with validation, cosine schedule, gradient clipping, best-val checkpointing, early stopping
4. **Sweep**: one-factor-at-a-time over data balance, epochs, learning rate, and rank, with a pre-registered selection rule
5. **Ensemble**: OR-combination computed from saved predictions, plus a complementarity analysis (the two detectors catch substantially different camouflage samples)

Long stages are cached: completed runs, the baseline, and generated data are skipped on re-execution.

## Requirements

- Single 24GB GPU (developed on an RTX 3090, RunPod, PyTorch 2.8 + CUDA 12.8)
- Hugging Face access to `meta-llama/Llama-3.1-8B-Instruct`
- An OpenRouter API key, only if regenerating the camouflage data (about $0.12)

## Key caveats

With 40 test samples per attack type, one sample is 2.5 points; single-run differences under roughly 10 points are treated as inconclusive. Greedy decoding is not bit-reproducible across GPU hardware. Details in the notebook's conclusions.

## Credits

Task bank, static payload templates, and few-shot detector prompt from [aaditya79/defense-eval-camouflage-injection](https://github.com/aaditya79/defense-eval-camouflage-injection), released with Pai (2026), "Blind Spots in the Guard". This repo builds on that setup with a new question: fine-tuning as a defense.
