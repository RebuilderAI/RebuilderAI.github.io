---
date: 2022-07-27
layout: post  
title: 렌더 스튜디오를 위한 배경에셋 추천 기술 개발 
subtitle: Project Review
description:   
image:  
category: Zero-shot, image to text generation  
tags:
  - Zero-shot image to text generation
  - CLIP 
author: Gwan Hyeong Koo, Sehyun Kim
---

# 렌더 스튜디오를 위한 배경에셋 추천 기술 개발

최종 편집: 2022년 7월 27일 오전 10:47

안녕하세요~ 😃 저희는 리빌더 AI에서 방학동안 인턴을 수행하고 있는 구관형, 김세현 입니다. 저희는 방학동안 “렌더 스튜디오를 위한 배경에셋 추천 기술” 프로젝트를 진행했습니다. 어떤 내용의 프로젝트이고 결과는 어떻게 되었는지에 대해 이야기를 해보려고 합니다.

### 서론

 VRIN 앱을 이용해서 사용자들이 물체를 3D 모델로 변환하면, 판매까지 이어질 수 있을까요? 아마도 힘들 것 같습니다,, 물체의 배경이 없기 때문에 판매하긴 어렵겠죠. 우리가 일반적으로 보는 온라인 쇼핑몰에 판매 사진을 올리려면, 보통 스튜디오를 가서 사진을 찍는다고 합니다. 텀블러, 물컵과 같은 물건들도 스튜디오에 가서 촬영을 한 후 판매 사진에 등록한다고 하네요.

<p align="center">
  <img src="https://user-images.githubusercontent.com/48863707/181142759-cf95c93c-7bb2-4a9a-8177-6cc6b0724eb0.png" width="50%" height="50%">
  <p align="center"> 사진 1) 네이버쇼핑에 등록된 판매 사진들 </p>
</p>

만약 물체 사진만 찍어도 물체와 어울리는 배경 스튜디오 사진을 자동으로 생성해주면 어떨까요? 리빌더 AI의 AI팀에서는 사용자가 VRIN 앱을 이용해 물체를 3D 모델로 변환하면, 곧바로 어울리는 배경 스튜디오 사진을 추천해주는 시스템을 고안했습니다. 그리고, 이번 방학동안 인턴을 수행하게 된 저희가 해당 프로젝트를 맡아서 진행하게 되었습니다! 


### 내 제품과 가장 잘 어울리는 배경 스튜디오는?

저희의 모델을 요약하면 다음과 같습니다!

1. 판매하고자 하는 제품을 촬영하여 이미지 파일을 업로드한다.
2. AI 기반 Visual-Language Model과 Language Model을 통해 이미지와 가장 잘 어울리는 배경을 설명하는 text를 생성한다.
3. 해당 text와 가장 적합한 배경 에셋을 검색한 후 제품과 함께 배경 에셋을 3D 모델로 렌더링하여 제품 판매용 렌더링 이미지를 생성한다.

<p align="center">
    <img src="https://user-images.githubusercontent.com/48863707/181145573-f9e2822d-cf15-4d28-9da3-7ab283abdea0.png" width="60%" height="60%">
    <p align="center">VRIN 앱을 사용했을 때 배경 추천 프로세스</p>
</p>

<p align="center">
    <img src="https://user-images.githubusercontent.com/48863707/181145532-01b59470-5048-438a-bcba-ad34969dd3d0.png" width="60%" height="60%">
    <p align="center">구체적인 추천 과정</p>
</p>


### AI 모델 리서치
영상정보와 텍스트를 연결시켜 이해하는 feature embedding 모델들을 찾아보았고, 최종적으로 Socratic Model 구조를 선정하였습니다.  

