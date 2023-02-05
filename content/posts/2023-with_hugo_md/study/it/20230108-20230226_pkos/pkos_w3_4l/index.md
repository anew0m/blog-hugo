---
date: 2023-01-29T22:12:09+09:00
lastmod: 2023-01-29T22:12:09+09:00
author: "MinHo Jung"
authorLink: ""
license: "CC BY-NC-ND 4.0"
draft: false
toc: true
hiddenFromHomePage: false
hiddenFromSearch: false
linkToMarkdown: false
rssFullText: false

title: "PKOS 3주차 - 회고(4L)"
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

---

## Liked: 좋았던 점

-   데이터 관리 방법

당연히 알고 있어야할 데이터 관리에 대한 부분을 알게 되어 좋았습니다.

파드에서 관리되는 데이터는 당연히 파드가 소멸되면 데이터도 같이 소멸되는데 그 부분을 깜빡하고 있었습니다.

이 문제는 PVC(Persistent Volume Claim) 저장소 마운트를 통해 해결 할 수 있었습니다.

## Lacked: 아쉬웠던 점

-   조각조각 이해는 가지만 아직 큰 그림이 그려지지 않음

쿠버네티스와 관련된 기반 지식이 아직 많이 부족합니다. 다른분들이 올려주는 유머코드를 이해하지 못하고 멍하니 바라만 보고 있었습니다.

이번 명절에 쿠버네티스 강의를 따로 듣고자 했는데 다른 업무로 인해 강의 수강하지 못했습니다. 업무와 과제를 끝마치고 틈틈히 지식을 쌓아야할 것같습니다.

## Learned: 배운 점

-   ALB와 데이터 백업/복구 방법

지난시간 NLB에 대해 알게 되었는데 이번 시간엔 ALB를 알게 됐습니다.

같은 로드밸런서인데 Network 단에서 처리되는것과 Application 단에서 처리되는것의 차이로 보이는데 좀 더 학습이 필요합니다.

이번 시간 파드 내 PVC 저장소를 마운트해 데이터를 관리하는법을 배웠습니다. PVC 저장소를 snapshot 하여 백업과 복구하는 방법을 배웠습니다.

## Longed for: 앞으로 바라는 점

-   오늘 배운 점을 실무에 어떻게 적용할 수 있을지 고민

지금 배우고 있는 스터디 과정을 실무에 적용해보고 싶은데 기반 지식이 너무나 부족합니다.

무엇보다 AWS와 네트워크 관련 지식이 더욱 필요하다고 생각되는데, 자투리 시간을 활용해 공부할 수 있는 시간을 더 확보해야겠습니다.

