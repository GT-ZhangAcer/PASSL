# Getting Started

This page provides basic tutorials about the usage of PASSL.
For installation instructions, please see [INSTALL.md](INSTALL.md).

## Data Preparation
- Download the ImageNet dataset from the [link](http://www.image-net.org/).
- Then, and move validation images to labeled subfolders, using [the following shell script](https://raw.githubusercontent.com/soumith/imagenetloader.torch/master/valprep.sh)
- Create a symlink under PASSL/data/ as the following examples: ```ln -s $YOUR_ILSVRC2012_PATH data/ILSVRC2012```
- At last, the folder looks like:
    ```
    PASSL
    ├── configs
    ├── LICENSE
    ├── passl
    ├── tools
    ├── README.md
    └── data
        └── ILSVRC2012
            ├── train
            └── val
    ```
## Train existing methods

**Note**: The default learning rate in config files is for 8 GPUs. If using differnt number GPUs, the total batch size will change in proportion, you have to scale the learning rate following `new_lr = old_lr * new_ngpus / old_ngpus`.

### Train with single/multiple GPUs

```shell
python tools/train.py -c ${CONFIG_FILE} --num-gpus {GPUS} [optional arguments]
```
Optional arguments are:
- `--resume ${CHECKPOINT_FILE}`: Resume from a previous checkpoint file.
- `--pretrained ${PRETRAIN_WEIGHTS}`: Load pretrained weights for the backbone.

An example:
```shell
# checkpoints and logs saved in OUTPUTS=outputs/
python tools/train.py -c configs/moco/moco_v2_r50.yaml --num-gpus 8
```

### ImageNet Linear Classification

**First**, extract backbone weights:
```shell
python tools/extract_weight.py ${CHECKPOINT} --output ${WEIGHT_FILE}
```
Arguments:
- `CHECKPOINTS`: the checkpoint file of a PASSL method named as `epoch_*.pdparams`.
- `WEIGHT_FILE`: the output backbone weights file, e.g., `pretrains/moco_v2_r50.pdparams`.

**Next**, train and test linear classification:
```shell
# Train
python tools/train.py -c ${CLS_CONFIG_FILE} --pretrained ${WEIGHT_FILE} --num-gpus 8
# Evaluation
python tools/train.py -c configs/*clas_*.yaml --load ${CLS_WEIGHT_FILE} --evaluate-only --num-gpus 8
```
Augments:
- `CLS_CONFIG_FILE`: Use config files under "configs/benchmarks/\*_clas_\*/". Note that if you want to test benchmark MoCo v2, you have to use the config file named `configs/moco/*_clas_*.yaml`, e.g., `configs/moco/moco_clas_r50.yaml`.
- Optional arguments include:
    - `--resume ${CHECKPOINT_FILE}`: Resume from a previous checkpoint file.
The trained linear weights in conjuction with the backbone weights can be found at [MoCo v1 linear](https://passl.bj.bcebos.com/models/moco_v1_r50_clas.pdparams) and [MoCo v2 linear](https://passl.bj.bcebos.com/models/moco_v2_r50_clas.pdparams)
