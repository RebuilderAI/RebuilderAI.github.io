---
date: 2022-08-02 06:00:00  
layout: post  
title: Semantic Segmentation for Point Clouds
subtitle: Segmentation in 3D space
description: Semantic segmentation survey
image:  https://user-images.githubusercontent.com/36464983/182502852-41f3fcf4-31d9-466a-afe5-f76c67ac2c1b.png
category: 3D Semantic Segmentation
tags:
  - Semantic segmentation
  - 3D vision
  - Multi-view stereo
author: ansh941
---

# Semantic Segmentation for 3D Point clouds
최근 이미지/비디오 기반으로 2D space에서의 salient object detection 프로젝트를 진행하고 있습니다. 하지만 단순히 2D 이미지를 사용해 detection 하는 것은 최대한 적은 수의 이미지로 균일하게 segmentation 하는 저희의 목표에 도달하기엔 한계가 있다 판단했습니다. 따라서 저희는 기존의 모델에 3D의 knowledge를 도입 또는 3D에서의 segmentation을 진행하고자 하고 있습니다. 그 과정에서 3D point cloud에서의 Segmentation을 고려해보고자 리서치를 진행했고, 해당 분야에 대한 전반적인 부분을 공유하고자 합니다.
![taxonomy.png](https://user-images.githubusercontent.com/36464983/182502852-41f3fcf4-31d9-466a-afe5-f76c67ac2c1b.png)

**Deep Learning for 3D Point Clouds: A Survey[1]**에 따르면 3D point clouds를 위한 딥러닝 기법들은 다음와 같이 분류됩니다. 크게는 Classification, Object detection and tracking, Segmentation으로 나뉘는데, 여기서는 그 중에서 Segmentation에 대해서만 다루겠습니다.

# 3D point cloud Segmentation

## 1. Datasets
3D point cloud segmentation을 위한 데이터셋들은 각기 다른 타입의 센서들<sup id="a1">[1](#f1)</sup>에 의해 얻어집니다. 이러한 데이터셋들은 다양한 challenge들<sup id="a2">[2](#f2)</sup>의 알고리즘 개발을 위해 사용됩니다. 데이터셋 목록은 아래와 같습니다.
![datasets.png](https://user-images.githubusercontent.com/36464983/182503092-9124ffbb-548a-4606-a42c-da11e6d220d3.png)

## 2. Evaluation Metrics
3D point cloud segmentation의 경우에는 OA(Overall Accuracy), mIoU(mean Intersection over Union)의 2가지가 성능 평가에 가장 자주 사용되는 metric 입니다.<br>
각 수식은 아래와 같습니다.<br>
$OA = {\sum_{i=1}^N c_{ii} \over \sum_{j=1}^N\sum_{k=1}^N c_{jk}}$,<br>
여기서 N은 클래스의 수이며, $c_{ii}, c_{jk}$는 $N \times N$ confusion matrix에서의 값.[2]<br>
$mIoU = {1 \over C}\sum_{c=1}^C {TP_c \over TP_c + FP_c + FN_c}$<br>
여기서 $C$는 클래스의 수이며, $TP_c,\ FP_c,\ FN_c$는 Confusion matrix에서 각 class $c$의 True Positive, False Positive, False Negative.[3]<br>

## 3. Methods
3D point cloud segmentation은 global geometric structure와 각 점의 fine-grained details에 대한 이해가 필요합니다.
Segmentation 단위에 따라 3D point cloud segmentation은 3가지 분류로 나눠집니다.

1. Semantic segmentation(Scene level)
2. Instance Segmentation(Object level)
3. Part Segmentation(Part level)

이번에는 그 중 Semantic segmentation에 대해서만 다룹니다.

## 3.1 3D Semantic segmentation
point cloud가 주어졌을 때, semantic segmentation의 목표는 각 point를 의미별 subset으로 나누는 것입니다. 이 또한 패러다임에 따라 4가지로 분류할 수 있습니다.

1. Projection-based
2. Discretization-based
3. Point-based
4. Hybrid-based

projection과 discretization 기반 기법들의 첫 걸음은 point cloud를 intermediate regular representation<sup id="a3">[3](#f3)</sup>으로 바꾸는 것입니다. 예를 들면  이 있습니다. 그 이후 intermediate segmentation result를 다시 raw point cloud에 투영합니다. 하지만 point-based 기법들은 불규칙한 point cloud에서 바로 진행됩니다.<br>
![Representations.png](https://user-images.githubusercontent.com/36464983/182503294-93e5f63f-47c8-46f1-a5a7-dad1b215d29a.png)

### 3.1.1 Projection-based Methods
이 기법들은 일반적으로 3D point cloud를 2D images로 투영하는 것이며, multi-view 및 spherical image를 포함합니다.<br>

**Multi-view Representation**<br>
이 기법들은 대부분 가상의 multi-view point를 선정하고 3D point cloud를 해당 view point에 투영하여 2D segmentation 모델을 이용합니다. 위 그림의 (a)에서 multi-view representation을 간단하게 나타내고 있습니다.<br>
먼저 3D point cloud를 2D에 투영하고 그 이미지를 segmentation model에 넣어 pixel별 score를 얻고 각기 다른 view에 대해 reprojection된 점수를 융합하는 방법이 있습니다[4]. 이와 비슷하게, multiple camera position을 이용해 point cloud의 RGB와 depth snapshot을 생성하고, 이렇게 만든 snapshot을 2D segmentation network를 이용해 pixel-wise labeling을 수행하는 방법이 있습니다[5]. 이 방법에서는 RGB와 depth image를 이용해 예측된 score는 residual correction을 이용해 융합됩니다.<br>
또 다른 방법으로는 point cloud가 locally euclidean surface에서 샘플링되었다는 가정을 기반으로 tangent convolution을 사용한 방법이 있습니다[6]. 이 기법은 각 점 주변의 local surface geometry를 가상의 tangent plane으로 투영하며, 그 이후에 tangent convolution은 surface geometry에서 직접적으로 연산됩니다. 이 기법은 확장성이 좋으며, 수많은 점들로 이뤄진 large-scale point cloud 또한 처리할 수 있음을 보입니다.<br>
전반적으로, multi-view segmentation 기법들은 viewpoint selection과 occlusion에 예민합니다. 또한, 이러한 방법들은 필연적으로 투영 과정에서 정보의 손실을 불러오기 때문에 geometric and structural information을 잘 활용하지 못했습니다.<br>

**Spherical Representation**<br>
이 기법들은 2D CNN에 3D point cloud를 넣기 위해, spherical projection으로 3D point cloud를 2D grid representation으로 투영합니다[7,8]. sparse하고 irregular한 point cloud를 그대로 사용하면 비효율적이고 계산을 낭비하게 되는데, 이러한 문제들을 방지하고 좀 더 압축된 representation을 얻고자 sphere 좌표계로 투영합니다. spherical projection은 single view projection에 비해 더 많은 정보가 유지되며 sparse하고 irregular한 lidar point cloud의 labeling에 적합합니다. 하지만 이러한 intermediate representation은 discretization error와 occlusion 문제를 피할 수 없습니다.<br>
위 그림의 (b)에서 spherical representation을 간단히 보여주고 있습니다.<br>

### 3.1.2 Discretization-based Methods
이 기법들은 일반적으로 point cloud를 volumetric and sparse permutohedral lattices와 같은 dense/sparse discrete representation으로 변환하고 3D CNN 기반의 segmentation을 진행합니다.<br>

**Dense Discretization Representation**<br>
초기의 방법들은 일반적으로 point cloud를 dense-grid로 voxel화 하고 3D convolution을 활용했습니다.<br>
point cloud를 voxel 단위로 나눈 후 이렇게 생성된 중간 산출물를 3D CNN에 넣어 voxel-wise segmentation을 진행하는 방식으로, voxel 내 모든 point를 같은 label로 할당하는 것으로 마무리 됩니다[9]. 이러한 기법은 point cloud를 나누는 방식에 따른 voxel의 size와 경계의 artifact에 따라 성능이 심하게 제한됩니다. 이러한 문제를 해결하고 fine-grained 하면서 global consistent semantic segmentation을 이루기 위해, deterministic trilinear interpolation을 통해 coarse voxel 예측값을 3D-FCNN으로부터 생성된 point cloud에 매핑하고 FC-CRF를 이용한 모델 SEGCloud가 제안되었습니다[10].<br>
이후에는 각 voxel에서 local geometrical structure를 encoding 하기 위한 kernel-based interpolated VAE 아키텍처가 제안되었습니다[11]. binary occupancy representation을 사용하는 대신, RBF를 사용해 각 voxel에 대해 연속적인 representation을 얻고, 각 voxel 안 점들의 분포를 capture했습니다. VAE는 각 voxel의 point distribution을 compact latent space로 매핑하는데 사용됩니다. 또한, Group-CNN을 이용해 강건한 feature 학습을 달성합니다.<br>
전체적으로 volumetric representation은 자연스럽게 3D point cloud의 이웃한 structure를 그대로 사용할 수 있게 합니다. 이와 같은 regular data format은 3D convolution을 바로 이용할 수 있게 해줘 성능 향상에 도움이 되었습니다. 하지만 voxelization 과정은 discretization artifact와 information loss가 나타날 수 밖에 없습니다. 일반적으로 voxel의 high resolution은 높은 메모리와 계산 코스트를 갖게 하는 반면, low resolution은 detail의 loss를 일으킵니다. 따라서 적절한 grid resolution을 선택하는 것이 중요하게 작용합니다.<br>
위 그림의 (c)에서 설명한 representation을 나타내고 있습니다.<br>

**Sparse Discretization Representation**<br>
Volumetric representation은 0이 아닌 값의 수의 비율이 적기 때문에 자연스럽게 sparse 해집니다. 그러므로, dense CNN을 적용하는 것은 효율적이지 않습니다. 따라서 indexing structure 기반의 submanifold sparse convolutional network가 제안되었습니다[12]. 이 기법은 convolution의 output을 사용 중인 voxel에 한정하여 memory와 computational cost를 줄이는데 탁월한 효과를 보였습니다. 한편, 이러한 sparse convolution은 추출된 feature의 밀도를 제어할 수 있습니다. 이 submanifold sparse convolution은 고차원이면서 spatially-sparse한 데이터를 효과적으로 처리할 수 있습니다. <br>
이후 비슷한 느낌의 3D video perception을 위한 네트워크가 제안되었습니다[13]. generalized sparse convolution을 사용하는데 이는 고차원 데이터 처리에 뛰어납니다. 또한, trilateral-stationary conditional random field를 이용해 consistancy를 강화하였습니다.<br>
한편으로는, BCL(Bilateral Convolution Layer)를 사용한 네트워크가 제안되었습니다[14]. 이 기법은 raw point cloud를 permutohedral sparse lattice로 interpolate 하고, 그 후 BCL을 적용해 sparse하게 채워진 lattice의 사용 중인 부분을 convolve 합니다. 이렇게 나온 output은 다시 raw point cloud로 interpolation합니다. 추가적으로, 이 기법은 multi-view image와 point cloud의 joint processing을 flexible하게 만들어줍니다.<br>
그 이후에는 large point cloud를 효과적으로 처리할 수 있도록 하는 네트워크가 제안되었습니다[15].<br>

### 3.1.3 Hybrid Methods
사용 가능한 모든 정보를 더욱 효과적으로 활용하기 위해, 3D scan에서 multi modal을 학습하는 몇 가지 방법이 제안되었습니다. 2D CNN stream에서 추출한 RGB feature와 3D CNN stream에서 추출한 geometry feature를 합성하고 미분 가능한 back-projection layer를 사용한 joint 3D-multi-view network가 제안되었고[16], 2D 텍스처를 학습하기 위한 unified point-based framework가 제안되었습니다[17]. [17]은 voxelization 없이 sparse하게 샘플링된 point 집합에서 local geometric feature와 global context를 추출하기 위해 point-based network를 적용했습니다. 고전적인 point cloud space에서 2D multi-view image와 spatial geometric feature에서 얻은 feature들을 aggregate 하기 위한 Multi-view PointNet도 제안되었습니다[18].<br>

### 3.1.4 Point-based Methods
point-based network는 irregular point cloud를 바로 사용하는 것입니다. 하지만, point cloud가 순서가 없고, 체계적이지 않아 일반적인 CNN에 곧바로 적용하기엔 좋지 않습니다. 하지만 shared MLP를 통해 point 별 feature와 global feature를 학습하기 위핸 PointNet이 제안되었습니다. PointNet을 기반으로 point-based network들이 최근까지도 많이 제안되고 있습니다. 이러한 기법들은 rough하게 pointwiseMLP, point convolution, RNN-based, graph-based로 나뉩니다.<br>
Point-based Method는 그 자체로도 양이 많아 다음 포스팅에 이어 작성하겠습니다.<br>

# Reference
[1] Guo, Y., Wang, H., Hu, Q., Liu, H., Liu, L., & Bennamoun, M. (2020). Deep learning for 3d point clouds: A survey. *IEEE transactions on pattern analysis and machine intelligence*, *43*(12), 4338-4364.
[2] T. Hackel, N. Savinov, L. Ladicky, J. Wegner, K. Schindler, and M. Pollefeys, “[Semantic3D.net](http://semantic3d.net/): A new large-scale point cloud classification benchmark,” ISPRS, 2017. 
[3] J. Behley, M. Garbade, A. Milioto, J. Quenzel, S. Behnke, C. Stachniss, and J. Gall, “SemanticKITTI: A dataset for semantic scene understanding of lidar sequences,” in ICCV, 2019.
[4] F. J. Lawin, M. Danelljan, P. Tosteberg, G. Bhat, F. S. Khan, and M. Felsberg, “Deep projective 3D semantic segmentation,” in CAIP, 2017.
[5] A. Boulch, B. Le Saux, and N. Audebert, “Unstructured point cloud semantic labeling using deep segmentation networks.” in 3DOR, 2017.
[6] M. Tatarchenko, J. Park, V. Koltun, and Q.-Y. Zhou, “Tangent convolutions for dense prediction in 3D,” in CVPR, 2018.
[7] B. Wu, A. Wan, X. Yue, and K. Keutzer, “SqueezeSeg: Convolutional neural nets with recurrent crf for real-time road-object segmentation from 3D lidar point cloud,” in ICRA, 2018.
[8] B. Wu, X. Zhou, S. Zhao, X. Yue, and K. Keutzer, “SqueezeSegV2: Improved model structure and unsupervised domain adaptation for road-object segmentation from a lidar point cloud,” in ICRA, 2019.
[9] J. Huang and S. You, “Point cloud labeling using 3D convolutional neural network,” in ICPR, 2016.
[10] L. Tchapmi, C. Choy, I. Armeni, J. Gwak, and S. Savarese, “SEGCloud: Semantic segmentation of 3D point clouds,” in 3DV, 2017.
[11] H.-Y. Meng, L. Gao, Y.-K. Lai, and D. Manocha, “VV-Net: Voxel vae net with group convolutions for point cloud segmentation,” in ICCV, 2019.
[12] B. Graham, M. Engelcke, and L. van der Maaten, “3D semantic segmentation with submanifold sparse convolutional networks,” in CVPR, 2018.
[13] C. Choy, J. Gwak, and S. Savarese, “4D spatio-temporal convnets: Minkowski convolutional neural networks,” in CVPR, 2019.
[14] H. Su, V. Jampani, D. Sun, S. Maji, E. Kalogerakis, M.-H. Yang, and J. Kautz, “SplatNet: Sparse lattice networks for point cloud processing,” in CVPR, 2018.
[15] R.A.Rosu,P.Schu ̈tt,J.Quenzel,andS.Behnke,“LatticeNet:Fast point cloud segmentation using permutohedral lattices,” arXiv preprint arXiv:1912.05905, 2019.
[16] A. Dai and M. Nießner, “3DMV: Joint 3D-multi-view prediction for 3D semantic scene segmentation,” in ECCV, 2018.
[17] H.-Y. Chiang, Y.-L. Lin, Y.-C. Liu, and W. H. Hsu, “A unified point-based framework for 3D segmentation,” in 3DV, 2019.
[18] M. Jaritz, J. Gu, and H. Su, “Multi-view pointNet for 3D scene understanding,” in ICCVW, 2019.

# Footnote
<b id="f1"><sup>1</sup></b> MLS(Mobile Laser Scanners), ALS(Aerial Laser Scanners), TLS(Terrestrial Laser Scanners), RGB-D cameras, 그 외 다른 3D scanners<br>
<b id="f2"><sup>2</sup></b> Similar distractors, shape incompleteness, class imbalance 등<br>
<b id="f3"><sup>3</sup></b> multi-view, spherical, volumetric, permutohedral lattice, hybrid representation 등의 기법을 이용합니다.<br>
