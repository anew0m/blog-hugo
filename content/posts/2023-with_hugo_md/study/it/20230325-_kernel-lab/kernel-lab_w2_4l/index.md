---
date: 2023-04-01T17:35:39+09:00
lastmod: 2023-04-01T21:49:39+09:00
author: "MinHo Jung"
authorLink: ""
license: "CC BY-NC-ND 4.0"
draft: false
toc: true
hiddenFromHomePage: false
hiddenFromSearch: false
linkToMarkdown: false
rssFullText: false

title: "커널분석LAB 2주차 회고(4L)"
subtitle: ""
description: ""

images: []
featuredImage: ""
featuredImagePreview: ""

categories: ["IT/Kernel", "회고"]
tags: ["Study-Group-Offline", "Modulabs", "커널연구회", "JaeJoon Jung/KernelLab"]

alias: []
series: []
---

```
- 특정 활동에 대해 느낀 생각과 경험을 중심으로 회고를 진행하는 방식
- 주로 협업 프로젝트 진행 과정의 특정 지점(milestone)에서 구성원들과 중간 점검을 할 때 더욱 편리하게 쓸 수 있을 것 같음
```

## Liked: 좋았던 점
- 스터디의 방향성

스터디는 2~3개월 기초를 다진뒤 심도 있게 코드 분석을 진행할 예정이라고 하셨습니다.

그리고 상대적으로 코드가 빈번하게 변경되는 디바이스 보다는 장기적인 관점으로 스케줄러나 메모리, 알고리즘 분야에 대한 분석으로 진행할 계획이라고 하셔서 좋았습니다.




## Lacked: 아쉬웠던 점
- Linked-List 관련 코드 분석

코드에 대한 설명을 해주셨지만 조금 아쉬웠습니다. Linked-List에 대한 흐름을 알고 있어서 이해하는데 어려움은 없었지만 해당 내용을 모른다면 조금 따라가기 어렵지 않을까 생각했습니다.




## Learned: 배운 점
- Linked-List 관련 코드 및 동작 분석

Linked-List는 이미 알고 있었지만, 탐색하는 코드에서 동시성을 고려한 Locking 탐색이 인상적이었습니다.




## Longed for: 앞으로 바라는 점
- 발표 주제 선정 필요

랩짱님께선 스터디원의 적극적인 참여를 위해 커널관련 자유 주제로 발표하길 원하셨습니다. 발표는 매달 2주차, 4주차에 이뤄지고 순서는 스터디를 신청한 순서로 진행할 예정이라고 합니다. 

제 경우 7월 2주차에 발표할 예정인데 관련해서 주제 선정이 필요합니다. 

스터디간 떠오른 아이디어는 커널 내 알고리즘이나 스케줄러 코드가 변경된 이력에 대해 변경된 사유와 변경시 이점(성능 등)에 대한 분석은 어떨까 생각했습니다. 그리고 분석간 시사점이 있다면 도출하고 향후 연구 방향도 제시한다면 좋을 것 같다는 생각을 했습니다.



- 리눅스 커널 관련 코드 규칙이 있는지

리눅스 커널 코드를 보면 작성된 코드에 규칙이 있는 것 같습니다. 랩짱님께서 코드 설명간 종종 설명해 주시지만 정리된 문서가 있다면 좋을것 같아서 찾아보려고 합니다.



