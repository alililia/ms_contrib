# Contents

- [MVD Description](#mvd-description)
- [Dataset](#dataset)
- [Environment Requirements](#environment-requirements)
- [Quick Start](#quick-start)
- [Script Description](#script-description)
    - [Script and Sample Code](#script-and-sample-code)
    - [Script Parameters](#script-parameters)
    - [Training Process](#train)
    - [Evaluation Process](#evaluation)
- [Model Description](#model-description)
    - [Training Performance](#training-performance)
    - [Evaluation Performance](#evaluation-performance)
- [Description of Random Situation](#random-situation)
- [Reference](#reference)
- [ModelZoo Homepage](#modelzoo-homepage)

# MVD Description

MVD: Multi-view Variational Distillation for Person Re-identification.
Mindspore implementation for **Farewell to Mutual Information: Variational Distillation for Cross-Modal Person Re-identification** in CVPR 2021. Please read [our paper](https://openaccess.thecvf.com/content/CVPR2021/papers/Tian_Farewell_to_Mutual_Information_Variational_Distillation_for_Cross-Modal_Person_Re-Identification_CVPR_2021_paper.pdf) for a more detailed description of the training procedure. You can also read [Pytorch Version](https://github.com/FutabaSakuraXD/Farewell-to-Mutual-Information-Variational-Distiilation-for-Cross-Modal-Person-Re-identification) for further reference.

# Dataset

## Preparation

(1) SYSU-MM01 Dataset [1]: The SYSU-MM01 dataset can be obtained from this [website](https://github.com/wuancong/SYSU-MM01).

**run `python pre_process_sysu.py`(in `DDAG_mindspore/third_party/pre_process_sysu.py`) in to prepare the dataset, the training data will be stored in ".npy" format.**

(2) RegDB Dataset [2]: The RegDB dataset can be downloaded from this [website](http://dm.dongguk.edu/link.html) by submitting a copyright form.

(Named: "Dongguk Body-based Person Recognition Database (DBPerson-Recog-DB1)" on their website).

## Recommended Dataset Organization(Example)

```text
dataset
├──sysu                                     # SYSU-MM01 dataset
│  ├── cam1
│  ├── cam2
│  ├── cam3
│  ├── cam4
│  ├── cam5
│  ├── cam6
│  ├── exp
│  │   ├── available_id.mat
│  │   ├── available_id.txt
│  │   ├── test_id.mat
│  │   ├── test_id.txt
│  │   ├── train_id.mat
│  │   ├── train_id.txt
│  │   ├── val_id.mat
│  │   └── val_id.txt
│  ├── train_ir_resized_img.npy             # The following .npy files are generated by pre_process_sysu.py
│  ├── train_ir_resized_label.npy
│  ├── train_rgb_resized_img.npy
│  └── train_rgb_resized_label.npy
└── regdb                                   # RegDB Dataset
   ├── idx
   ├── Thermal
   └── Visible

```

**Note**: For SYSU-MM01 dataset, please first check whether it contains the above 4 `.npy` files. If not, please run `MVD/third_party/pre_process_sysu.py` to generate `.npy` files.

# Environment Requirements

- Hardware
    - Support Ascend and  environment.
    - For Ascend: Ascend 910.
    - For : cuda==10.1.
- Framework
    - Mindspore=1.5.0(See [Installation](https://www.mindspore.cn/install/))
- Third Package
    - Python==3.7.5
    - psutil==5.8.0
    - tqdm==4.62.0

*Note: these third party package are not stricted a specific version. For more details, please see `requriements.txt`.

# Quick Start

Example: SYSU-MM01 dataset training and all search inference


On Ascend:

```shell
cd MVD/scripts/ # please enter this path before sh XXX.sh, otherwise path errors :)
bash run_standalone_train_sysu_ascend.sh [DATASET_PATH] [CHECKPOINT_PATH] [SYSU_MODE] [DEVICE_ID]
```

Explanation: `[DATASET_PATH]` specifies your own path to  SYSU-MM01 or RegDB dataset. For example, if you organize dataset as [above structure](### Recommended Dataset Organization(Example)), then the path should be `/.../dataset/sysu` or `/.../dataset/regdb` respectively. `[CHECKPOINT_PATH]` specify your path to **resnet50** pretrain `.ckpt` file.

## Pretrain Checkpoint File

`--pretrain` parameter allows you to specify resnet50 checkpoint for pretrain backbone, while `--resume` parameter allows you to specify to previously saved whole network checkpoints to resume training.

We use `/model_zoo/r1.1/resnet50_ascend_v111_imagenet2012_official_cv_bs32_acc76` pretrain file, and the file [link](https://download.mindspore.cn/model_zoo/r1.1/resnet50_ascend_v111_imagenet2012_official_cv_bs32_acc76/).

For convenience, you can rename it as `resnet50.ckpt` and save it directly under `MVD/`, then you can leave `--pretrain resnet50.ckpt` unchanged.

# Script Description

## Scripts and Sample Code

```text
MVD
├── scripts
│   ├── run_eval_regdb_ascend.sh                  # Inference: RegDB dataset (infrared to visible) or (visible to infrared) on Ascend
│   ├── run_eval_regdb_.sh                     # Inference: RegDB dataset  (infrared to visible) or (visible to infrared)  on 
│   ├── run_eval_sysu_ascend.sh                   # Inference: SYSU-MM01 dataset (all or indoor) search on Ascend
│   ├── run_eval_sysu_.sh                      # Inference: SYSU-MM01 dataset (all or indoor) search on 
│   ├── run_standalone_train_regdb_ascend.sh      # Training: RegDB dataset  (infrared to visible) or (visible to infrared)  on Ascend
│   ├── run_standalone_train_regdb_.sh         # Training: RegDB dataset  (infrared to visible) or (visible to infrared)  on 
│   ├── run_standalone_train_sysu_ascend.sh       # Training: SYSU-MM01 dataset (all or indoor) search on Ascend
│   ├── run_standalone_train_sysu_.sh          # Training: SYSU-MM01 dataset (all or indoor) search on 
├── src
│   ├── dataset.py                                    # class and functions for Mindspore dataset
│   ├── evalfunc.py                                   # for evaluation functions
│   ├── loss.py                                       # loss functions
│   ├── models                                        # network architecture
│   │   ├── mvd.py                                    # main model
│   │   ├── resnet.py
│   │   ├── trainingcell.py                           # combine loss function, optimizer with network architecture
│   │   └── vib.py                                    # variational information bottleneck
│   └── utils.py
├── third_party
│   └── pre_process_sysu.py                           # preprocess SYSU-MM01 dataset to generate .npy format files
├── train.py
├── eval.py
├── requirements.txt
└── README.md
```

## Script Parameters

The following table describes the most commonly used arguments in scripts. You can change freely as you want.

| Config Arguments  |                         Explanation                          |
| :---------------: | :----------------------------------------------------------: |
|    `--MSmode`     | Mindspore running mode, either 'GRAPH_MODE' or 'PYNATIVE_MODE'. |
| `--device_target` |              choose "", "Ascend" or "Cloud"               |
|    `--dataset`    |              which dataset, "SYSU" or "RegDB".               |
|      `--`      | which  to run(default: 0), only effective when `--device_target ` |
|   `--device_id`   | which Ascend AI core to run(default:0), only effective when `--device_target Ascend` |
|   `--data_path`   | manually define the data path(for `SYSU`, path folder must contain `.npy` files, see [`pre_process_sysu.py`](#anchor1) ). |
|   `--pretrain`    | specify resnet-50 pretrain file path(default "" for no ckpt file)* |
|    `--resume`     | specify checkpoint file path for whole model(default "" for no ckpt file, `--resume` loads weights after `--pretrain`, and thus will overwrite `--pretrain` weights)* |
|   `--sysu_mode`   | choose from `["all", "indoor"]`, only effective when `args.dataset=SYSU` |
|  `--regdb_mode`   | choose from `["i2v", "v2i"]`, only effective when `args.dataset=RegDB` |
|  `--save_period`  | specify  every XXX epochs to save network weights into checkpoint files |

***Note: Please note that mindspore compulsorily requires checkpoint files have `.cpkt` as file suffix, otherwise may trigger errors during loading.**

We recommend that these following hyper-parameters in `.sh` files should be kept by default. If you want ablation study or fine-tuning, feel free to change :)

| Config Arguments |                         Explanation                          |
| :--------------: | :----------------------------------------------------------: |
|    `--optim`     |             choose "adam" or "sgd"(default adam)             |
|      `--lr`      |     initial learning rate( 0.0035 for adam, 0.1 for sgd)     |
|    `--epoch`     | the total number of training epochs, by default 80 Epochs(may be different from original paper). |
| `--warmup_steps` |                warm-up strategy, by default 5                |
| `--start_decay`  |         the start epoch of lr decay, by default 15.          |
|  `--end_decay`   |        the ending epoch of lr decay , by default 27.         |
|  `--loss_func`   | for ablation study, by default "id+tri" which is cross-entropy loss plus triplet loss. You can choose from `["id", "id+tri"]`. |

For more detailed and comprehensive arguments description, please refer to `train.py`.

## Training Process



For Ascend：

```shell
cd MVD/scripts/ # please enter this path before sh XXX.sh, otherwise path errors :)
bash run_standalone_train_sysu_ascend.sh [DATASET_PATH] [CHECKPOINT_PATH] [SYSU_MODE] [DEVICE_ID]
```

You can replace `run_standalone_train_sysu_.sh` or `run_standalone_train_sysu_ascend.sh` with other training scripts.

## Training Result

Training logs and corresponding checkpoint files will be stored in:

- SYSU-MM01 dataset + all search: `/scripts/train_sysu_all/SYSU_train_performance.txt`
- SYSU-MM01 dataset + indoor search: `/scripts/train_sysu_indoor/SYSU_train_performance.txt`
- RegDB dataset + visible to infrared(v2i): `/scripts/train_regdb_v2i/RegDB_train_performance.txt`
- RegDB dataset + infrared to visible(i2v): `/scripts/train_regdb_i2v/RegDB_train_performance.txt`

`.txt` files are training performance and `.ckpt` files are checkpoint files.

At the end of every epoch training, `train.py` will use a random testing set (different from training set) to evaluate the model performance. So you will see rank-1 and mAP performance. And this programming pattern of evaluation is analogy to `test.py`.

## Evaluation Process



On Ascend:

```shell
cd DDAG_mindspore/scripts/ # please enter this path before sh XXX.sh, otherwise path errors :)
bash run_eval_sysu_ascend.sh [DATASET_PATH] [CHECKPOINT_PATH] [SYSU_MODE] [DEVICE_ID]
```

Explanation: `[DATASET_PATH]` specifies your own path to  SYSU-MM01 or RegDB dataset, same as [Quick Start](## Quick Start) section. `[CHECKPOINT_PATH]` specifies your **saved checkpoint files during training**, not resnet50.

## Evaluation Result

After running `bash run_eval_XXX.sh [DATASET_PATH] [CHECKPOINT_PATH]`, you will get direct inference result.

- SYSU-MM01 dataset + all search: `/scripts/eval_sysu_all/SYSU_train_performance.txt`
- SYSU-MM01 dataset + indoor search: `/scripts/eval_sysu_indoor/SYSU_train_performance.txt`
- RegDB dataset + visible to infrared(v2i): `/scripts/eval_regdb_v2i/RegDB_train_performance.txt`
- RegDB dataset + infrared to visible(i2v): `/scripts/eval_regdb_i2v/RegDB_train_performance.txt`

# [Model Description]

## Training Performance

| Parameters                 | Ascend 910                                                   | 
| -------------------------- | ------------------------------------------------------------ | 
| Model Version              | MVD: baseline + modal specific & modal share backbone + VIB | 
| Resource                   | Ascend 910; CPU 2.60GHz, 192cores; Memory 755G; OS Euler2.8  | 
| uploaded Date              | 12/19/2021 (month/day/year)                 | 
| MindSpore Version          | 1.3.0, 1.5.0                                             | 
| Dataset                    | SYSU-MM01, RegDB                              | 
| Training Parameters（SYSU-MM01） | Epochs=80, steps per epoch=695, batch_size = 64 | 
| Training Parameters（RegDB） | Epochs=80, steps per epoch=695, batch_size = 64 | 
| Optimizer                  | Adam                                                 | 
| Loss Function              | Softmax Cross Entropy + Triplet Loss                         | 
| outputs                    | feature vector + probability                              | 
| Loss                       | 1.7161                                     |  2.0663      |
| Speed                      | 830 ms/step (1pcs, PyNative Mode)               | 
| Total time                 | SYSU(1pcs, PyNative Mode) : about  13h; RegDB: about 3h30min | 
| Parameters (M)             | 161.9M                                     | 
| Checkpoint for Fine tuning | 329.2M (.ckpt file)                                     | 
| Scripts                    | [link](https://gitee.com/mindspore/models/tree/master/research/cv/MVD) |

## Evaluation Performance

| Parameters        | Ascend                                                      | 
| ----------------- | ----------------------------------------------------------- | 
| Model Version     | MVD: baseline + modal specific & modal share backbone + VIB |
| Resource          | Ascend 910; OS Euler2.8                                     | 
| Uploaded Date     | 12/19/2021 (month/day/year)                                 | 
| MindSpore Version | 1.5.0, 1.3.0                                                |
| Dataset           | SYSU-MM01, RegDB                                            |
| batch_size        | 64                                                          |
| outputs           | feature                                                     | 
| Accuracy          | See following 4 tables ↓                                    |                                                             

## SYSU-MM01 (all-search mode)

| Metric | Value(Pytorch) | Value(Mindspore, ) | Value(Mindspore, Ascend 910) |
| :----: | :------------: | :-------------------: | :--------------------------: |
| Rank-1 |     60.02%     |        60.08%         |            58.64%            |
|  mAP   |     58.80%     |        57.55%         |            57.57%            |

## SYSU-MM01 (indoor-search mode)

| Metric | Value(Pytorch) | Value(Mindspore, ) | Value(Mindspore, Ascend 910) |
| :----: | :------------: | :-------------------: | :--------------------------: |
| Rank-1 |     66.05%     |        69.57%         |            67.73%            |
|  mAP   |     72.98%     |        73.13%         |            73.08%            |

## RegDB(Visible-Thermal)

| Metric | Value(Pytorch) | Value(Mindspore,  --trial 1) | Value(Mindspore, Ascend 910, -- trial 1) |
| :----: | :------------: | :------------------------------: | :--------------------------------------: |
| Rank-1 |     73.20%     |              77.91%              |                  77.28%                  |
|  mAP   |     71.60%     |              72.35%              |                  72.44%                  |

## RegDB(Thermal-Visible)

| Metric | Value(Pytorch) | Value(Mindspore,  --trial 1) | Value(Mindspore, Ascend 910, --trial 1) |
| :----: | :------------: | :------------------------------: | :-------------------------------------: |
| Rank-1 |     71.80%     |              76.50%              |                 76.07%                  |
|  mAP   |     70.10%     |              71.37%              |                 70.37%                  |

***Note**: The aforementioned pytorch results can be seen in original [pytorch repo](https://github.com/FutabaSakuraXD/Farewell-to-Mutual-Information-Variational-Distiilation-for-Cross-Modal-Person-Re-identification).

# Description of Random Situation

- In `utils.py`, `IdentitySampler` used to sample different identities and images in both visible and infrared(thermal) modal, and we set random seed in `IndentitySampler`. This randomness will affect both inference part in `train.py` and in `eval.py`. Therefore small different rank-1 and mAP fluctuations(about 1%) between inference in `train.py` and `eval.py` may be seen even on the same training results.

- When testing on RegDB dataset, there is a `--trial` argument specifying which id to be selected; different  `--trial`  choosing may cause slight rank-1 mAP fluctuation.

# Reference

Please kindly cite the original paper references in your publications if it helps your research:

```text
@inproceedings{VariationalDistillation,
title={Farewell to Mutual Information Variational Distiilation for Cross-Modal Person Re-identification},
author={Xudong Tian and Zhizhong Zhang and Shaohui Lin and Yanyun Qu and Yuan Xie and Lizhuang Ma},
booktitle={Computer Vision and Pattern Recognition},
year={2021}
}
```

Please kindly reference the url of mindspore repository in your code if it helps your research and code.

# [ModelZoo Homepage]

Please check the official [homepage](https://gitee.com/mindspore/models).
