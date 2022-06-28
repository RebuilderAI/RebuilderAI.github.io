# RebuilderAI Blog Guides

# 처음 들어오셨다면?
  1. docs 폴더 안에서 _author 폴더에 이름 등록
  - 파일을 (영문 이름).md (성 아님) 으로 만들고 아래 내용을 md 파일 최상단에 붙여넣습니다.
  ~~~
  ---
  layout: author
  photo: https://avatars.githubusercontent.com/u/48865276?s=400&u=238d4a53bbf988ff089b4cffbe71b14c31b1d414&v=4
  name: q10
  display_name: Kyuyeol Park
  position: Tech Leader
  bio: 안녕하세요~ 박규열입니다.
  github_username: parkinkon1
  ---  
  ~~~


2. _posts 폴더에서 글 작성
- 파일 이름 규칙:
  > 2020-08-29-reconstruction-test.md. (년-월-일-제목-기타-기타2.md)

3. 작성한 markdown 파일 최상단에 아래 내용 붙여넣기

~~~
---
date: 2022-06-29 10:09:10  
layout: post  
title: 리빌더에이아이 변환기술은 어떻게 되어있을까?  
subtitle: 리빌더에이아이 변환 기술 설명  
description: 글 설명  
image: https://user-images.githubusercontent.com/48865276/175257706-7b24117c-b1e3-4c69-9346-b3d90330935b.png  
category: lidar  
tags:
  - 3d reconstruction
  - nerf
  - graphics
author: Kyuyeol Park
---
~~~
