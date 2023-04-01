---
date: 2023-04-01T21:52:07+09:00
lastmod: 2023-04-01T22:45:07+09:00
author: "MinHo Jung"
authorLink: ""
license: "CC BY-NC-ND 4.0"
draft: false
toc: true
hiddenFromHomePage: false
hiddenFromSearch: false
linkToMarkdown: false
rssFullText: false

title: "(작성중) 리눅스 커널 분석 환경 구축 - 2.OS기본설정"
subtitle: ""
description: ""

images: []
featuredImage: ""
featuredImagePreview: ""

categories: ["IT/Kernel"]
tags: ["Study-Group-Offline", "Modulabs", "커널연구회", "JaeJoon Jung/KernelLab"]
---



## 2. OS 기본 설정

### 2.1 개요

이번 OS 기본 설정 부분에선 패키지를 업데이트하고, SSH 접속과 관련된 네트워크를 설정하려고 합니다.

네트워크 설정을 위해 Oracle VM VirtualBox(이하 VB)에서 네트워크 설정을 NAT 방식과 어댑터브릿지 방식 등이 존재하는데, 본 글에선 IP 관리 편의를 위해 NAT 방식으로 진행하겠습니다.



### 2.2 공통 설정

#### 2.2.1 패키지 관리

1. apt-get
   - apt-get update
   - apt-get upgrade

2. 기타 패키지 설치
   - vim
     - vim vundle 설치
     - ctags 설치
     - cscope 설치
     - tagbar 설치
   - ufw
   - ssh
   - git



#### 2.2.2 네트워크 설정



### 2.9 참고자료

- 리눅스 커널 분석을 위한 vim, ctag, cscope, tagbar 세팅
  - https://jen6.tistory.com/119
