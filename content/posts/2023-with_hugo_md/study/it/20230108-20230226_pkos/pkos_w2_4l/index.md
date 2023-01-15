---
date: 2023-01-15T23:50:00+09:00
lastmod: 2023-01-15T23:50:00+09:00
author: "MinHo Jung"
authorLink: ""
license: "CC BY-NC-ND 4.0"
draft: false
toc: true
hiddenFromHomePage: false
hiddenFromSearch: false
linkToMarkdown: false
rssFullText: false

title: "PKOS 2주차 - 회고(4L)"
subtitle: ""
description: ""

images: []
featuredImage: ""
featuredImagePreview: ""

categories: ["IT/Kubernetes"]
tags: ["Study-Group-Online", "CloudNet@", "Gasida/PKOS"]

alias: []
series: []
---

[PKOS 1주차 - 회고(4L)](../pkos_w1_4l)

[PKOS - Production Kubernetes Online Study 포스팅을 시작하며](../pkos_intro)

[PKOS 1주차 - AWS kOps 설치 및 기본 사용](../pkos_w1_hands-on)

[PKOS 2주차 - 회고(4L)](../pkos_w2_4l)

---

## Liked: 좋았던 점
- 네트워크 동작과정

기존에 잘 알지 못했던 네트워크 동작 과정에 대해 알 수 있어서 좋았습니다.
어려울 수 있는 내용을 그림과 실제 동작하는 모습을 보여주어 보다 쉽게 이해할 수 있었습니다.


## Lacked: 아쉬웠던 점
- 실습환경 미흡

1주차 때도 겪었던 부분이라 강의 시작 약 1시간전에 미리 셋팅을 했는데 결과적으로 준비가 미흡했습니다.

첫번째로는 ssh 접속 도구로 SupperPutty 를 사용하는데 접속하려고하면 알 수 없는 오류로 실행이 되지 않았습니다. 며칠전만해도 잘 돌아갔는데... 실습 당일이러니 불필요한 곳에 시간이 많이 낭비 됐다는 생각이 듭니다.

두번째로는 모임장님께서 안내해주신 원클릭 배포를 수행하기 이전 기존 실습 구성을 잔재(?)가 남아있어 원할하게 동작하지 않았습니다. ssh 접속까지는 했지만 `kubectl get pod` 명령 등이 실행되지 않았습니다.


## Learned: 배운 점
- 실습을 포기하고 강의에 집중

지난주와 마찬가지로 이번주에도 실습환경 구성이 늦어져서 실습 진행을 하지 못했습니다.
지난주엔 실습을 어떻게든 따라가기 위해 강의를 듣는중 마는둥 했었는데, 이번주엔 실습환경 구성하는것을 과감히 포기하고 모임장님의 강의에 집중했습니다.


## Longed for: 앞으로 바라는 점
- 기초지식 보충 필요

모임장님께서 쉽게 설명을 해주시는 덕분에 이해는 곧 잘 되는 편입니다. 하지만 생각의 사고가 설명해주시는데 그치는 한계가 느껴집니다. 네트워크나 쿠버네티스 등의 지식이 조금 더 쌓인다면 보다 폭 넓게 생각을 할 수 있텐데라는 아쉬움이 남습니다.

다가오는 3주차 사이에 명절이 있기에 그 시간을 활용해서 쿠버네티스를 공부해보고 시간적 여유가 있다면 네트워크 공부도 겸해보겠습니다.

