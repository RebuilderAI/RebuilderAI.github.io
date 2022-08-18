---
date: 2022-08-16 10:09:10  
layout: post  
title: 일반 인공지능을 향하여
subtitle: CLIP 모델부터 Flamingo, 그리고 GATO까지
description: Brief introduction to multi-modal learning AI models
image: https://user-images.githubusercontent.com/36873797/185295737-7d05b246-688c-4749-aeee-1632ce06496e.jpg
category: zero-shot learning, multi-modal learning
tags:
  - multi-modal learning
  - CLIP
  - zero-shot learning
author: Gwan Hyeong Koo, Sehyun Kim
---

# 일반 인공지능을 향하여

### 부제 : CLIP 모델부터 flamingo, 그리고 GATO까지

<aside>
  
👦🏻 발표자 : 구관형, 김세현

</aside>

## 📑 Multimodal learning이란?
---

사람은 5가지의 감각기관을 통해서 다양한 형태의 데이터를 얻습니다. 이때 다양한 형태(modality)의 데이터를 “modal”이라고 합니다.

<p><img src="https://user-images.githubusercontent.com/36873797/185296232-2d06d548-aa7b-453f-816a-d7a18157af34.png" align="center" style="margin-left:5px; margin-top: 5px;">
<p><img src="[https://user-images.githubusercontent.com/36873797/185296232-2d06d548-aa7b-453f-816a-d7a18157af34.png](https://user-images.githubusercontent.com/36873797/185296907-03976064-9ae7-4b77-943d-cefc799b8c21.png))" align="center" style="margin-left:5px; margin-top: 5px;">

사람은 다양한 형태의 데이터를 통해서 상황을 인지하고 파악합니다. 눈으로 시각적인 정보를 받고 소리로 이를 판단하죠. 그럼 현재 AI는 어떻게 연구되고 있을까요?

이미지를 통해 시각적인 정보를 학습하는 **“비전"** 분야는 이미지, 혹은 동영상 데이터를 통해 학습합니다. text 기반의 언어를 통해 언어를 학습하는 **“자연어"**분야는 언어를 학습합니다. 이 두가지 분야는 현재 AI의 흐름을 선도하는 연구분야입니다. 하지만 비전과 자연어 분야는 각 분야에 대해서만 고도화가 되었기에, 독립적인 분야라고 할 수 있습니다. 즉, AI 모델은 눈으로 보기만 하거나 언어를 사용하기만 하는 것이죠. 그럼 이 두 가지 분야를 합치면 어떤 일들이 가능할까요?

<aside>
  
💡 AI 모델이 “비전"과 “자연어" 혹은 “음성" 분야를 함께 인식하면 어떤 일이 일어날까?

</aside>

바로 이렇게 여러 가지 modal 을 이용해 AI 모델을 학습시키는 분야를 Multimodal learning이라고 합니다. 기존 “비전" 분야의 퍼포먼스와 “자연어" 분야의 퍼포먼스를 합치면 더욱 일반적인 인공지능에 다가설 수 있습니다.

## 📑 CLIP 이란, “사물 인식의 새로운 모델”

---

