---
date: 2023-01-29T19:05:16+09:00
lastmod: 2023-01-29T19:05:16+09:00
author: "MinHo Jung"
authorLink: ""
license: "CC BY-NC-ND 4.0"
draft: false
toc: true
hiddenFromHomePage: false
hiddenFromSearch: false
linkToMarkdown: false
rssFullText: false

title: "PKOS 3주차 - Ingress & Storage"
subtitle: ""
description: ""

images: []
featuredImage: ""
featuredImagePreview: ""

categories: ["IT/Kubernetes"]
tags: ["Study-Group-Online", "CloudNet@", "Gasida/PKOS"]

alias: ["쿠버네티스", "Kubernetes", "kubernetes", "K8s", "k8s"]
series: ["PKOS"]
---

[PKOS 1주차 - 회고(4L)](../pkos_w1_4l)

[PKOS - Production Kubernetes Online Study 포스팅을 시작하며](../pkos_intro)

[PKOS 1주차 - AWS kOps 설치 및 기본 사용](../pkos_w1_hands-on)

[PKOS 2주차 - 회고(4L)](../pkos_w2_4l)

[PKOS 2주차 - 쿠버네티스 네트워크](../pkos_w2_hands-on)

[PKOS 3주차 - 회고(4L)](../pkos_w3_4l)

[PKOS 3주차 - Ingress & Storage](../pkos_w3_hands-on)

---

작성중 입니다.

---

# PKOS 3주차



## 들어가기전에

본 내용은 `CloudNet@` 팀에서 진행하는 `쿠버네티스 실무 실습` 스터디를 기반으로 작성된 내용입니다.

또한 개념 설명에서 사용된 이미지의 출처는 스터디 학습 자료에서 가져온 것을 밝힙니다.

- 참조: https://www.notion.so/AWS-EKS-VPC-CNI-1-POD-f89e3e5967b24f8c9aa5bfaab1a82ceb



### 과제 수행결과

#### 과제1

- 목표 : Ingress(with 도메인)에 PATH /mario 는 mario 게임 접속하게 설정하고, /tetris 는 tetris 게임에 접속하게 설정하고, SSL 적용 후 관련 스샷 올려주세요
  - 특히 ALB 규칙 RUles 를 상세히 보고, 의미를 이해해 볼 것



#### 과제2

- 목표 : 호스트 Path(local-path-provisioner) 실습 및 문제점 확인과 성능 측정 후 관련 스샷 올려주세요



#### 과제3

- 목표 : AWS EBS를 PVC로 사용 후 온라인 볼륨 증가 후 관련 스샷 올려주세요



#### 과제4

- 목표 : AWS Volume SnapShots 실습 후 관련 스샷 올려주세요


