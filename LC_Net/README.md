

## Installation

1. Install [PyTorch](https://pytorch.org/) (make sure to select the correct cuda version)
2. Install the requirements
```pip install -r requirements.txt```
3. Install [spconv](https://github.com/traveller59/spconv) (make sure that you have already selected the correct cuda version)
4. Install [OpenPCDet](https://github.com/open-mmlab/OpenPCDet)
5. Install [faiss-cpu](https://github.com/facebookresearch/faiss/blob/main/INSTALL.md) - NOTE: avoid installing faiss via pip, use the conda version, or build it from source alternatively.

We have provided the code implementations of the two dataset: KITTI and KITTI-360, respectively.

Tested in the following environments:
* Ubuntu 18.04/20.04
* cuda 10.2/11.1/11.3
* pytorch 1.8/1.9/1.10

## Preprocessing

### KITTI

Download the [SemanticKITTI](http://semantic-kitti.org/dataset.html#download) dataset and generate the loop ground truths:

```python -m data_process.generate_loop_GT_KITTI --root_folder KITTI_ROOT```

where KITTI_ROOT is the path where you downloaded and extracted the SemanticKITTI dataset.

NOTE: although the semantic labels are not required to run LCDNet, we use the improved ground truth poses provided with the SemanticKITTI dataset.

### KITTI-360

Download the [KITTI-360](http://www.cvlibs.net/datasets/kitti-360/download.php) dataset (raw velodyne scans, calibrations and vehicle poses) and generate the loop ground truths:

```python -m data_process.generate_loop_GT_KITTI360 --root_folder KITTI360_ROOT```

where KITTI360_ROOT is the path where you HAVE downloaded and extracted the KITTI-360 dataset.

### Optional: Ground Plane Removal

To achieve better results, it is suggested to preprocess the datasets by removing the ground plane:

```python -m data_process.remove_ground_plane_kitti --root_folder KITT_ROOT```

```python -m data_process.remove_ground_plane_kitti360 --root_folder KITT_ROOT360```

If you skip this step, please remove the option ```--without_ground``` in all the following steps.

## Training

The training script will use all the available GPUs, if you want to use only a subset of the GPUs, use the environment variable ```CUDA_VISIBLE_DEVICES```. If you don't know how to do that, check [here](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#env-vars).

To train on the KITTI dataset:

```python -m training_KITTI_DDP --root_folder KITTI_ROOT --dataset kitti --batch_size B --without_ground```

To train on the KITTI-360 dataset:

```python -m training_KITTI_DDP --root_folder KITTI360_ROOT --dataset kitti360 --batch_size B --without_ground```

To track the training progress using [Weights & Biases](https://wandb.ai/), add the argument ```--wandb```.
The per-GPU batch size B must be at least 2, and a GPU with at least 8GB of RAM is required (12GB or more is preferred). In our experiments we used a batch size of 6 on 4 x 24GB GPUs, for a total batch size of 24.

The network's weights will be saved in the folder ```./checkpoints``` (you can change this folder with the argument ```--checkpoints_dest```), inside a subfolder named with the starting date and time of the training (format ```%d-%m-%Y_%H-%M-%S```), for example: ```20-02-2022_16-38-24```

## Evaluation

### Loop Closure

To evaluate the loop closure performance of the trained model on the KITTI dataset:

```python -m evaluation.inference_loop_closure --root_folder KITTI_ROOT --dataset kitti --validation_sequence 08 --weights_path WEIGHTS --without_ground```

where WEIGHTS is the path of the pretrained model, for example ```./checkpoints/20-02-2022_16-38-24/checkpoint_last_iter.tar```

Similarly, on the KITTI-360 dataset:

```python -m evaluation.inference_loop_closure --root_folder KITTI360_ROOT --dataset kitti360 --validation_sequence 2013_05_28_drive_0002_sync --weights_path WEIGHTS --without_ground```

### Point Cloud Registration

To evaluate the loop closure performance of the trained model on the KITTI and KITTI-360 dataset:

```python -m evaluation.inference_yaw_general --root_folder KITTI_ROOT --dataset kitti --validation_sequence 08 --weights_path WEIGHTS --ransac --without_ground```

```python -m evaluation.inference_yaw_general --root_folder KITTI360_ROOT --dataset kitti360 --validation_sequence 2013_05_28_drive_0002_sync --weights_path WEIGHTS --ransac --without_ground```

To evaluate LCDNet in a very fast manner, you can just remove the ```--ransac``` argument.

## Pretrained Model

A model pretrained on the KITTI dataset can be found [here](https://entuedu-my.sharepoint.com/:f:/g/personal/kangcheng_liu_staff_main_ntu_edu_sg/Ejv6a1HYOsxMgPyvKKf2USsBp5pfV3cby2icvaMbbf-JoQ?e=QakXwd).
