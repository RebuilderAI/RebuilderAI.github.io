---
date: 2022-07-19
layout: post
title: 'NeRFactor: Neural Factorization of Shape and Reflectance Under an Unknown Illumination'
subtitle: NeRFactor paper review
description: SIGGRAPH ASIA 2021
image: https://user-images.githubusercontent.com/55485826/179668889-de983eb0-1483-473d-8de2-8269e372d2f2.png
category: Reconstruction, Re-lighting, Material editing
tags:
  - Vision
  - 3D reconstruction
  - Graphics
author: jgnooo
---

> **NeRFactor: Neural Factorization of Shape and Reflectance Under an Unknown Illumination**
> Xiuming Zhang, Pratul P. Srinivasan, Boyang Deng, Paul Debevec, William T. Freeman, Jonathan T. Barron
> **TOG 2021 (Proc. SIGGRAPH Asia)**

### Introduction
---
논문에서 중점적으로 다루는 문제는 어떤 Object의 geometry와 Material 속성을 Multi-view 이미지로부터 복원하는 것 입니다. 특히, Multi-view 이미지와 이미지들에 대응하는 Camera들을 입력으로 받았을 때, Ojbect의 Shape과 Spatial하게 변화하는 Reflectance를 복원하는 것 입니다. (_하나의 Unknown lighting condition을 가정_) 논문에서는 이 문제를 해결하기 위한 Key idea로, **NeRF**(Neural Radiance Fields, ECCV 2020)의 volumetric geometry를 surface representation으로 **Distill**하는것, 그리고 Reflectance와 environment lighting을 해결하는 것과 동시에 Geometry를 함께 Refine하는 것을 제시하였으며 **NeRFactor** representation이라는 것을 소개하였습니다.

NeRFactor(Neural Factorization)은 3D neural field라는 것을 복원하기위해 최적화됩니다. 여기서 3D neural field는 Surface normal, Light visibility, Albedo, Bidirectional Reflectance Distribution Functions(BRDFs)를 표현하며, 특별한 supervision없이 Re-rendering loss, Smoothness prior, Data-driven BRDF prior를 활용해 최적화합니다. 따라서, NeRFactor는 Multi-view 이미지(+ camera)를 입력으로 받아서 3D neural fields를 잘 표현할 수 있도록 MLP를 통해 최적화(Parameterized)되고, 이를 통해 Re-lighting, Material editing 등의 어플리케이션에 활용될 수 있습니다.

### Method
---
![main](https://user-images.githubusercontent.com/55485826/179673208-34d8cb40-2912-4446-a34d-58a36a357953.png)
    _그림 1) NeRFactor model_

##### Assuming
>    1. Input: Multi-view 이미지 + Camera
>    2. Output : Surface normal, Light visibility, Albedo, Reflectance
>    3. One unknown illumination condition

##### Shape
---
Input으로 Multi-view 이미지와 Camera를 입력으로 받으면, NeRF(최적화 완료된 상태)를 통해 Initial geometry를 먼저 계산합니다. 위에서 언급한대로, 최적화된 NeRF를 통해 Density를 계산하고 Continuous surface representation으로써 사용하게 됩니다. 자세하게 설명하면, NeRF를 통해 Density를 계산하고 이를 통해 Depth(Distance t)를 계산한 후에 Surface point를 계산합니다 (아래 식).

![surface_point](https://user-images.githubusercontent.com/55485826/179676591-f9e390f5-7324-4a42-917d-98ad51cdeb4a.png)
    _식 1) Surface point_

Surface point는 그림 1)에서처럼 Visibility, BRDF, Albedo, Normal을 계산하기위한 MLP의 입력으로 사용됩니다.

먼저, Normal을 구하기 위해 NeRF의 density의 Gradient를 계산합니다. 이 방법을 통해 Normal을 계산하면 그림 2)와 같이 Artifact가 발생하기 때문에 Normal MLP를 통해 Re-parameterize하고, 식 2)를 통해 최적화됩니다.

![normal](https://user-images.githubusercontent.com/55485826/179677256-efc66c4d-611c-4b29-9369-43861367c9b9.png)
    _그림 2) Surface normal_

![normal_loss](https://user-images.githubusercontent.com/55485826/179677722-6624448c-6537-49d1-ba0c-ebf4f27f891c.png)
    _식 2) Normal loss function_

두번째로, Visibility를 NeRF의 density로부터 계산합니다. 마찬가지로, 그림 3)과 같이 Noise가 발생하기 때문에 Visibility MLP를 통해 Re-parameterize하고, 식 3)을 통해 최적화됩니다.

![visibility](https://user-images.githubusercontent.com/55485826/179678594-5592db97-1179-4e6b-93fc-79c6b8cb2411.png)
    _그림 3) Light visibility_

![visibility_loss](https://user-images.githubusercontent.com/55485826/179678584-c10db7ca-9a70-48f3-8fe8-7cbc2747ace4.png)
    _식 3) Visibility loss function_

##### Reflectance
---
Reflectance를 모델링하기 위해, 먼저 Albedo MLP를 통해 입력 Surface location에서의 Albedo를 학습하였습니다.

![albedo_loss](https://user-images.githubusercontent.com/55485826/179681138-46331815-9df8-4464-a7da-1c8b13801715.png)
    _식 4) Albedo loss_

다음으로, BRDF를 표현하기위해 먼저 그림 1)에서처럼 BRDF identity MLP를 통해 Latent code를 계산하고, BRDF MLP의 입력으로 사용됩니다. BRDF MLP는 Generative Latent Optimization approach (MERL 데이터셋으로 Pre-trained)를 활용해 Latent code와 Albedo 그리고 입사각과 반사각을 Rusinkiewicz 좌표로 변환하여 입력으로 받고 Reflectance를 계산합니다.

##### Rendering
---
지금까지 과정을 통해 계산한, Surface normal, Visibility, Albedo, BRDF, Lighting을 이용해 최종 Color를 Rendering합니다. Rendering은 Rendering equation을 통해 계산하며 식은 아래와 같습니다.

![rendering_eq](https://user-images.githubusercontent.com/55485826/179683499-2109de78-7254-47a9-9cc7-2b056b0da4de.png)
    _식 5) Rendering equation_

최종 Loss는 식 1) ~ 식 4)를 더한 Reconstuction loss로 활용됩니다.