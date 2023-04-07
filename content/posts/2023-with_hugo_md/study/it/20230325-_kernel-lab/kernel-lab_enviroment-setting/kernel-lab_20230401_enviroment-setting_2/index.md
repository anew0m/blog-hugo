---
date: 2023-04-01T21:52:07+09:00
lastmod: 2023-04-07T23:35:07+09:00
author: "MinHo Jung"
authorLink: ""
license: "CC BY-NC-ND 4.0"
draft: false
toc: true
hiddenFromHomePage: false
hiddenFromSearch: false
linkToMarkdown: false
rssFullText: false

title: "리눅스 커널 분석 환경 구축 - 2.OS기본설정"
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

네트워크 설정을 위해 Oracle VM VirtualBox(이하 버추얼 박스)에서 네트워크 설정을 NAT 방식과 어댑터브릿지 방식 등이 존재하는데, 본 글에선 공유기 IP 관리 편의를 위해 NAT 방식으로 진행하겠습니다.



### 2.2 공통 설정

- 제 경우 우분투 내 기본 터미널이 안열리는 문제가 있었습니다. 이 경우 'Ubuntu Software'를 실행해서 터미널을 내려 받으면 기본 터미널 외 다른 터미널을 이용할 수 있습니다.
  - ![image-20230407212623401](files/img/image-20230407212623401.png)
- '{user_id} is not in the sudoers file.' 오류가 발생할 경우 다음 링크에서 참조 후 조치 가능합니다.
  - https://blog.servis.co.kr/index.php/2020/01/29/is-not-in-the-sudoers-file-this-incident-will-be-reported/
  - https://lifegoesonme.tistory.com/460
  - https://starseeker711.tistory.com/176
  - ![image-20230407213844263](files/img/image-20230407213844263.png)

#### 2.2.1 패키지 관리

1. apt-get

   - apt-get update
     - ubuntu
       - ![image-20230407213924167](files/img/image-20230407213924167.png)
     - 
   - apt-get upgrade
     - ubuntu (약 10분 소요)
       - ![image-20230407213955315](files/img/image-20230407213955315.png)
       - ![image-20230407215036581](files/img/image-20230407215036581.png)

2. 기타 패키지 설치
   - git
     
     - ubuntu
       - sudo apt-get install git
         - ![image-20230407215636776](files/img/image-20230407215636776.png)
     
   - vim

     - vim 설치

       - ubuntu
         - sudo apt-get install vim
           - ![image-20230407215414452](files/img/image-20230407215414452.png)

     - vim vundle 설치

       - 폴더 생성
         - mkdir ~/.vim && mkdir ~/.vim/bundle/
           - ![image-20230407220127120](files/img/image-20230407220127120.png)
       - VundleVim 깃 클론
         - git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
           - ![image-20230407220345121](files/img/image-20230407220345121.png)
       - .vimrc 에 내용 추가
         - 참고: https://github.com/VundleVim/Vundle.vim
           - ![image-20230407220959503](files/img/image-20230407220959503.png)
       - vim 실행 후 Plugin 설치
         - vim 실행 후 ':PluginInstall' 입력
           - ![image-20230407221126227](files/img/image-20230407221126227.png)
           - ![image-20230407221150281](files/img/image-20230407221150281.png)
           - ![image-20230407221543571](files/img/image-20230407221543571.png)

     - ctags 설치

       - ctags 설치

         - sudo apt-get install exuberant-ctags
           - ![image-20230407221652609](files/img/image-20230407221652609.png)

       - easytags플러그인 설치

         - .vimrc 에 easytags 관련 플러그인 추가

           - Plugin 'xolox/vim-misc'
           - Plugin 'xolox/vim-easytags'
             - ![image-20230407222621556](files/img/image-20230407222621556.png)

         - vim 실행 후 ':PluginInstall' 입력

           - ![image-20230407222712472](files/img/image-20230407222712472.png)

         - .vimrc 설정 정보 추가

           - ```shell
             set tag=./tags;/
             " easy-tag
             " tags를 비동기로 불러와준다. (필수) tag사이즈가 커지게 되면 vim이 블록되는 시간이 길어져서 답답하다
             let g:easytags_async=1
             " highlight를 켜면 좋지만 이것도 속도가 느려진다.
             let g:easytags_auto_highlight = 0 
             " struct의 멤버변수들도 추적이 된다.
             let g:easytags_include_members = 1
             " 현재 프로젝트에서 쓰는 tags파일을 우선 로드하고 없을 경우 global tags를 로드한다.
             let g:easytags_dynamic_files = 1
             ```

             - ![image-20230407223240332](files/img/image-20230407223240332.png)

     - cscope 설치

       - cscope 설치

         - sudo apt-get install cscope
           - ![image-20230407223323523](files/img/image-20230407223323523.png)

       - cscope 플러그인 설치

         - .vimrc 에 cscope 관련 플러그인 추가

           - Plugin 'ronakg/quickr-cscope.vim'

             - ![image-20230407223706838](files/img/image-20230407223706838.png)
             - vim 실행 후 ':PluginInstall' 입력
             - ![image-20230407223734302](files/img/image-20230407223734302.png)

           - .vimrc 설정 정보 추가

             - ```shell
               function! LoadCscope()
                 let db = findfile("cscope.out", ".;")
                 if (!empty(db))
                   let path = strpart(db, 0, match(db, "/cscope.out$"))
                   set nocscopeverbose " suppress 'duplicate connection' error
                   exe "cs add " . db . " " . path
                   set cscopeverbose
                 " else add the database pointed to by environment variable 
                 elseif $CSCOPE_DB != "" 
                   cs add $CSCOPE_DB
                 endif
               endfunction
               au BufEnter /* call LoadCscope()
               ```

               - ![image-20230407223955680](files/img/image-20230407223955680.png)

     - tagbar 설치

       - .vimrc 에 tagbar 관련 플러그인 추가
         - Plugin 'majutsushi/tagbar'
           - ![image-20230407224350188](files/img/image-20230407224350188.png)
         - vim 실행 후 ':PluginInstall' 입력
           - ![image-20230407224417787](files/img/image-20230407224417787.png)

   - ssh

     - ssh 설치
       - sudo apt-get install openssh-server
         - ![image-20230407224810919](files/img/image-20230407224810919.png)
         - ![image-20230407224833921](files/img/image-20230407224833921.png)

   - ufw

     - ssh 접속 포트 방화벽 허용 설정
       - sudo ufw allow ssh
         - ![image-20230407225141361](files/img/image-20230407225141361.png)



