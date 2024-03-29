---
date: 2022-09-14 06:00:00  
layout: post  
title: 3d 모델에서 반사를 표현하는 방법
subtitle: 기초부터 보는 PBR shading
description: PBR shading basics
image: https://user-images.githubusercontent.com/48865276/190091259-6b0d2185-fe10-441b-a048-261af8fd2353.jpg
category: 2D, 3D, shading, graphics
tags:
  - Image rendering
  - PBR shading
  - graphics
author: q10
---


# 3d 모델에서 반사를 표현하는 방법

### 부제: 기초부터 보는 PBR shading

이번주 R&D팀에서 소개할 내용은 3d 모델에서 반사를 표현하는 방법입니다.

3d 모델 파일은 obj, glb, fbx등 다양한 형식이 있습니다. 이를 게임이나 쇼핑몰 등에서 사용할 때에는 **렌더링(rendering)**이라는 과정을 거치게 됩니다.

렌더링에서 그림자, 재질 등을 표현하는 방법을 shading이라고 하고, 얼마나 현실의 빛을 잘 모방하는지에 따라 렌더링 이미지 퀄리티가 크게 달라지게 됩니다. 리빌더에이아이에서도 실사 에셋을 복원하고, 현실적인 가상공간 서비스 제공을 위해 shading 방법을 많이 고려하며 연구를 진행해 나가고 있습니다.

이번 글에서는 실제 3d 모델에 shading이 적용되어 우리 눈에 보이기까지의 과정을 간단하게 설명드리려고 합니다.

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/ff7b5396-1798-44b8-a16b-03eace5e5c6a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220914T064601Z&X-Amz-Expires=86400&X-Amz-Signature=59f78092f3405c25aca7ff4b7223ed10e4d554747c4314afc3e88b8ac7340f59&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)
    _그림 1) 리빌더에이아이 스캐닝 예시_



## 3D 모델을 어떻게 눈에 보이게 할 수 있을까?

눈에 보인다는 것은 3차원 공간을 2차원 이미지로 변환하는 과정입니다. 이 때 내 시점에 맞게 변환하는 과정이 필요하고, 카메라 좌표를 알고 있으면 이를 수학적으로 계산할 수 있습니다.

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/8e58af8d-23b5-48b8-904e-2e94f9cca9cf/2022-09-14_16-03-13.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220914T070438Z&X-Amz-Expires=86400&X-Amz-Signature=704d977640da97ee0a11d202c703ce63fda643de494d5c195f4d795133ab9bf7&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%222022-09-14_16-03-13.jpg%22&x-id=GetObject)
    _그림 2) 2D 이미지와 카메라 좌표계_ 



### 3d 모델이란 무엇일까?

3d 모델은 포인트클라우드 정보와 폴리곤 정보를 통해 표현됩니다.

- 이미지: (x, y) 좌표로 표현
- 3d pointcloud: (x, y, z) 좌표로 표현
- 3d mesh: (x, y, z) 좌표(pointcloud)와 폴리곤(face) 리스트로 표현
- obj 파일 구조: mesh.obj, mesh.mtl, texture.png

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a0653e0d-60da-424f-8746-9986f8695a0b/mesh-pointcloud-faces.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220914%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220914T071756Z&X-Amz-Expires=86400&X-Amz-Signature=fb997759b04b5429d79fe3495b72b2a23d8d4af0c24ab61845ae6ebcc36621e1&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22mesh-pointcloud-faces.png%22&x-id=GetObject)
    _그림 3) mesh 표현 방식: pointcloud + faces_ 



가장 대표적인 3d 모델 포맷인 obj 파일 구조를 뜯어보면 아래와 같습니다. 포인트클라우드에 대한 x,y,z 좌표(v)와 어떤 점들을 이을지에 대한 폴리곤 리스트(f)가 정의되어 있음을 확인할 수 있습니다. obj 파일 맨 첫 줄에는 어떤 mtl 파일을 사용할지(mtllib)가 정의되어 있습니다.