![Untitled (5)](https://user-images.githubusercontent.com/48863707/181146131-3f686988-5ef1-4755-99b9-3cebe4d3fab7.png)

**<핵심 아이디어>**

- VLM(Visual-Linguistic Model: “눈으로 보고 생각하기”), LM(Linguistic Model: 입으로 말하는 과정)을 수행
- vision-text 결과를 language model로 재검토

**<선정 이유>**

- 거대한 규모의 pretrained model (BERT, GPT3, CLIP)을 활용, 최고 성능의 결과 기대
- 상품과 배경의 분위기가 학습이 가능한지 확인이 쉬움
- Multiple modalities(language, audio)를 통해 특정 specific modality (vision)의 denoise를 예방(마치 사람이 특정 물체를 보고 말로 표현하는 과정과 유사)


### CLIP을 이용해 text를 생성하고, 이를 다시 GPT-3로 가다듬으면..?

Socratic model 이라는 논문에서는 CLIP을 이용해 여러 text를 생성하고, 이를 다시 GPT-3로 가다듬어 아주 매끄러운 문장을 생성해내는 것을 보여주었습니다. 저희도 Socratic model 구조와 비슷하게 물체에 대한 정보 text를 4개 이상 뽑아내었고, 해당 문장들을 다시 GPT-3로 가다듬어 매끄러운 문장으로 만들고자 했습니다. 그 결과..

<p align="center">
    <img src="https://user-images.githubusercontent.com/48863707/181146221-c88da3ca-6e06-4bfa-a1a1-e5d81190e090.png" width="20%" height="20%">
    <p align="center">향수 이미지</p>
</p>


위와 같은 사진을 CLIP 모델에 넣고, 얻은 text 결과들을 다시 GPT-3로 변환한 결과는 다음과 같았습니다.

Image Caption 결과(영문): “**광활하고 건조한 풍경 속에 있는 향기로운 선인장”**

이는 GPT-3가 CLIP으로 구해낸 형식이 있는 문장들을 다시 매끄럽게 잘 표현했음을 보여주었습니다.



### LM model 수정 → “물체를 판매할 때 어떤 스튜디오/ 어떤 조명과 / 어떤 배경과 / 어떤 분위기로 해야하는지?” 를 고려했다!

위에서 얻은 결과는 단순히 물체를 표현하는 문장에 지나지 않습니다. 저희가 원하는 결과는 이 물건이 판매될 때 어떤 스튜디오 배경이 가장 어울리는가? 입니다,, 따라서 LM이 이러한 상황을 이해하도록 수정했고, 결과는 다음과 같이 수정되었습니다. 

<p align="center">
    <img src="https://user-images.githubusercontent.com/48863707/181146336-bcad19f6-2e77-48b1-96b7-a424261024c2.jpg" width="30%" height="30%">
    <p align="center">직접 촬영한 텀플러</p>
</p>

결과 : “A **cool, modern studio** with clean lines and **bright lighting** would be a great fit for selling this item. Props that would complement the item would be a simple **white background,** a few other similar items to show scale, and maybe some greenery to add a pop of color.”

(한국어 번역) “깔끔한 라인과 밝은 조명이 돋보이는 시원하고 모던한 스튜디오가 이 아이템을 판매하기에 아주 적합합니다. 아이템을 보완할 수 있는 소품으로는 **심플한 흰색 배경**, **제품의 크기**를 잘 부각할 몇 가지 **유사한 아이템**, 그리고 색상 톤에 변화를 줄 만한 **푸른 잎사귀나 자연물**이 있습니다.”


### DALLE-mini를 이용한 배경 생성

현재 저희가 진행하는 프로젝트는 존재하는 배경 에셋들 중 입력 이미지와 가장 어울리는 배경 에셋을 추천해주는 시스템입니다. 하지만 배경 에셋을 완전히 새로 생성해낸다면 얼마나 좋을까요? DALLE-mini를 이용해서 추천 배경도 생성해보았습니다. 물론 당장은 서비스에 적용이 힘들지만, 상용화된다면 많은 온라인 판매자들이 편리하게 사용할 것으로 생각됩니다. 더 이상 스튜디오에 가서 판매 사진을 찍지 않아도 되는 것이죠 ㅎㅎ

<p align="center">
    <img src="https://user-images.githubusercontent.com/48863707/181146396-a9c93e5d-c4a3-4015-b604-981fe3509a8e.png" width="60%" height="60%">
    <p align="center">추천 배경 생성</p>
</p>


### 데모 사이트 제작

저희가 만든 프로젝트의 데모 사이트를 제작했습니다. 사진을 입력하면 그와 어울리는 배경을 추천해주는 프로세스입니다. 물론 배경은 리빌더AI가 보유하고 있는 3D 배경 asset 입니다!

링크 : [http://recommendation.vrin.co.kr:9080/](http://recommendation.vrin.co.kr:9080/)

### 한계 및 발전 계획

1. 현재 이미지를 텍스트로 변환하고 배경 에셋과 매칭하는 시간은 평균 10~13초 정도 소요됩니다. 최적화를 더 진행하거나 모델을 수정한다면 더 빠를 것이라고 생각됩니다. 
2. 세밀한 추천(주요 색상, 무드, 촬영 기법, 조명, 적절한 소품 배치, 텍스처, 사진의 전반적인 분위기 묘사) 기능을 향상시키면 더 완벽할 것입니다.
3. 단순히 스튜디오 3D 렌더링(배경 에셋) 이미지뿐만 아니라 **공간 에셋**에 대해서도 추천이 가능한지 검토중입니다. 넓은 독서실이나 미팅룸과 같은 open된 공간 에셋을 의미합니다,,


