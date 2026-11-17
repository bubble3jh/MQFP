<h1 align="center">Multi-Query Frequency Prompting<br>for Physiological Signal Domain Adaptation</h1>

<p align="center">
  <b>Knowledge-Based Systems, 2025</b><br>
  Jinho Kang, Hoyoon Byun, Taero Kim, Jiyoung Jung, Kyungwoo Song
</p>

<p align="center">
  <a href="https://www.sciencedirect.com/science/article/abs/pii/S095070512501127X"><img src="https://img.shields.io/badge/Paper-ScienceDirect-F36D21" alt="Paper"></a>
  <a href="https://doi.org/10.1016/j.knosys.2025.114082"><img src="https://img.shields.io/badge/DOI-10.1016%2Fj.knosys.2025.114082-0085CA" alt="DOI"></a>
  <img src="https://img.shields.io/badge/PyTorch-EE4C2C?logo=pytorch&logoColor=white" alt="PyTorch">
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/29397798-9e08-43fe-9acd-f0a800080a9a" width="90%" alt="Frequency-domain prompting is robust to pulse shift">
</p>

Official PyTorch implementation of **MQFP**, a prompt-learning framework that adapts a pre-trained blood pressure
model to a new PPG dataset by prompting in the frequency domain.

- **Robust to pulse shift.** A prompt added in the frequency domain, followed by reconstruction of the signal,
  withstands physiological pulse shifts far better than a prompt added to the raw signal.
- **Multi-query prompts.** Prompts are selected by signal features such as variability, periodicity, and locality,
  which makes the synthesized prompt flexible and interpretable.
- **Accurate and light.** Across three benchmark datasets, MQFP reduces the mean absolute error by 41.67% compared
  with the baselines while cutting the trainable parameters by up to 98.4%.

## Overview

<p align="center">
  <img src="https://github.com/user-attachments/assets/bc81898c-904c-41b4-838a-5d3346d5d7e6" width="90%" alt="Multi-query prompt selection">
</p>

<p align="center"><em>Prompts are chosen from a pool by queries on several features of the signal.</em></p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/974c8365-d165-4e1f-b17e-d8a2a2fb634d" width="90%" alt="Prompted PPG signals: frequency prompting versus vanilla prompting">
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/24014140-c3c4-42ef-a86a-4c19eb560d30" width="90%" alt="Comparison with vanilla prompting and full fine-tuning">
</p>

<p align="center"><em>Frequency prompting yields far more plausible signals than vanilla prompting, and in 10-shot
transfer it performs better while updating 1.6% of the weights of full fine-tuning.</em></p>

## Getting started

### Installation

```bash
pip install -r requirements.txt
```

### Data and pre-trained models

Datasets are acquired and pre-processed as in
[bp-benchmark](https://github.com/inventec-ai-center/bp-benchmark).

```
├── bp-algorithm
├── datasets
│   ├── splits
│   │   ├── bcg_dataset
│   │   │   ├── feat_fold_0.csv
│   │   │   ├── ...
│   │   │   └── feat_fold_4.csv
```

Set the paths of the pre-trained checkpoints in `code/train/core/load_model.py`:

```python
import os
root = '/path/to/model'
join = os.path.join
model_fold = {"bcg": {0: {0: join(root, 'bcg-resnet1d/fold0.ckpt'),
                    {1: join(root, 'bcg-resnet1d/fold1.ckpt'),
                    {2: join(root, 'bcg-resnet1d/fold2.ckpt'),
                    {3: join(root, 'bcg-resnet1d/fold3.ckpt'),
                    {4: join(root, 'bcg-resnet1d/fold4.ckpt')}}}
```

### Running

```bash
cd code/train

# sweep every training setting
bash experiments/bash_for_bash.sh

# Sensors -> BCG transfer
python train.py --transfer=sensors --target=bcg --batch_size=6 --clip=false \
    --config_file=core/config/dl/resnet/resnet_bcg.yaml --diff_loss_weight=0.4 --epochs=10 \
    --global_coeff=0.1 --lr=0.009 --method=prompt_global --normalize=true --num_pool=9 \
    --pca_dim=16 --query_dim=16 --reset_head=false --shots=10 --train_head=true \
    --train_imag=true --trunc_dim=50 --use_emb_diff=true --wd=0.03
```

### Arguments

| Argument | Description |
|---|---|
| `--transfer` | Source dataset of the pre-trained model |
| `--target` | Target dataset for domain adaptation |
| `--config_file` | Configuration file of the experiment, chosen by the target dataset |
| `--method` | `original` for the baselines, `prompt_global` for MQFP |
| `--shots` | Amount of training data in the few-shot setting |
| `--lr`, `--wd`, `--batch_size` | Learning rate, weight decay, batch size |
| `--global_coeff` | Coefficient of the prompt added to the original signal |
| `--normalize` | Normalize the prompted input to the distribution of the source dataset |
| `--clip` | Clip to the range of the original signal |
| `--num_pool` | Number of keys and prompts in the prompt pool |
| `--trunc_dim` | Size of the prompted region of the frequency spectrum |
| `--query_dim` | Size of the queries and keys |
| `--pca_dim` | Reduced dimension of the PCA |
| `--train_imag` | Train the imaginary part |
| `--train_head` | Train the last head layer of the pre-trained model |
| `--reset_head` | Reset the weights of the last head layer |
| `--use_emb_diff` | Auxiliary loss that keeps the embedding close to the source embedding |
| `--diff_loss_weight` | Weight of the auxiliary loss |

## Abstract

In this study, we address challenges in applying deep learning to photoplethysmography (PPG) signals for blood pressure prediction, including restricted data availability, domain shifts, and inadequate focus on minority groups with hypertension or hypotension. We propose a novel prompt-learning framework that employs frequency-domain prompting to enhance the robustness of pre-trained models against physiological pulse shifts without extensive retraining. By integrating learnable prompts based on critical signal features such as variability, periodicity, and locality, our method enables more precise adaptations to diverse datasets and conditions. Experimental results across three benchmark datasets demonstrate a significant improvement in accuracy, with a 41.67% reduction in mean absolute error compared to baseline models, alongside a reduction in trainable parameters by up to 98.4%. This approach boosts performance and promotes equitable, efficient healthcare diagnostics across diverse settings.

## Citation

```bibtex
@article{kang2025multi,
  title={Multi-Query Frequency Prompting for Physiological Signal Domain Adaptation},
  author={Kang, Jinho and Byun, Hoyoon and Kim, Taero and Jung, Jiyoung and Song, Kyungwoo},
  journal={Knowledge-Based Systems},
  pages={114082},
  year={2025},
  publisher={Elsevier}
}
```

The lab-maintained copy of this repository is at [MLAI-Yonsei/MQFP](https://github.com/MLAI-Yonsei/MQFP).
