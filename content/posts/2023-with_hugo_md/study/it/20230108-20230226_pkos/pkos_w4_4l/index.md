---
date: 2023-02-05T21:58:38+09:00
lastmod: 2023-02-05T21:58:38+09:00
author: "MinHo Jung"
authorLink: ""
license: "CC BY-NC-ND 4.0"
draft: false
toc: true
hiddenFromHomePage: false
hiddenFromSearch: false
linkToMarkdown: false
rssFullText: false

title: "PKOS 4주차 - 회고(4L)"
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

[PKOS 2주차 - 쿠버네티스 네트워크](../pkos_w2_hands-on)

[PKOS 3주차 - 회고(4L)](../pkos_w3_4l)

[PKOS 3주차 - Ingress & Storage](../pkos_w3_hands-on)

[PKOS 4주차 - 회고(4L)](../pkos_w4_4l)

[PKOS 4주차 - Harbor & Gitlab & ArgoCD](../pkos_w4_hands-on)

[PKOS 5주차 - 회고(4L)](../pkos_w5_4l)

[PKOS 5주차 - Prometheus & Grafana](../pkos_w5_hands-on)

[PKOS 6주차 - 회고(4L)](../pkos_w6_4l)

[PKOS 6주차 - Alert Manager & Logging System](../pkos_w6_hands-on)

---

## Liked: 좋았던 점
- 이전 주차들에 비해 비교적 쉬웠던 내용

이번 4주차는 쿠버네티스 생태계 도구에 대한 간단한 실습이라 다른 주차들에 비해 비교적 난이도가 낮게 느껴졌습니다.

쿠버네티스 학습 난이도와 더불어 업무 시간을 쪼개어 학습하는데 부담을 느끼고 있었는데, 가뭄에 단비처럼 휴식할 수 있는 한 주가 되지 않을까 생각됩니다.




## Lacked: 아쉬웠던 점
- 로컬환경이 아닌 클라우드 환경에서 도구 연동

학습 교재기반으로 스터디가 진행되다보니 소개된 도구들이 로컬 환경에 맞추어져 있습니다.

로컬 환경에서 구축도 필요하지만 클라우드 환경에서 어떻게 연동할 수 있는지 그리고 대체할 수 있는 서비스가 어떤게 있는지 궁금했습니다.




## Learned: 배운 점
- 쿠버네티스 생태계 도구

쿠버네티스 생태계의 도구 3가지를 알게 됐습니다. 

첫번째는 Harbor(로컬 컨테이너 이미지 저장소)이고, 두번째는 GitLab(로컬 Git 소스 저장소)이며, 세번째는 ArgoCD(GitOps 시스템) 입니다.



## Longed for: 앞으로 바라는 점

- 클라우드 환경에서 도구 활용법

오늘 소개된 도구들을 AWS 클라우드 환경에서 어떻게 적용해 볼 수 있는지 자료 조사가 필요합니다.

Harbor가 컨테이너 이미지 저장소 역할을 한다고 하는데 AWS ECR과 대체될 수 있는지 등에 대해 조사하고, 조사한 자료를 바탕으로 팀원들과 의견을 주고 받으면 좋을 것 같습니다.