#### 2.2.2 공유폴더

1. 게스트 확장 기능 설치

   - 이미지 삽입

     - ![image-20230407225823543](files/img/image-20230407225823543.png)

   - 이미지 설치

     - ```shell
       sudo mount /dev/cdrom /cdrom
       sudo cd /cdrom
       sudo ./VBoxLinuxAdditions.run
       ```

       - ![image-20230407230606050](files/img/image-20230407230606050.png)

2. 공유폴더 설정

   - 공유폴더를 마운트할 폴더 생성

     - sudo mkdir /mnt/{folder_name}

       - 제 경우엔 /mnt 하위에 './host/_share_folder ' 로 생성했습니다.

       - ![image-20230407225345415](files/img/image-20230407225345415.png)

   - 버추얼 박스에서 공유폴더 설정

     - ![image-20230407231052538](files/img/image-20230407231052538.png)
     - ![image-20230407231102715](files/img/image-20230407231102715.png)

   - 설정을 마친뒤 재부팅 후 그룹 부여 확인

     - 그룹 부여
       - sudo adduser {사용자} vboxsf
       - 제 경우 'sudo adduser study vboxsf'
         - ![image-20230407232257789](files/img/image-20230407232257789.png)

   - 설정을 마친뒤 재부팅 후 공유폴더 확인

     - 폴더 이동 및 테스트 파일 생성
       - cd /mnt/host/_share_folder
         - ![image-20230407232745106](files/img/image-20230407232745106.png)
       - echo aaa > test.txt
         - ![image-20230407232836616](files/img/image-20230407232836616.png)
         - ![image-20230407232903875](files/img/image-20230407232903875.png)



#### 2.2.3 네트워크 설정

네트워크는 크게 NAT 방식과 어댑터 브릿지 방식이 있습니다. 이 두가지에 대해 설명드리지만 저는 IP 관리의 편의를 위해 NAT 방식으로 진행하겠습니다.



1. NAT 방식
   - NAT 방식은 버추얼 박스를 운용중인 컴퓨터 하위에 사설(Private) IP로 네트워크가 구성됩니다.
     - ![image-20230407204831377](files/img/image-20230407204831377.png)
   - 따라서 공유기에서 버추얼 박스를 운용중인 컴퓨터에 대해 포트포워딩 설정을 해주어야합니다.
     - ![image-20230407205529087](files/img/image-20230407205529087.png)
     - ![image-20230407205538585](files/img/image-20230407205538585.png)
   - 그리고 버추얼 박스에서 포트포워딩 설정을 해주어야 합니다.
     - ![image-20230407210252957](files/img/image-20230407210252957.png)
   - 접속 확인
     - ![image-20230407233514470](files/img/image-20230407233514470.png)
2. 어댑터 브릿지 방식
   - 어댑터 브릿지 방식은 버추얼 박스를 운용중인 컴퓨터와 동일한 레벨로 IP가 할당됩니다.
     - ![image-20230407210506960](files/img/image-20230407210506960.png)
   - 따라서 공유기에서 가상 컴퓨터 IP에 대해 포트포워딩을 해주어야하며, 가상 컴퓨터 IP를 가급적 고정 IP로 설정 해주시는 것이 좋습니다.
     - ![image-20230407205529087](files/img/image-20230407205529087.png)
     - ![image-20230407205538585](files/img/image-20230407205538585.png)



### 2.9 참고자료

- 우분투 오류 is not in the sudoers file. This incident will be reported.
  - https://blog.servis.co.kr/index.php/2020/01/29/is-not-in-the-sudoers-file-this-incident-will-be-reported/
- 리눅스 커널 분석을 위한 vim, ctag, cscope, tagbar 세팅
  - https://jen6.tistory.com/119
