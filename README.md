# CCNet: Criss-Cross Attention for Semantic Segmentation
By [Zilong Huang](http://speedinghzl.github.io), [Xinggang Wang](http://www.xinggangw.info/index.htm), [Lichao Huang](https://scholar.google.com/citations?user=F2e_jZMAAAAJ&hl=en), [Chang Huang](https://scholar.google.com/citations?user=IyyEKyIAAAAJ&hl=zh-CN), [Yunchao Wei](https://weiyc.github.io/), [Wenyu Liu](http://mclab.eic.hust.edu.cn/MCWebDisplay/PersonDetails.aspx?Name=Wenyu%20Liu).

This code is a implementation of the experiments on Cityscapes in the [CCNet](https://arxiv.org/abs/1811.11721). 
We implement our method based on open source [pytorch segmentation toolbox](https://github.com/speedinghzl/pytorch-segmentation-toolbox). 

**Update on 2018/12/10. Renew the code and release trained models with R=1,2. The trained model with R=2 achieves 79.74% on val set and 79.01% on test set with single scale testing.** 

**Update on 2018/11/28. Release Code.**

## Introduction
![motivation of CCNet](https://user-images.githubusercontent.com/4509744/50546460-7df6ed00-0bed-11e9-9340-d026373b2cbe.png)
Long-range dependencies can capture useful contextual information to benefit visual understanding problems. In this work, we propose a Criss-Cross Network (CCNet) for obtaining such important information through a more effective and efficient way. Concretely, for each pixel, our CCNet can harvest the contextual information of its surrounding pixels on the criss-cross path through a novel criss-cross attention module. By taking a further recurrent operation, each pixel can finally capture the long-range dependencies from all pixels. Overall, our CCNet is with the following merits: 
- **GPU memory friendly**  
- **High computational efficiency** 
- **The state-of-the-art performance** 

## Architecture
![Overview of CCNet](https://user-images.githubusercontent.com/4509744/50546462-851dfb00-0bed-11e9-962a-bffab2401997.png)
Overview of the proposed CCNet for semantic segmentation. The proposed recurrent criss-cross attention takes as input feature maps **H** and output feature maps **H''** which obtain rich and dense contextual information from all pixels. Recurrent criss-cross attention module can be unrolled into R=2 loops, in which all Criss-Cross Attention modules share parameters.

## Visualization of the attention map
![Overview of Attention map](https://user-images.githubusercontent.com/4509744/50546463-88b18200-0bed-11e9-9f87-c74327c11a68.png)
To get a deeper understanding of our RCCA, we visualize the learned attention masks as shown in the figure.  For each input image, we select one point (green cross) and show its corresponding attention maps when **R=1** and **R=2** in columns 2 and 3 respectively. In the figure, only contextual information from the criss-cross path of the target point is capture when **R=1**. By adopting one more criss-cross module, ie, **R=2**  the RCCA can finally aggregate denser and richer contextual information compared with that of **R=1**. Besides, we observe that the attention module could capture semantic similarity and long-range dependencies. 

### License

CCNet is released under the MIT License (refer to the LICENSE file for details).

### Citing CCNet

If you find CCNet useful in your research, please consider citing:

    @article{huang2018ccnet,
        title={CCNet: Criss-Cross Attention for Semantic Segmentation},
        author={Huang, Zilong and Wang, Xinggang and Huang, Lichao and Huang, Chang and Wei, Yunchao and Liu, Wenyu},
        journal = {arXiv preprint arXiv:1811.11721},
        year={2018}
    }
    
### Requirements

To install PyTorch==0.4.0 or 0.4.1, please refer to https://github.com/pytorch/pytorch#installation.   
4 x 12G GPUs (_e.g._ TITAN XP)  
Python 3.6   
gcc (GCC) 4.8.5  
CUDA 8.0  

### Compiling

Some parts of **InPlace-ABN** and **Criss-Cross Attention** have native CUDA implementations, which must be compiled with the following commands:
```bash
cd libs
sh build.sh
python build.py

cd ../cc_attention
sh build.sh
python build.py
``` 
The `build.sh` script assumes that the `nvcc` compiler is available in the current system search path.
The CUDA kernels are compiled for `sm_50`, `sm_52` and `sm_61` by default.
To change this (_e.g._ if you are using a Kepler GPU), please edit the `CUDA_GENCODE` variable in `build.sh`.

### Dataset and pretrained model

Plesae download cityscapes dataset and unzip the dataset into `YOUR_CS_PATH`.

Please download MIT imagenet pretrained [resnet101-imagenet.pth](http://sceneparsing.csail.mit.edu/model/pretrained_resnet/resnet101-imagenet.pth), and put it into `dataset` folder.

### Training and Evaluation
Training script.
```bash
python train.py --data-dir ${YOUR_CS_PATH} --random-mirror --random-scale --restore-from ./dataset/resnet101-imagenet.pth --gpu 0,1,2,3 --learning-rate 1e-2 --input-size 769,769 --weight-decay 1e-4 --batch-size 8 --num-steps 60000 --recurrence 2
``` 

【**Recommend**】You can also open the OHEM flag to reduce the performance gap between val and test set.
```bash
python train.py --data-dir ${YOUR_CS_PATH} --random-mirror --random-scale --restore-from ./dataset/resnet101-imagenet.pth --gpu 0,1,2,3 --learning-rate 1e-2 --input-size 769,769 --weight-decay 1e-4 --batch-size 8 --num-steps 60000 --recurrence 2 --ohem 1 --ohem-thres 0.7 --ohem-keep 100000
``` 

Evaluation script.
```bash
python evaluate.py --data-dir ${YOUR_CS_PATH} --restore-from snapshots/CS_scenes_60000.pth --gpu 0 --recurrence 2
``` 

All in one.
```bash
./run_local.sh YOUR_CS_PATH
``` 


### Models
We run CCNet with *R=1,2* three times on cityscape dataset separately and report the results in the following table.
Please note there exist some problems about the validation/testing set accuracy gap (1~2%). You need to run multiple times
to achieve a small gap or turn on OHEM flag. Turning on OHEM flag also can improve the performance on the val set. In general，
I recommend you use OHEM in training step.

We train all the models on fine training set and use the single scale for testing.
The trained model with **R=2 79.74**  can also achieve about **79.01** mIOU on **cityscape test set** with single scale testing (for saving time, we use the whole image as input).

| **R** | **mIOU on cityscape val set (single scale)**           | **Link** |
|:-------:|:---------------------:|:---------:|
| 1 | 77.31 & **77.91** & 76.89 | [77.91](https://drive.google.com/open?id=13j06I4e50T41j_2HQl4sksrLZihax94L) |
| 2 | **79.74** & 79.22 & 78.40 | [79.74](https://drive.google.com/open?id=1IxXm8qxKmfDPVRtT8uuDNEvSQsNVTfLC) |
| 2+OHEM | 78.67 & **80.00** & 79.83  | [80.00](https://drive.google.com/open?id=1eiX1Xf1o16DvQc3lkFRi4-Dk7IBVspUQ) |

## Acknowledgment
The work was mainly done during an internship at [Horizon Robotics](http://en.horizon.ai/).

## Thanks to the Third Party Libs
Self-attention related methods:   
[Object Context Network](https://github.com/PkuRainBow/OCNet)    
[Dual Attention Network](https://github.com/junfu1115/DANet)   
Semantic segmentation toolboxs:   
[pytorch-segmentation-toolbox](https://github.com/speedinghzl/pytorch-segmentation-toolbox)   
[semantic-segmentation-pytorch](https://github.com/CSAILVision/semantic-segmentation-pytorch)   
[PyTorch-Encoding](https://github.com/zhanghang1989/PyTorch-Encoding)
