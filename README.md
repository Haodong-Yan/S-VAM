<div align="center">

# [S-VAM: Shortcut Video-Action Model by Self-Distilling Geometric and Semantic Foresight](https://haodong-yan.github.io/S-VAM/)

Haodong Yan\*, Zhide Zhong\*, Jiaguan Zhu, Junjie He, Weilin Yuan, Wenxuan Song,
Xin Gong, Yingjie Cai, Guanyi Zhao, Xu Yan, Bingbing Liu, Ying-Cong Chen, Haoang Li

**HKUST (Guangzhou) &nbsp; | &nbsp; Huawei Foundation Model Department**

[![arXiv](https://img.shields.io/badge/arXiv-2603.16195-b31b1b.svg)](https://arxiv.org/abs/2603.16195)
[![Project Page](https://img.shields.io/badge/Project-Page-blue)](https://haodong-yan.github.io/S-VAM/)

</div>

## News

- **[2026/07]** Code and pretrained weights released.
- **[2026/06]** S-VAM is accepted by **ECCV 2026**!

## Overview

<p align="center">
  <img src="assets/intro.png" width="90%">
</p>

S-VAM establishes a *shortcut* that foresees coherent geometric and semantic representations in a single forward pass. Lightweight decouplers are trained via self-distillation to map one-step diffusion features to DINOv2 (semantic) and Depth Anything v3 (geometric) targets. A Uni-Perceiver then aggregates these foreseen representations with original diffusion features to condition an EDM-based diffusion policy.

**Avg.Len of 4.16 on Calvin ABC-D benchmark.**

## Installation

```bash
conda create -n s-vam python=3.10
conda activate s-vam

# Install CALVIN (environment + dataset)
# Follow https://github.com/mees/calvin for render dependencies
git clone --recurse-submodules https://github.com/mees/calvin.git
cd calvin && sh install.sh && cd ..

# Install S-VAM requirements
# Note: calvin requires torch==1.13 but S-VAM works with torch>=2.0, just ignore the warning
pip install -r requirements.txt
```

## Checkpoints

| Model | Source | Size | Purpose |
|-------|--------|------|---------|
| `svd-robot-calvin-ft` | [HuggingFace](https://huggingface.co/yjguo/svd-robot-calvin-ft) | ~8 GB | Finetuned SVD video model |
| `clip-vit-base-patch32` | [HuggingFace](https://huggingface.co/openai/clip-vit-base-patch32) | ~600 MB | Text encoder (frozen) |
| `s-vam-weights` | [ModelScope](https://modelscope.cn/models/haodong123/s-vam-weights) | ~6 GB | Decoupler weights + trained action model |

```bash
# Download base models from HuggingFace
huggingface-cli download yjguo/svd-robot-calvin-ft --local-dir ./svd-robot-calvin-ft
huggingface-cli download openai/clip-vit-base-patch32 --local-dir ./clip-vit-base-patch32

# Download S-VAM weights from ModelScope
pip install modelscope
modelscope download haodong123/s-vam-weights --local_dir ./s-vam-weights
```

All checkpoint paths can be specified via command-line arguments (`--hidden2dino_ckpt`, `--hidden2dpa_ckpt`, `--action_model_folder`), so you can place them anywhere.

## Reproducing Results on Calvin ABC-D

### Download Calvin ABC-D Dataset

Follow instructions in the [official CALVIN repo](https://github.com/mees/calvin) to download the Calvin ABC-D dataset (~500 GB). Set `CALVIN_DATA_DIR` to the path of `task_ABC_D`.

### Training

```bash
accelerate launch \
  --num_processes 4 \
  --num_machines 1 \
  step2_train_action_calvin.py \
  --root_data_dir ${CALVIN_DATA_DIR} \
  --video_model_path ${VIDEO_MODEL_PATH} \
  --text_encoder_path ${TEXT_ENCODER_PATH} \
  --hidden2dino_ckpt ${HIDDEN2DINO_CKPT} \
  --hidden2dpa_ckpt ${HIDDEN2DPA_CKPT} \
  --log_dir ./logs/s_vam_calvin
```

Checkpoints are saved every 20k steps to `<log_dir>/<run_tag>/checkpoints/`.

### Evaluation

```bash
export PYTHONPATH="./calvin/calvin_env:.:$PYTHONPATH"

python policy_evaluation/calvin_evaluate_our.py \
  --action_model_folder ${ACTION_MODEL_CKPT} \
  --calvin_abc_dir ${CALVIN_DATA_DIR} \
  --video_model_path ${VIDEO_MODEL_PATH} \
  --clip_model_path ${CLIP_MODEL_PATH} \
  --hidden2dino_ckpt ${HIDDEN2DINO_CKPT} \
  --hidden2dpa_ckpt ${HIDDEN2DPA_CKPT} \
  --force_eval
```

## Acknowledgement

This codebase is built upon [VPP](https://github.com/roboterax/video-prediction-policy), [Stable Video Diffusion](https://huggingface.co/stabilityai/stable-video-diffusion-img2vid-xt), [CALVIN](https://github.com/mees/calvin), and [MDT](https://github.com/intuitive-robots/mdt_policy). We thank the authors for their excellent work.

## Citation

```bibtex
@misc{yan2026svam,
  title={S-VAM: Shortcut Video-Action Model by Self-Distilling Geometric and Semantic Foresight},
  author={Haodong Yan and Zhide Zhong and Jiaguan Zhu and Junjie He and Weilin Yuan and Wenxuan Song and Xin Gong and Yingjie Cai and Guanyi Zhao and Xu Yan and Bingbing Liu and Ying-Cong Chen and Haoang Li},
  year={2026},
  eprint={2603.16195},
  archivePrefix={arXiv},
  primaryClass={cs.CV},
  url={https://arxiv.org/abs/2603.16195},
}
```

## License

TBD
