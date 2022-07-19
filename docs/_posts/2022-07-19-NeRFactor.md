---
date: 2022-07-19
layout: post
title: 'NeRFactor: Neural Factorization of Shape and Reflectance Under an Unknown Illumination'
subtitle: NeRFactor paper review
description: SIGGRAPH ASIA 2021
image: https://user-images.githubusercontent.com/55485826/179668889-de983eb0-1483-473d-8de2-8269e372d2f2.png
category: Reconstruction, Re-lighting, Material editing
tags:
  - vision
  - 3d reconstruction
  - graphics
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
---
>    1. Input: Multi-view 이미지 + Camera
>    2. Output : Surface normal, Light visibility, Albedo, Reflectance
>    3. One unknown illumination condition