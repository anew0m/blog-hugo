---
date: 2023-04-09T19:05:44+09:00
lastmod: 2023-04-09T19:05:44+09:00
author: "MinHo Jung"
authorLink: ""
license: "CC BY-NC-ND 4.0"
draft: false
toc: true
hiddenFromHomePage: false
hiddenFromSearch: false
linkToMarkdown: false
rssFullText: false

title: "커널분석LAB 3주차 회고(4L)"
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
- 스터디 구성원간 활발한 소통

스터디에 참여하시는 분들이 점점 질문 빈도가 잦아지고 있습니다. 저도 예상하지 못했던 부분에 대해 질문을 하시기도하고 질문에 대한 답변을 랩짱님 외 다른 분들이 답변 해주시기도 하는 모습이 좋았습니다.




## Lacked: 아쉬웠던 점
- 체계적인 학습 진도

평소 체계적으로 진도를 나아가는게 좋다고 생각합니다. 하지만 코드 분석간 궁금증이 생겼을 때 좀 더 깊게 나가지 못했던게 조금 아쉬웠습니다. 이번 주제가 끝난 후 타 주제에서 심도 있게 다뤄보신다고 하셨는데 당장 궁금한걸 해결 못하니 아쉬웠습니다.

> - volatile 은 내부 동작이 어떻게 되길래 동시에 접근하지 못하게 막을 수 있는걸까?
> - 다른 CPU의 메모리? 메모리를 관리하는 주체는 어디일까? OS가 관리하는 줄 알았는데..
> - 왜 메모리 재할당보다 메모리 재활용이 더 빠른걸까?




## Learned: 배운 점
- volatle 키워드를 이용한 동시성 제어

다중 CPU 이용시 동시성을 제어하기 위해 'WRITE_ONCE()' 나 'READ_ONCE()'와 같은 함수로 여러 CPU가 동시에 접근하지 못하게 차단합니다. 또 volatle 는 코드 최적화를 통해 임의로 코드가 변환되는 걸 막는다고 합니다.




## Longed for: 앞으로 바라는 점
- 코드에 대한 설명을 좀 더 추상화하면 이해하기 쉬울것 같습니다.

링크드 리스트 코드에 대한 설명을 지속적으로 하고 있는데, 학부 때 배운 자료구조의 흐름과 크게 다르지 않다고 느낍니다. 처음 접근하는 분들이 이해하는데 어려움이 있다면 물리적인 메모리 기준이 아니라 코드상의 논리적은 기준으로 그림을 도식화 한다면 한결 이해하기 쉬울 것 같습니다.





