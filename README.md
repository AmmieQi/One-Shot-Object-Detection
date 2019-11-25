# One-Shot Object Detection with Co-Attention and Co-Excitation

## Introduction

![Image](images/method.png)

This project is a pure pytorch implementation of One-Shot Objection. A lot of code is modified from [jwyang/faster-rcnn.pytorch](https://github.com/jwyang/faster-rcnn.pytorch).  

[**One-Shot Object Detection with Co-Attention and Co-Excitation**]()  
Ting-I Hsieh, Yi-Chen Lo, Hwann-Tzong Chen, Tyng-Luh Liu.  
Neural Information Processing Systems (NeurIPS), 2019

### What we are doing and going to do

- [x] Support tensorboardX.
- [x] Support pytorch-1.0 (this branch).
- [x] Upload the ImageNet pre-trained model.
- [x] Provide checkpoint model.
- [ ] Train PASCAL_VOC datasets

## Preparation

First of all, clone the code

```bash
git clone https://github.com/timy90022/One-Shot-Object-Detection.git
```

### 1. Prerequisites

* Python or 3.6
* Pytorch 1.0

### 2. Data Preparation

* **COCO**: Please also follow the instructions in [py-faster-rcnn](https://github.com/rbgirshick/py-faster-rcnn#beyond-the-demo-installation-for-training-and-testing-models) to prepare the data.
e scripts provided in this repository.

### 3. Pretrained Model

We used two pretrained models in our experiments, ResNet50. This pretrained remove all
COCO-related ImageNet classes by matching the WordNet synsets of ImageNet classes to COCO
classes, resulting in 933,052 images from the remaining 725 classes, while the original one contains 1,284,168 images of 1000 classes. You can download the models from:

* ResNet50: [Google Drive](https://drive.google.com/file/d/1SL9DDezW-neieqxWyNlheNefwgLanEoV/view?usp=sharing)

Download and unzip them into the ../data/

### 4. Reference image

The reference image only crop out the patches that are enclosed by the predicted bounding boxes of Mask R-CNN and the bounding boxes need to measure the following conditions.

* The IOU threshold    > 0.5
* The score confidence > 0.7

You can download the models from:
* Reference file: [Google Drive](https://drive.google.com/file/d/1O1AQtjozgpdtuETGE6X4UItpqcVPUiXH/view?usp=sharing)

Download and unzip them into the ../data/

### 5. Compilation

This part is the same as the [jwyang/faster-rcnn.pytorch](https://github.com/jwyang/faster-rcnn.pytorch).
Install all the python dependencies using pip:

```bash
pip install -r requirements.txt
```

Compile the cuda dependencies using following simple commands:

```bash
cd lib
python setup.py build develop
```

It will compile all the modules you need, including NMS, ROI_Pooing, ROI_Align and ROI_Crop. The default version is compiled with Python 2.7, please compile by yourself if you are using a different python version.

**As pointed out in this [issue](https://github.com/jwyang/faster-rcnn.pytorch/issues/16), if you encounter some error during the compilation, you might miss to export the CUDA paths to your environment.**

## Train

Before training, set the right directory to save and load the trained models. Change the arguments "save_dir" and "load_dir" in trainval_net.py and test_net.py to adapt to your environment.

In coco dataset, we split it to 4 group. It will train and test different category. Just to adjust "*--g*".

If you want to train part of dataset, try to modify "*--seen*". When training, use 1 to see train_categories.  When testing, use 2 to see test_categories. If you want to see both, use 3 to seen all categories.

To train a model with res50 on coco, simply run:

```bash
CUDA_VISIBLE_DEVICES=$GPU_ID python trainval_net.py \
                   --dataset coco --net res50 \
                   --bs $BATCH_SIZE --nw $WORKER_NUMBER \
                   --lr $LEARNING_RATE --lr_decay_step $DECAY_STEP \
                   --cuda --g $SPLIT --seen $SEEN
```

Above, BATCH_SIZE and WORKER_NUMBER can be set adaptively according to your GPU memory size. **On NVIDIA V100 GPUs with 32G memory, it can be up to 16 batch size**.

If you have multiple (say 8) V100 GPUs, then just use them all! Try:

```bash
python trainval_net.py --dataset coco --net res50 \
                       --bs $BATCH_SIZE --nw $WORKER_NUMBER \
                       --lr $LEARNING_RATE --lr_decay_step $DECAY_STEP \
                       --cuda --g $SPLIT --seen $SEEN --mGPUs

```

## Test

If you want to evlauate the detection performance of res50 model on coco test set.

You can train by yourself or download the models from [Google Drive](https://drive.google.com/file/d/1O1AQtjozgpdtuETGE6X4UItpqcVPUiXH/view?usp=sharing) and unzip them into the ```./models/res50/```.

Simply run:

```bash
python test_net.py --dataset coco --net res50 \
                   --checksession $SESSION --checkepoch $EPOCH --checkpoint $CHECKPOINT \
                   --cuda --g $SPLIT
```

Specify the specific model session, chechepoch and checkpoint, e.g., SESSION=1, EPOCH=6, CHECKPOINT=416.

## Acknowledgments

Code is modified from [jwyang/faster-rcnn.pytorch](https://github.com/jwyang/faster-rcnn.pytorch) and [AlexHex7/Non-local_pytorch](https://github.com/AlexHex7/Non-local_pytorch). All credit is attributed to them.