![2022-09-14_16-14-14](https://user-images.githubusercontent.com/48865276/190088451-fd6f42e4-a848-4ab4-909c-5b957ca5c40d.jpg)
![2022-09-14_16-15-15](https://user-images.githubusercontent.com/48865276/190088392-f62dce64-5af2-4a01-bf1c-8a7d63197e5b.jpg)
    _그림 4) obj 파일 구조_


mtl 파일에는 어떤 텍스처 정보를 사용할지가 정의되어 있습니다.

![2022-09-14_16-15-32](https://user-images.githubusercontent.com/48865276/190088595-e6fa0cb3-4d82-45e9-8d7c-cdaa3c747592.jpg)
    _그림 5) mtl 파일 예시_ 



### 3d 모델을 이미지로 변환하는 방법
3d 모델을 가지고 이미지를 렌더링하는 가장 기초적인 방법은 단순하게 내가 보는 시점으로 물체를 가져오는 것입니다. 실제 절대좌표 공간(world space)에서 카메라를 원점으로 하는 좌표계(camera space)로 좌표를 변환한 후, 이미지 평면에 그대로 투영하면 됩니다.

![2022-09-14_16-35-17](https://user-images.githubusercontent.com/48865276/190090816-c0ab0032-af5a-4f40-9557-db119b9d1f0b.jpg)
    _그림 6) 카메라 좌표변환_ 

camera space에서 image space로 투영할 때는 다양한 방법이 있지만, 아래처럼 NDC(Normalized Device Coordinate) 좌표계로 변환하는 것이 쉽습니다. 사각뿔을 정육면체 형태로 좌표변환하면 ndc 좌표계로 변환되며, z축을 기준으로 꾹 누르면 우리가 보는 이미지가 나오게 됩니다. 이 때 z축 기준 앞에 있는 점이나 폴리곤을 이미지에 먼저 그리면 됩니다.  
게임 등에서는 일반적으로 ndc 좌표계로 변환 전 사각뿔의 가장 가까운 평면(near plane), 가장 먼 평면(far plane)과의 거리를 정해놓습니다. 그래서 게임에서 일정 거리 이상 떨어지면 갑자기 물체가 없어질 때도 있습니다.

![2022-09-14_16-40-48](https://user-images.githubusercontent.com/48865276/190092097-446e60ac-6a55-4aa1-bc10-f1c4fd1e85c7.jpg) _그림 7) NDC coordinates_ 



형상을 렌더링하는 방법은 이게 끝이지만, 이렇게만 표현하게 되면 모델의 색상을 그대로 이미지에 투영하기 때문에, 그림자나 재질 표현이 불가능합니다. 따라서 shading이 필요해지게 됩니다. shading은 반사성 재질과 그림자를 고려해서 색상을 입히는 방법입니다.



## 빛의 반사

### 빛이란 무엇일까?

사진의 한 픽셀로 들어오는 빛들은 아래와 같이 나눌 수 있습니다.  
- scattering(눈에 보이는 최종 빛) = 반사광(reflection) + 투과광(transmission) + 자체발광(illuminance)

투과광과 발광 정도는 일반적이지 않으므로, 보통 표면에서 반사되는 반사광(reflection)만 생각합니다. 반사광은 다시 정반사와 난반사로 알 수 있습니다.  
- reflection(반사광) = diffuse(난반사) + specular(정반사)

- 눈으로 반사되는 빛의 비율: BSDF = BRDF + BTDF
- 반사의 정의
  - scattering(눈에 보이는 최종 빛) = 반사광(reflection) + 투과광(transmission) + 자체발광(illuminance)

### 우리는 반사에 대해 얼마나 알까?
  - 반사의 정의
    - scattering(눈에 보이는 최종 빛) = 반사광(reflection) + 투과광(transmission) + 자체발광(illuminance)
    - diffuse reflection(난반사)와 specular reflection(정반사)
  - diffuse(난반사)는 표면의 거칠기 때문이 아니다!
    - 미세표면(microsurface)이 거칠면 산란이 일어난다
    - 그럼 아주 매끈한 표면은 정반사할까? —> 돌을 아무리 매끄럽게 갈아도 거울이 되지는 않는다
    - diffuse는 표면 내부로 들어갔다 나온 빛 —> 물체 자체의 색상을 가지고 반사되어 나온다 (물체 자체의 색상: diffuse color = albedo = baseColor)
    - 표면에서 아무리 산란되어도 이는 diffuse reflection이 아니다



![2022-09-14_17-17-23](https://user-images.githubusercontent.com/48865276/190099999-ba6f98df-56f0-44da-aec3-bc8020d1e886.jpg)

![2022-09-14_17-14-51](https://user-images.githubusercontent.com/48865276/190099424-ce7469e8-ab41-4090-b5ad-a6683cd7fef6.jpg)    _그림 8) 빛의 정반사와 난반사_ 




### 빛의 “물리적" 반사 (Physically Based Rendering)
 - 반사의 두 가지 요소: diffuse, specular
   - diffuse(분산광, diffusion): 빛을 한 번 흡수했다가 방출
   - specular(반사광, specular): 빛이 표면에서 튕겨져 나감
 - specular 반사를 표현하는 법: metallic(금속성)과 roughness(거칠기)
   - 반사(specular)도 두 가지로 나눌 수 있다: 정반사(smooth)와 난반사(산란, rough)
   - specular 이미지: (더미데이터, metallic, roughness)로 이루어진 3채널 이미지

- 금속성이 아닌 물체의 specular 반사율(ks)는 보통 4% 정도 (기본 반사율)
- 반사율이 같아도, roughness(roughness ↔ gloss)에 따라 완전히 다르게 보인다!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/682abef2-97bd-41df-95a4-598c9056e259/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5307240f-19ee-4ee4-9c61-1560c051f8e0/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c741d72e-b33d-4acd-9e3f-bd9159f63cdf/Untitled.png)



### 난반사(diffuse) 뜯어보기

사실 정반사는 뜯어볼 게 없습니다.. ㅎㅎ




### 정반사(specular) 뜯어보기
- 프레넬
- image based lighting(IBL)





## PBR workflow

- pbr workflow
  - https://substance3d.adobe.com/tutorials/courses/the-pbr-guide-part-2
- obj 파일 구조

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2f23ed9c-d38f-4a29-a5c1-8a6b0a5835d2/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3b91e65f-afd4-4612-9580-2eee56446ccc/Untitled.png)




## 우리가 쓰는 3D 복원 방식 (for VRIN business)

- differentiable rendering







1. 빛의 반사
   - 빛이란 무엇인가
   - 빛의 각도에 따른 밝기 변화
   - 밝기 변화를 표현하려면 표면의 법선벡터(normal)가 필요하다
   - phong shading

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d01fa8a9-e64f-46a8-b6de-2bc73099a9d8/Untitled.png)





![2022-09-14_16-51-03](https://user-images.githubusercontent.com/48865276/190094298-efaf20b7-495c-4a68-8449-adc28bc1a81c.jpg)


![2022-09-14_16-51-03](https://user-images.githubusercontent.com/48865276/190094806-9a73a1e3-ebd9-40c5-a43c-60b6afb019a2.jpg)



![2022-09-14_16-52-29](https://user-images.githubusercontent.com/48865276/190094844-34bd649d-80da-4eef-9ab2-7dd63aaf3576.jpg)





![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/94008234-2626-4397-9b8d-77f0e5bbe990/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/233fb3ff-1bb3-44cc-8a9a-9e5a30e49438/Untitled.png)

- 금속은 specular reflection만 한다
- 입사각과 반사각이 같을 때 가장 밝기가 세지는 않다 (off-specular reflection) —> 프레넬 때문

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8f41ea7b-2f56-4143-a3c3-caff8e348f97/Untitled.png)



- Lambertian surface
- phong shading
- Cook-Torrance shading

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a606b067-fe95-4567-9297-564c65909597/Untitled.png)

- 프레넬

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/10218eb5-c0b4-4915-b452-c53c1cfb72fb/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4223d58d-67cf-4059-a36d-08f7a43588bb/Untitled.png)

- 빛의 전반사와 스넬의 법칙
  - 매질 경계에서는 빛이 굴절한다
  - 특정 각도에서는 완전히 정반사한다

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bd32184b-34ab-4040-a78f-0633e9506bda/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bc726e40-74f2-44e0-a6d3-2738e541a78c/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f54d2405-c5e8-4015-9844-68badb4c1618/Untitled.png)

- 프레넬(Fresnel) - 어떠한 물체도 가장자리에서 보면 완벽한 거울이다!

  - 모든 물체는 빛을 반사한다 (각도에 따라 반사율이 달라진다!)
  - 세 가지 특성
    1. 어떤 물체도 스치듯이 빛이 들어오면 100% 반사된다 (어떤 물체도 가장자리는 항상 거울처럼 역할)
    2. 재료에 상관없이 보는 각도에 따른 반사율 그래프 개형이 비슷하다
    3. 모든 물체가 반사하는데, 대부분 잘 안보이는 이유?(가죽 재질 등) - 거칠기(roughness)가 크면 reflection이 blur해지기 때문
  - 모든 것은 빛납니다
    - https://lifeisforu.tistory.com/380?category=567143
  - 모든 것은 프레넬을 가집니다
    - https://lifeisforu.tistory.com/381?category=567143

- ## Distribution Function

- Image-based lightning (IBL)























# Reference

1. 