저희는 CLIP 모델을 간략하게 설명하고자 합니다. CLIP은 분류(classification) 모델이지만, 단순히 이미지 데이터만 사용하지 않습니다. 여기서 분류 모델이라는 것은 어떤 사진에 고양이가 있다면 고양이가 있다고 분류해주는 모델입니다.
![classification_demo](https://user-images.githubusercontent.com/36873797/185297171-a13133d1-8d3c-4ab4-892c-415c01ddd171.png)

사실 위 그림에 있는 cat 이라는 표현은 숫자로 바뀌어 학습됩니다. 예를 들어, cat의 class가 50번 이라면 cat 대신에 50이라는 숫자가 one-hot vector로 바뀌고, 그것이 AI가 학습하는데 사용됩니다. 즉, “CAT”이라는 단어의 의미가 모델에서 전혀 쓰이지 않고, 단순히 분류만 해주는 것입니다. 
![classification_demo2](https://user-images.githubusercontent.com/36873797/185297168-8611eeb3-6a16-443a-bbfb-4afb4a53cd40.png)

<aside>
💡 기존 분류 모델은 “CAT”이 가지는 의미론적 정보 (semantic information)을 학습하는 것이 아니라, 단순히 확률적으로 분류해주는 것에 불과하다!

</aside>

하지만 “자연어"를 여기에 사용해보면 어떨까요? 다시 말해서, 고양이 사진에 정답 label을 단순히 class id로 주는 것이 아니라, 어떤 문장으로 주는 것입니다. 

![classification_demo3](https://user-images.githubusercontent.com/36873797/185297166-99759733-3cbc-404a-a037-75af4a2d6179.png)

정답 label을 의미론적인 문장으로 주고, 이미지와 함께 학습시키는 아이디어가 바로 CLIP 논문의 전부입니다. CLIP은 이렇게 4억개의 이미지와 텍스트(caption) 쌍으로 대규모 학습했습니다. 즉, text를 통해 자연어의 supervision을 사용하여 학습한 셈이죠. 그 결과 이미지를 단순히 분류하는 것이 아니라, 의미론적인 해석이 가능해졌습니다. 

![classification_demo4](https://user-images.githubusercontent.com/36873797/185297163-e575fe52-4578-4e51-9f44-6dc1691e476a.png)

따라서 CLIP 모델은 위와 같이 학습 시에 배우지 않았던 것에 대해서도 예측이 가능합니다. 이를 zero-shot 이라고 하는데요, 분명이 이 강아지의 색이나 기분, 그리고 어떤 종인지에 대해 학습시키지 않았음에도 정답 label을 자연어로 학습하다 보니 잘 유추해냅니다.

<aside>
💡 CLIP은 “비전”과 “Text” 두 가지 modal을 활용하여 강력한 zero-shot을 구축해냈다

</aside>

## 📑 Flamingo : 사용자와 상호작용을 극대화한 대화형 인공지능 기술

---

위에서 소개해드린 CLIP을 적극적으로 활용한 모델이 있습니다. Flamingo는 올해 5월, 구글 딥마인드에서 발표한 모델입니다. (왜 논문 제목이 플라밍고일까요..)

우리는 CLIP을 통해서 이제 “비전" task에서 text와 같은 의미론적인 해석이 가능하다는 사실을 알게 되었습니다. 기존 분류 모델과는 다르게, 이젠 어떤 이미지를 보고 text를 의미론적으로 해석하는게 가능한 것이죠. 그럼 이런건 어떨까요. 사용자와 사진으로 대화해보는 것입니다!!

![flamingo1](https://user-images.githubusercontent.com/36873797/185297162-d855b2df-32b6-4379-b864-31fd7d250334.png)

현재 많은 챗봇들은 단순히 텍스트를 통해서 소통합니다. 하지만, 우리는 CLIP을 이용해 시각적인 이미지 데이터를 의미론적으로 AI가 해석할 수 있다는걸 알았습니다. Flamingo는 여기서 착안해, 이미지와 text를 넘나드는 대화형 AI 모델을 선보였습니다. 이런 것도 가능하네요.

![flamingo2](https://user-images.githubusercontent.com/36873797/185297156-106ef3bd-486d-4b5e-b029-78a5c0b6e49d.png)

사용자가 입력하는 text의 흐름에 맞게 다음 text의 문맥을 유추합니다. 가령, 사람이 숫자의 덧셈을 입력하면 flamingo도 다음에는 덧셈을 대답해야지~ 라는 흐름을 알게 됩니다.

<aside>
💡 Flamingo는 단순히 대화형 image-text prompt 기술이 아니라, 일반적인 task를 AI 모델이 인지할 수 있음을 보여준 사례이다.

</aside>

정말 놀라운 점은, “비전” 분야에서는 단순히 한 task 에서만 AI 모델이 가능했습니다. 하지만, 이미지와 text 두 가지 modal을 사용함으로써 좀 더 범용적인 task에 적용이 가능하다는 점입니다. Flamingo는 검색 기술에 매우 활용성이 높을 것 같은데요, 예를 들어 내가 원하는 이미지의 사진을 넣고 text로 여기에 어떤 점이 비슷한 이미지를 찾아줘~ 라는 서비스에 사용 가능합니다.

![omnisearch](https://user-images.githubusercontent.com/36873797/185297153-98667e52-aef4-453b-acaf-a3b322136018.png)

실제로, 올해 5월부터 네이버 서치 팀에서는 “옴니서치" 서비스를 네이버 스마트렌즈에 적용했습니다. 텍스트와 이미지를 결합한 multimodality를 적극적으로 활용한 검색 기술입니다.

## 📑 GATO : 일반인공지능, 그 마지막 여정

---

구글 딥마인드는 올해 4월 29일 CVPR에 Flamingo를 제출함에 이어 5월 12일에는 GATO라는 모델을 발표합니다. (이건 그냥 아카이브에 바로 공개한거같네요..) GATO는 Flamingo의 연장선에 있다고 생각되는 모델인데요, 바로 범용적인 인공지능 모델입니다. 아타리 게임을 하고, 이미지 캡션 지정, 인간과 채팅, 로봇 팔로 컬러 블록을 쌓는 등 600여가지 기능을 갖춘 제너럴리스트(generalist) 에이전트를 “GATO”라고 이름을 지었습니다. 

![gato1](https://user-images.githubusercontent.com/36873797/185297147-da29be4b-a557-4a1e-90f9-506f8b9a78d3.png)

Flamingo도 한 가지 task가 아니라 여러 대화형 task가 가능했지만, GATO처럼 이렇게 다른 domain에서 task를 수행하진 못했습니다. GATO는 정말 여러가지 task를 수행할 수 있는 범용적인 인공지능 알고리즘인 셈입니다. 

이렇게만 보면 GATO는 정말 인간에 가까워진 인공지능인가..? 라는 생각이 듭니다. 하지만 논문을 보면 이는 전혀 아니라는 사실을 알 수 있습니다. 

![gato2](https://user-images.githubusercontent.com/36873797/185297141-a5f42f12-dfba-412c-aa40-81e80f4b961a.png)

GATO 논문은 “소문난 잔치에 먹을 게 없다"라는게 딱 어울리는데요, 사실 모든 작업들을 트랜스포머(transformer)를 통해 tokenization (토큰으로쪼개기)를 한 후, 순차적으로 일렬로 늘어놓고 예측 모델을 돌린 것입니다. GATO는 다양한 task들을 이렇게 tokenization시켜 12억개의 매개변수로 학습시켰다고 하네요.

그냥 무자비한 GPU 컴퓨팅 파워를 이용한 모델이었던 것입니다..

- tokenization 방식

| continuous | 1024개의 uniform unit (여기서는 값 자체보다는 토큰들이 가지는 순서를 학습하는 것으로 일치시키는 것으로 생각됨) |
| --- | --- |
| discrete | 0,1024를 이용 → 아타리 게임도 이렇게 버튼을 이산화 |
| text | setence Pice [0,320000] |
| image | non-overlapping 18x18 patches -> normalized [-1,1] |

그럼 GATO와 같이 여러 task를 학습시키면 정말 사람과 비슷한 범용적인 인공지능이 가능할까요? 절대 아닙니다. 그저 다양한 환경에서 학습된 에이전트들을 한번에 고용해서, 한 네트워크에 집어넣은 것과 동일합니다. 어찌 되었든, 여러 범용적인 task들에 모두 사용이 가능하지만 사람과 동일한 것은 아닙니다. 단순히 너무 많은 데이터로 학습되어 그저 input이 어떤게 왔을 때 output이 어떤게 올 수 있도록 확률 분포 모델링된 모델에 불과합니다. 

사람과 다른점은 무엇일까요? 사람은 누가 학습시키지 않아도, 스스로 사고하고 추론할 수 있습니다. 하지만 인공지능은 강제로 학습을 해야하고, 그 학습 조차도 수많은 데이터를 확률 통계적으로 학습한 것에 지나지 않습니다. 

<aside>
💡 인공지능 기술은 그동안 풀지 못했던 수많은 문제를 해결할 수 있는 엄청난 알고리즘이다. 하지만 인간과 비슷한 방식으로 작동되는 것은 아니다. 단순히 수 많은 학습 결과의 산물에 지나지 않는다.

</aside>

## 📑 너도나도 GPU ?? 혁신적인 모델이 나와야 새로운 발전이!

---

Openai의 GPT-3는 2020년 초에 발표되어 엄청난 퍼포먼스를 보였습니다. GPT-3는 현재 가장 규모가 큰 언어모델로 이전 모델인 GPT-2보다 최대 100배나 큰 1750억개의 매개변수가 사용되었습니다. 감이 잘 안오시겠지만, 일반 평범한 클라우드 플랫폼에서 이 정도로 훈련시키려면 대략 300년이 넘게 걸린다고 합니다. 학습에 들어간 단어는 4990억건에 가깝다고 합니다. CLIP도 마찬가지로 4억개의 image-text pair를 Tesla V100 GPU 592대에서 18일동안 학습시켰다고 합니다. (RN50x64 모델 기준)

이렇게 구글 딥마인드나 OpenAI와 같은 거대 기업들이 어마어마한 컴퓨팅 지원을 바탕으로 초거대 AI 모델을 만드는 추세가 이어지고 있습니다. 하지만 이러한 학습은 너무 많은 컴퓨팅 파워를 사용해 기후변화에 큰 영향을 주기도 합니다. 과연 인공지능의 미래는 어떻게 될까요? 인간에게 도움이 되기 위해 개발된 인공지능 모델은 편리함을 주었지만, 거꾸로 기후변화에 큰 요인으로 작용하고 있습니다. 

현재 Multimodal learning은 비전과 자연어 분야에서 잘 작동하는 모델을 그대로 가져와 사용한 것에 지나지 않습니다. 특별한 알고리즘이 없었죠.. 더욱 강력한 modality를 사용 가능하기 위해선 CLIP과 같은 거대 AI 모델도 좋지만 새로운 알고리즘이 등장해야할 때인 것 같습니다.
