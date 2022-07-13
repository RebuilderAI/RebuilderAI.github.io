---
date: 2022-07-12 06:00:00  
layout: post  
title: 리빌더AI 물체 인식 기술
subtitle: Object recognition
description: Object recognition technique in RebuilderAI 
image:  https://user-images.githubusercontent.com/36464983/178456743-5e9f52b0-cabe-42cd-8c48-18845a456944.png
category: Object Recognition
tags:
  - Salient object detection
  - Transparent object detection
  - Pre-processing for 3d reconstruction
author: ansh941
---

# 목차

1. 물체 인식 문제
2. SOD(Salient Object Detection)
3. 투명 영역 인식 문제
4. TOD(Transparent Object Detection)

# 1. 물체 인식 문제

저희는 휴대폰으로 영상을 찍어 주변에서 흔하게 볼 수 있는 물체를 종류에 상관없이 깔끔하게 복원하고 싶습니다.

이 경우 물체와 물체가 아닌 것들이 모두 영상에 담기게 되는데요, 3D로 만들고 싶은 물체만 복원하려면 어떻게 해야할까요?

간단하게 생각해서 해당 물체를 제외하고 나머지 부분을 모두 제거하면 됩니다.

저희는 이러한 문제를 SOD(Salient Object Detection) 기술을 이용해 풀어내고자 하고 있습니다.

# 2. SOD(Salient Object Detection)?

Salient는 가장 중요한 이라는 뜻을 가지고 있는데요, 그러면 Salient object detection은 가장 중요한 물체를 찾는 문제라고 보실 수 있습니다.

그러면 저희의 상황에 따라 예시를 들면 영상에서 가장 중요한 물체를 검출하는 것입니다. 즉, 해당 영상에서의 주요한 물체를 찾고 그 외 나머지를 배경으로 처리해 제거해버리는 작업인 것이죠.

아래 그림과 같이 말이죠.

<img src="https://user-images.githubusercontent.com/36464983/178453186-eae3fac4-10d9-43f5-86d9-e6066c381ab6.jpeg" width="500" height="500"/>
그림 1: 원본 이미지<br><br>

<img src="https://user-images.githubusercontent.com/36464983/178453329-fd570a97-3753-4edc-bd57-4ac24542a179.jpg" width="500" height="500"/>
그림 2: 검출된 이미지<br><br>

저희가 있는 공간에서 장식으로 존재하는 식물을 salient object로 detection한 모습입니다.

이러한 task는 일반적인 object detection이나 segmentation과는 어느 정도 다른 부분이 존재합니다.

보통의 object detection이나 segmentation은 클래스가 정해져 있고, 그 클래스에 속하지 않는 다른 물체는 검출할 수 없습니다. 그렇게 되면 주변에 보이는 아무 물체를 scan해서 3D로 만든다는 것은 어렵게 되는 것이죠.

따라서 저희는 detection이나 segmentation이 아닌, SOD 모델을 선정해 진행하게 된 것입니다.

# 3. 투명 영역 인식 문제

하지만 이것이 전부는 아닙니다.

3D 복원에서 투명한 영역의 복원은 전통적인 문제 중에 하나입니다.

많은 3D 복원 모델이 depth를 이용하고 있는데요, 투명한 영역의 경우에는 depth가 제대로 계산되지 않기 때문에 제대로 복원이 되지 않는 문제가 있었습니다. 투명한 영역을 복원하기 위한 연구들이 따로 진행되고 있을 정도로 활발히 연구되고 있는 문제입니다. 이러한 문제를 해결하기 위해 많은 연구들이 진행되어 왔고, 그 중에는 반사나 굴절 등의 성질을 이용하려 하는 것이 시도들이 많았습니다.

depth가 제대로 계산되지 않는 이유는 뒤가 투명하게 비치기 때문입니다. 일반적으로 depth는 현재 위치와 다른 위치에서 본 이미지 값 차이를 이용해 계산하게 되는데 뒤가 비치면 그것이 제대로 계산될 수 없기 때문이죠.

그로 인한 결과를 아래 그림에서 보실 수 있겠습니다.
![이미지와 depth
출처 : [https://ai.googleblog.com/2020/02/learning-to-see-transparent-objects.html](https://ai.googleblog.com/2020/02/learning-to-see-transparent-objects.html)](https://1.bp.blogspot.com/-VNB-xPguois/XkQ7sqF3VWI/AAAAAAAAFRc/ek6wUpjOGNw0u9vALnnW9zXCZ8VsWTjcACLcBGAsYHQ/s1600/image3.png)

그림 3: 이미지와 depth<br>

![이미지와 depth를 통해 복원한 결과
출처 : [https://ai.googleblog.com/2020/02/learning-to-see-transparent-objects.html](https://ai.googleblog.com/2020/02/learning-to-see-transparent-objects.html)](https://1.bp.blogspot.com/-mcdDuutDw5Q/XkQ7ssTGUvI/AAAAAAAAFRg/OIyqAa7Qbgs1yGWB7vFKL2E90hCPXiuYACEwYBhgL/s640/image2.gif)

그림 4: 이미지와 depth를 통해 복원한 결과<br>

결과를 보시면 다른 불투명한 물체는 제대로 계산된 것에 비해, 투명한 물체들은 제대로 계산된 부분이 그렇지 않은 부분보다 현저히 적습니다. 그렇기 때문에 reconstruction 결과에서도 대부분 사라져있는 모습을 보실 수 있습니다.

따라서 저희는 투명한 영역을 예외로 처리하고 기존과는 다른 알고리즘을 적용할 필요가 있다 생각했고, 투명한 영역을 검출할 필요성을 느꼈습니다. 이런 문제는 TOD(Transparent Object Detection)로 분류됩니다.

# 4. TOD(Transparent Object Detection)
<img src="https://user-images.githubusercontent.com/36464983/178453073-9e2112ec-41fd-4352-b674-4a4ef82bac07.jpeg" width="500" height="500"/>
그림 5: 원본 이미지<br><br>

<img src="https://user-images.githubusercontent.com/36464983/178453390-78e6c921-e0bb-4f93-84c4-04d17bd39517.jpg" width="500" height="500"/>
그림 6: 검출된 이미지<br><br>

위 그림은 저희가 상주하는 공간에서 사용 중인 보틀을 찍은 것으로, 원본 이미지와 SOD 및 TOD 작업을 통해 검출한 결과입니다. 이러한 비교 결과에서 투명한 영역이 검출되었다는 것을 구분할 수 있도록 mask 값을 낮게해 어둡게 보이게 하였습니다.

위 보틀처럼 투명한 부분이 존재할 경우, 이러한 부분을 예외로 처리해 문제없이 복원하고자 생각한 것이 투명 영역을 검출해 따로 불투명 영역과 mask를 구분하여 3D reconstruction 파트에 전달해주는 것이었습니다.

이러한 2가지 과정을 3D reconstruction을 위한 전처리 파이프라인에 포함하여, 물체를 최대한 깔끔하게 reconstruction 할 수 있도록 합니다.

# Reference 
그림 3,4 출처 : https://ai.googleblog.com/2020/02/learning-to-see-transparent-objects.html
