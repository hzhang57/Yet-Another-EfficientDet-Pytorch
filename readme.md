# Yet Another EfficientDet Pytorch

The pytorch re-implement of the official [EfficientDet](https://github.com/google/automl/efficientdet) with SOTA performance, original paper link: https://arxiv.org/abs/1911.09070


# Pretrained weights and performance

The performance is atmost 1.5% lower than the paper's, is close enough. 

| coefficient | pth_download | onnx_download | mAP 0.5:0.95(This Repo) | mAP 0.5:0.95(paper) |
| :----------: | :--------: | :-----------: | :--------: | :-----: |
| D0 | [efficientdet-d0.pth](https://github.com/zylo117/Yet-Another-Efficient-Pytorch/releases/download/1.0/efficientdet-d0.pth) | pending | 32.6 | 33.8
| D1 | [efficientdet-d1.pth](https://github.com/zylo117/Yet-Another-Efficient-Pytorch/releases/download/1.0/efficientdet-d1.pth) | pending | 38.2 | 39.6
| D2 | [efficientdet-d2.pth](https://github.com/zylo117/Yet-Another-Efficient-Pytorch/releases/download/1.0/efficientdet-d2.pth) | pending | 41.5 | 43.0
| D3 | [efficientdet-d3.pth](https://github.com/zylo117/Yet-Another-Efficient-Pytorch/releases/download/1.0/efficientdet-d3.pth) | pending | 44.9 | 45.8
| D4 | [efficientdet-d4.pth](https://github.com/zylo117/Yet-Another-Efficient-Pytorch/releases/download/1.0/efficientdet-d4.pth) | pending | 48.1 | 49.4
| D5 | [efficientdet-d5.pth](https://github.com/zylo117/Yet-Another-Efficient-Pytorch/releases/download/1.0/efficientdet-d5.pth) | pending | 49.5 | 50.7
| D6 | soon | pending | soon | 51.7
| D7 | soon | pending | soon | 52.2


# FAQ:

**Q1. Why implement this while there are several efficientdet pytorch projects already.**

A1: Because AFAIK none of them fully recovers the true algorithm of the official efficientdet, that's why their communities could not achieve or having a hard time to achieve the same score as the official efficientdet by training from scratch.

**Q2: What exactly is the difference among this repository and the others?**

A2: For example, this two are the most popular efficientdet-pytorch,

https://github.com/toandaominh1997/EfficientDet.Pytorch

https://github.com/signatrix/efficientdet

Here is the issues and why these are difficult to achieve the same score as the official one:

The first one:

1. Altered EfficientNet the wrong way, strides have been changed to adapt the BiFPN, but we should be aware that efficientnet's great performance comes from it's specific parameters combinations. Any slight alteration could lead to worse performance.

The second one:

1. Pytorch's BatchNormalization is slightly different from TensorFlow, momentum_pytorch = 1 - momentum_tensorflow. Well I didn't realize this trap if I paid less attentions. signatrix/efficientdet succeeded the parameter from TensorFlow, so the BN will perform badly by being dominated by the running mean and the running variance.

2. Mis-implement of Depthwise-Separable Conv2D. Depthwise-Separable Conv2D is Depthwise-Conv2D and Pointwise-Conv2D and BiasAdd ,there is only a BiasAdd after two Conv2D, while signatrix/efficientdet has a extra BiasAdd on Depthwise-Conv2D.

3. Misunderstand the first parameter of MaxPooling2D, the first parameter is kernel_size, instead of stride.

4. Missing BN after downchannel of the feature of the efficientnet output.

5. Using the wrong output feature of the efficientnet. This is big one. It takes whatever output that has the conv.stride of 2, but it's wrong. It should be the one whose next conv.stride is 2 or the final output of efficientnet.

6. Does not apply same padding on Conv2D and Pooling.

7. Missing swish activation after several operations.

8. Missing Conv/BN operations in BiFPN, Regressor and Classifier. This one is very tricky, if you don't dig deeper into the official implement, there are some same operations with different weights.

        
        illustration of a minimal bifpn unit
            P7_0 -------------------------> P7_2 -------->
               |-------------|                ↑
                             ↓                |
            P6_0 ---------> P6_1 ---------> P6_2 -------->
               |-------------|--------------↑ ↑
                             ↓                |
            P5_0 ---------> P5_1 ---------> P5_2 -------->
               |-------------|--------------↑ ↑
                             ↓                |
            P4_0 ---------> P4_1 ---------> P4_2 -------->
               |-------------|--------------↑ ↑
                             |--------------↓ |
            P3_0 -------------------------> P3_2 -------->
        
    For example, P4 will downchannel to P4_0, then it goes P4_1, 
    anyone may takes it for granted that P4_0 goes to P4_2 directly, right?
    
    That's why they are wrong, 
    P4 should downchannel again with a different weights to P4_0_another, 
    then it goes to P4_2.
    
And finally some common issues, their anchor decoder and encoder are different from the original one, but it's not the main reason that it performs badly.

Also, Conv2dStaticSamePadding from [EfficientNet-PyTorch](https://github.com/lukemelas/EfficientNet-PyTorch) does not perform like TensorFlow, the padding strategy is different. So I implement a real tensorflow-style [Conv2dStaticSamePadding](efficientnet/utils_extra.py#L9) and [MaxPool2dStaticSamePadding](efficientnet/utils_extra.py#L55) myself.

Despite of the above issues, they are great repositories that enlighten me, hence there is this repository.

This repository is mainly based on [efficientdet](https://github.com/signatrix/efficientdet), with the changing that makes sure that it performs as closer as possible as the paper.

Btw, debugging static-graph TensorFlow v1 is really painful. Don't try to export it with automation tools like tf-onnx or mmdnn, they will only cause more problems because of its custom/complex operations. 

And even if you succeeded, like I did, you will have to deal with the crazy messed up machine-generated code under the same class that takes more time to refactor than translating it from scratch.

___
# Update log

[2020-04-08] update coco eval script and simple inference script.

[2020-04-07] tested D0-D5 mAP, result seems nice. details can be found [here](benchmark/coco_eval_result)

[2020-04-07] fix anchors strategies, please pull the latest code to apply the fix.

[2020-04-06] adapt anchor strategies

[2020-04-05] finished re-implement efficientdet
 
# Demo

    # install requirements
    pip install pycocotools numpy opencv-python tqdm
    pip install torch==1.4.0
    pip install torchvision==0.5.0
     
    # run the simple inference script
    python efficientdet_test.py
    
# TODO

- [X] re-implement efficientdet
- [X] adapt anchor strategies
- [X] mAP tests
- [ ] training-scripts
- [ ] tensorflow's consistency tuning with pytorch. 
- [ ] efficientdet D6/D7 supports (this should be very easy)
- [ ] onnx support
- [ ] float16 mode
- [ ] float16 mode mAP tests
- [ ] tensorRT/TVM support
- [ ] re-implement tensorflow's weird bilinear interpolation algorithm in python, then cython.
    
# Known issues

1. Official EfficientDet use TensorFlow bilinear interpolation to resize image inputs, while it is different from many other methods (opencv/pytorch), so the output is definitely slightly different from the official one. But when I copy the input that generated by TensorFlow bilinear interpolation, the result is still slightly different. I will test mAP later, if this cause performance loss, either finetune the model, or figure out the divergence and fix it. But the benchmark uphere is tested when images are interpolated by opencv's bilinear algorithm, and the result seems nice.

2. D6/D7 BiFPN use unweighted sum for training stability, I'm working a fix to adapt to D6/D7.
    
# Comparison

Conclusion: They are almost the same. Believe it or not, you can check the tensor output, the difference is within 1e-3. 

I wonder where does this difference come from, and I'd like some help too.

## This Repo
<img src="https://raw.githubusercontent.com/zylo117/Yet-Another-Efficient-Pytorch/master/test/img_inferred_d0_this_repo.jpg" width="640">

## Official EfficientDet
<img src="https://raw.githubusercontent.com/zylo117/Yet-Another-Efficient-Pytorch/master/test/img_inferred_d0_official.jpg" width="640">

## Donation

If you like this repository, or if you'd like to support the author for any reason, you can donate to the author. Feel free to send me your name or introducing pages, I will make sure your name(s) on the sponsors list. 

<img src="https://raw.githubusercontent.com/zylo117/Yet-Another-Efficient-Pytorch/master/res/alipay.jpg" width="360">