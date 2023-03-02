---
date: 2023-03-02T20:16:40+09:00
lastmod: 2023-03-02T20:16:40+09:00
author: "MinHo Jung"
authorLink: ""
license: "CC BY-NC-ND 4.0"
draft: false
toc: true
hiddenFromHomePage: false
hiddenFromSearch: false
linkToMarkdown: false
rssFullText: false

title: "PKOS 7주차 - 보안"
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

[PKOS 7주차 - 회고(4L)](../pkos_w7_4l)

[PKOS 7주차 - 보안](../pkos_w7_hands-on)

---

왜 그런지 모르겠는데 이미지 위아래로 여백이 생깁니다.

에디터에서는 안그러는데 왜 그런지 좀 찾아봐야겠습니다.

본 글은 초안이기에 다듬는 과정에서 내용이 수정될 수 있습니다. 

---

# PKOS 7주차



## 들어가기전에

본 내용은 `CloudNet@` 팀에서 진행하는 `쿠버네티스 실무 실습` 스터디를 기반으로 작성된 내용입니다.

또한 개념 설명에서 사용된 이미지의 출처는 스터디 학습 자료에서 가져온 것을 밝힙니다.

- 참조: https://www.notion.so/AWS-EKS-VPC-CNI-1-POD-f89e3e5967b24f8c9aa5bfaab1a82ceb
- 참조: ‘[24단계 실습으로 정복하는 쿠버네티스 - 이정훈 저](https://product.kyobobook.co.kr/detail/S000200152893)’ 책을 기준하여 정리

7주차에는 보안과 관련된 실습을 진행했습니다.



### 과제 수행결과

- 과제 수행과정은 본문에 자세히 기술했습니다.

#### 과제1

- 목표 : 파드에서 EC2 메타데이터의 IAM Role 토큰 정보를 활용하여(boto3), 스터디에서 소개한 것 이외의 다른 AWS 서비스(혹은 Action)를 사용 후 코드나 스샷을 올려주세요

- 수행결과
  - ![image-20230303025504353](files/img/image-20230303025504353.png)

#### 과제2

- 목표 : 책 398~400페이지 - kubescape armo 웹 사용 후 관련 스샷을 올려주세요

- 수행결과
  - ![image-20230303031246067](files/img/image-20230303031246067.png)

#### 과제3

- 목표 : polaris 관련 실습(아무거나) 후 관련 스샷을 올려주세요

- 수행결과
  - ![image-20230303034436048](files/img/image-20230303034436048.png)

#### 과제4

- 목표 : 신규 서비스 어카운트(SA) 생성 후 '클러스터 수준(모든 네임스페이스 포함)에서 읽기 전용'의 권한을 주고 테스트 후 코드나 스샷을 올려주세요

- 수행결과
  - ![image-20230303041302081](files/img/image-20230303041302081.png)



## 1. 실습 환경 세팅

### 1.1 구성 환경

- 사전 준비
  - AWS 계정, SSH 키 페어, IAM 계정 생성 후 키, S3 버킷
- 전체 구성도
  - 기본 구성 환경은 1주차 내용과 동일
- 실습 환경 내용
  - CloudFormation 스택 실행 시 파라미터를 기입하면, 해당 정보가 반영되어 배포됩니다.
  - VPC는 kOps 배포를 위한 EC2가 위치할 MyVPC 1개와 실제 kOps 가 배포되어 구동되는 VPC 1개로 총 2개가 생성됩니다.
  - CloudFormation 에 EC2의 UserData 부분(Script 실행)으로 AWS kOps 설치를 진행합니다.
  - 마스터 노드 1대, 워커 노드는 기본은 2대로 구성됩니다.
  - 실습 환경은 지난 주차와 다르게 노드 사양은 c5.2xlarge(vCPU 8, Memory 16GiB) 으로 진행 됩니다.



### 1.2 실습 환경

- 본 실습은 **미국 동부(버지니아 북부) `us-east-1`** 에서 진행됩니다.

#### 1.2.1 S3 버킷 생성

1. S3 URL 접속
   - https://s3.console.aws.amazon.com/s3/buckets?region=us-east-1
2. 버킷 만들기 버튼 클릭
   - ![image-20230302225609235](files/img/image-20230302225609235.png)
3. 버킷 만들기
   - 버킷 이름 설정 후 기본 설정 그대로 버킷 만들기 버튼 클릭
     - 버킷 이름 : 20230302-learn-s3-mybucket
   - ![image-20230303041829667](files/img/image-20230303041829667.png)



#### 1.2.2 Cloud Formation을 이용한 kOps 생성(이하 배포)

1. Cloud Formation URL 접속 및 스택 생성 버튼 클릭

   - https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks
   - ![image-20230302225659561](files/img/image-20230302225659561.png)

2. 스택 생성 - 1단계 스택 생성

   - 템플릿 소스 URL 입력 - Amazon S3 URL
     - https://s3.ap-northeast-2.amazonaws.com/cloudformation.cloudneta.net/K8S/kops-oneclick2.yaml
       - 이전 주차와 다르게 템플릿 소스가 변경되었으니 입력시 주의
   - ![image-20230302225719229](files/img/image-20230302225719229.png)

3. 스택 생성 - 2단계 스택 세부 정보 지정

   - 스택 이름

     - 스택 이름 : mkops
     - ![image-20230302225725321](files/img/image-20230302225725321.png)

   - 파라미터

     - <<<<< Deploy EC2 : kops-ec2 >>>>>

       - 설정 설명
         - KeyName : kops-ec2에 SSH 접속을 위한 SSH 키페어 선택 *<- 미리 SSH 키 생성 해두자!*
         - MyIamUserAccessKeyID : 관리자 수준의 권한을 가진 IAM User의 액세스 키ID 입력
         - MyIamUserSecretAccessKey : 관리자 수준의 권한을 가진 IAM User의 시크릿 키ID 입력 <- 노출되지 않게 보안 주의
         - SgIngressSshCidr : kops-ec2에 SSH 접속 가능한 IP 입력 (집 공인IP/32 입력), 보안그룹 인바운드 규칙에 반영됨
         - LatestAmiId : kops-ec2에 사용할 AMI는 아마존리눅스2 최신 버전 사용, 기본값 그대로 사용
       - 설정 내용
         - 사용자 설정 입력
       - ![image-20230302225734683](files/img/image-20230302225734683.png)

     - <<<<< AWS kOps Config >>>>>

       - 설정 설명

         - KubernetesVersion : 쿠버네티스 설치 버전 (기본 v1.24.9) ⇒ 변경 가능

         - ClusterBaseName : kOps 클러스터 이름이며, 사용하게될 도메인 이름이다. ‘퍼블릭 도메인’ or ‘프라이빗 도메인’ or ‘Gossip DNS’ 가능

         - S3StateStore : kOps 클러스터의 설정/상태 정보가 저장될 S3 버킷의 이름을 지정 *← 미리 S3 버킷을 생성 해두자!*

         - MasterNodeInstanceType & WorkerNodeInstanceType: 마스터(기본 t3.medium) & 워커 노드 EC2 인스턴스의 타입 (기본 t3.medium) ⇒ 변경 가능

         - WorkerNodeCount : 워커노드의 갯수를 입력 ⇒ 변경 가능

         - VpcBlock : kOps 배포되고 동작할 VPC 네트워크 대역, 기본값 그대로 사용

       - 설정 내용

         - 다음 설정 외 기본 설정 사용
           - ClusterBaseName
             - learn-dc.link
           - S3StateStore : 위에 생성한 버킷 명칭 입력
             - 20230302-learn-s3-mybucket
           - MasterNodeInstanceType & WorkerNodeInstanceType
             - t3.xlarge

       - ![image-20230302225742318](files/img/image-20230302225742318.png)

     - <<<<< Region AZ >>>>>

       - 설정 설명
         - TargetRegion : kOps를 배포할 리전
         - AvailabliltyZone1 : kOps를 배포할 리전의 가용 영역
         - AvailabliltyZone2 : kOps를 배포할 리전의 가용 영역
       - 설정 내용
         - TargetRegion : us-east-1
         - AvailabliltyZone1 : us-east-1a
         - AvailabliltyZone2 : us-east-1c
       - ![image-20230302225807195](files/img/image-20230302225807195.png)

   - 다음 버튼 클릭

     - ![image-20230302225812388](files/img/image-20230302225812388.png)

4. 스택 생성 - 3단계 스택 옵션

   - 기본 설정 그대로 다음 버튼 클릭
     - ![image-20230302225816030](files/img/image-20230302225816030.png)

5. 스택 생성 - 4단계 mkops 검토

   - 스택 파라미터 설정 확인
     - ![image-20230302225824883](files/img/image-20230302225824883.png)
   - 기본 설정 그대로 전송 버튼 클릭
     - ![image-20230302225830948](files/img/image-20230302225830948.png)

6. 스택 생성 확인 및 접속 IP 확인

   - 스택 생성 확인
     - ![image-20230302225840537](files/img/image-20230302225840537.png)
   - 접속 IP 확인
     - KOPSEC2IP : 3.84.234.199
     - ![image-20230302225849078](files/img/image-20230302225849078.png)



#### 1.2.3 kOps 배포 확인

- 스택 생성 후 약 13분 후 접속
  - 내부적으로 설치되는데 시간 소요됨

##### 1. 접속 후 기본 설정

- 마스터노드 SSH 접속 - Putty

  - ![image-20230303003648811](files/img/image-20230303003648811.png)

- default NS 진입

  - ```bash
    kubectl ns default
    ```

    - ![image-20230303003706750](files/img/image-20230303003706750.png)



### 1.3 EC2 instance profiles 설정 및 Metrics-server 확인

#### 1.3.1 EC2 instance profiles 설정

- EC2 instance profiles 에 IAM Policy 추가(attach)

  - ```bash
    # 처음 입력 시 적용이 잘 안될 경우 다시 한번 더 입력 하자! - IAM Role에서 새로고침 먼저 확인!
    aws iam attach-role-policy --policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy --role-name masters.$KOPS_CLUSTER_NAME
    aws iam attach-role-policy --policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy --role-name nodes.$KOPS_CLUSTER_NAME
    aws iam attach-role-policy --policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AllowExternalDNSUpdates --role-name masters.$KOPS_CLUSTER_NAME
    aws iam attach-role-policy --policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AllowExternalDNSUpdates --role-name nodes.$KOPS_CLUSTER_NAME
    ```

    - ![image-20230303003727199](files/img/image-20230303003727199.png)



#### 1.3.2 Metrics-server 확인

- 메트릭 서버 확인 : 메트릭은 15초 간격으로 cAdvisor를 통하여 가져옴

  - ```bash
    kubectl top node
    kubectl top pod -A
    ```

    - ![image-20230303003749378](files/img/image-20230303003749378.png)
    - ![image-20230303003757948](files/img/image-20230303003757948.png)



## 2. EC2 IAM Role & Metadata

### 2.1 개요

- 참고 링크
  - 쿠버네티스 Secret을 해킹하는 엔지니어를 위한 안내서 - https://blog.kaonmir.site/10
  - [매뉴얼] Boto3가 AWS의 자격증명(Credentials)을 확인하는 순서 .from Python - https://tech.cloud.nongshim.co.kr/2021/03/12/boto3%EA%B0%80-aws%EC%9D%98-%EC%9E%90%EA%B2%A9%EC%A6%9D%EB%AA%85credentials%EC%9D%84-%ED%99%95%EC%9D%B8%ED%95%98%EB%8A%94-%EC%88%9C%EC%84%9C-from-python/
    - ![image-20230303011733038](files/img/image-20230303011733038.png)



### 2.2 실습

#### 2.2.1 워커 노드 1대 EC2 메타데이터 보안 제거

- 적용 및 재생성 5분 시간 소요

- kOps 클러스터 편집

  - ```bash
    # kops edit ig nodes-ap-northeast-2a
    kops edit ig nodes-us-east-1a
    
    # ...
    # 아래 3줄 제거
    # spec:
    #   instanceMetadata:              <- 제거
    #     httpPutResponseHopLimit: 1   <- 제거
    #     httpTokens: required         <- 제거
    # ...
    ```

    - ![image-20230303004346048](files/img/image-20230303004346048.png)

- 업데이트 적용 : 노드1대 롤링업데이트

  - ```bash
    kops update cluster --yes && echo && sleep 3 && kops rolling-update cluster --yes
    ```

    - ![image-20230303004449763](files/img/image-20230303004449763.png)
    - ![image-20230303005408594](files/img/image-20230303005408594.png)



#### 2.2.2 파드에서 EC2 메티데이터 사용 가능 확인

- netshoot-pod 생성

  - ```bash
    cat <<EOF | kubectl create -f -
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: netshoot-pod
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: netshoot-pod
      template:
        metadata:
          labels:
            app: netshoot-pod
        spec:
          containers:
          - name: netshoot-pod
            image: nicolaka/netshoot
            command: ["tail"]
            args: ["-f", "/dev/null"]
          terminationGracePeriodSeconds: 0
    EOF
    ```

    - ![image-20230303005453679](files/img/image-20230303005453679.png)

- 파드 이름 변수 지정

  - ```bash
    PODNAME1=$(kubectl get pod -l app=netshoot-pod -o jsonpath={.items[0].metadata.name})
    echo $PODNAME1
    PODNAME2=$(kubectl get pod -l app=netshoot-pod -o jsonpath={.items[1].metadata.name})
    echo $PODNAME2
    ```

    - ![image-20230303005541559](files/img/image-20230303005541559.png)

- EC2 메타데이터 정보 확인

  - ```bash
    kubectl exec -it $PODNAME1 -- curl 169.254.169.254 ;echo
    kubectl exec -it $PODNAME2 -- curl 169.254.169.254 ;echo
    ```

    - ![image-20230303010339088](files/img/image-20230303010339088.png)

- 파드1에서 EC2 메타데이터 정보 확인

  - ```bash
    kubectl exec -it $PODNAME1 -- curl 169.254.169.254/latest ;echo
    kubectl exec -it $PODNAME1 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/ ;echo
    kubectl exec -it $PODNAME1 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/nodes.$KOPS_CLUSTER_NAME | jq
    ```

    - ![image-20230303010406901](files/img/image-20230303010406901.png)

- 파드2에서 EC2 메타데이터 정보 확인

  - ```bash
    kubectl exec -it $PODNAME2 -- curl 169.254.169.254/latest ;echo
    kubectl exec -it $PODNAME2 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/ ;echo
    kubectl exec -it $PODNAME2 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/nodes.$KOPS_CLUSTER_NAME | jq
    ```

    - ![image-20230303010516224](files/img/image-20230303010516224.png)

- 메타데이터 메모해두기

  - ```json
    {
      "Code": "Success",
      "LastUpdated": "2023-03-02T15:45:21Z",
      "Type": "AWS-HMAC",
      "AccessKeyId": "ASIAZ2FILNMLU2KBYJ4Q",
      "SecretAccessKey": "JrjrVVWzWjNdKSn7vrdcnyxC3Zc8Y5f7u3VPzvSG",
      "Token": "IQoJb3JpZ2luX2VjEAAaCXVzLWVhc3QtMSJHMEUCIQDaiskr3mHtaNN0/WFSMUv2Ole4zxizOeQBLP2h2pZ7zAIgJmTySF7gino+lzFnXx/xE3aptK/7arVheJPWWGfNX8oqxQUIqf//////////ARABGgw2NzQ2NjI5MzUzMTkiDAXCj999T5Mcmtkf+yqZBSJBfNctXHcp76DeoNYkP45rQWu9T/KTradQeBL+isBCrpJtj1IivkP0zel4MnjJNkPuhGb0bjMN880Fp3TO2VrcLoaMJM+lQAdAwYdC8hEexnilhdGSqbLz6HLXiWABWv0PTqdpBkEIEf/6QllZxM+ODc6bDjjwFfQqu+1cKOnEaa9iLRVbKkOVbzd2u3AJlYugGQNbwFkZF1/bxo1QbwPLHYxVthlz6Hx4ubpMwcvoJFN5OQmh5HsnG1dimXtWEJnVaoIHA+n+TRSQsceuhL4zH5uQ5MZ6v4vYsnLQwM3elYag46uJXuofNnkitDHvZqWcetACx6jGkzdAhnlSrK85xloG+0XP1wv/czBldQsr8Jl1Ja9dTw+DHUJUU0rP8sXA49ZrzPMtc2taNTJKlxsnlPnUvJn5j7DHddyLJ35HN2MmoxoBklpo0VLCLPiEIgXn3CHcXDrS/KGJcYEJWRzJKvyStUSM/zVMz8yqiiyAZoqf19xdDnrxrvXgxG6SrHzW9ZRa5dSghtVyS9p8fIl1Grj/HSJFlwrGj46iuta4H9E8fielWFP4ggO6uU1APj+4qa5vJKicwy2IeN/kU2OjvDlotWGdfYKRK8npPE0DERekzYIc1Ap7NAeRECuLnemwEnr02Cj9bFV0PilKKSFGaeaULhmH/DO6YOk0bxq/r9YUz+85HHRQufK8AMxtghX5yI6QLsSRi6P8aK/Uck1YvOLqhinWMmvCNvm2UmXETmzQc3pJiC8xSx/L9oOmp211OGMb6FKrQNGSNkmQEtg6wetCqBI7MNbJ8dqR5CpeDCkWFTy4I7eb9CQUZzVKeyDx8pnkcPT2fBe4zajHS+h8rN3l6KRGcUG7WK8QwIuA+quKtuqBRwJ7MIGJg6AGOrEBTP3tE8CzcmyPoh8VkhHLQIlyWCJSOAYhaPWaK7vgQbjlP6wqs7POo98iNGzbq2CM+AWfVyD6iVKuk0qOgbjANtWaJnMKeaqZrS54S26Ppu23mnpUNiAJbTDjHHEJKKp9bhG0CR4IiIqdQc3B00/YCQPng90gaBI72i/+S2w4v5JMpf2SL5CB9/skMxfM3YjIOUE5mAp+WvLjioFftoPgyDSUGGbSZgWiCyA22uQzMbok",
      "Expiration": "2023-03-02T22:20:05Z"
    }
    ```



#### 2.2.3 파드(컨테이너) 탈취 후 EC2 메타데이터의 IAM Role 토큰 정보를 활용해 python boto3를 통해 SDK로 AWS 서비스 강제 사용

- 참고 링크
  - Managing Amazon EC2 instances - https://boto3.amazonaws.com/v1/documentation/api/latest/guide/ec2-example-managing-instances.html
  - [pkos] 쿠버네티스 스터디 - 7주차 과제 - https://velog.io/@xgro/pkos-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%8A%A4%ED%84%B0%EB%94%94-7%EC%A3%BC%EC%B0%A8-%EA%B3%BC%EC%A0%9C#-%EA%B2%B0%EA%B3%BC

- boto3 사용을 위한 파드 생성

  - ```bash
    cat <<EOF | kubectl create -f -
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: boto3-pod
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: boto3
      template:
        metadata:
          labels:
            app: boto3
        spec:
          containers:
          - name: boto3
            image: jpbarto/boto3
            command: ["tail"]
            args: ["-f", "/dev/null"]
          terminationGracePeriodSeconds: 0
    EOF
    ```

    - ![image-20230303010557155](files/img/image-20230303010557155.png)

- 파드 이름 변수 지정

  - ```bash
    PODNAME1=$(kubectl get pod -l app=boto3 -o jsonpath={.items[0].metadata.name})
    echo $PODNAME1
    PODNAME2=$(kubectl get pod -l app=boto3 -o jsonpath={.items[1].metadata.name})
    echo $PODNAME2
    ```

    - ![image-20230303010645287](files/img/image-20230303010645287.png)

- 파드1에서 boto3 사용 - ec2 메타정보를 가져오는 보안 비활성화

  - ```bash
    kubectl exec -it $PODNAME1 -- sh
    # ------------
    cat <<EOF> ec2.py
    import boto3
    
    ec2 = boto3.client('ec2', region_name = 'us-east-1')
    response = ec2.describe_instances()
    print(response)
    EOF
    
    python ec2.py  # aws ec2 describe-vpcs
    exit
    # ------------
    ```

    - ![image-20230303011308107](files/img/image-20230303011308107.png)
    - ![image-20230303011314010](files/img/image-20230303011314010.png)

- 파드2에서 boto3 사용 - ec2 메타정보를 가져오는 보안 활성화

  - ```bash
    kubectl exec -it $PODNAME2 -- sh
    # ------------
    cat <<EOF> ec2.py
    import boto3
    
    ec2 = boto3.client('ec2', region_name = 'us-east-1')
    response = ec2.describe_instances()
    print(response)
    EOF
    
    python ec2.py  # aws ec2 describe-vpcs
    exit
    # ------------
    ```

    - ![image-20230303011345789](files/img/image-20230303011345789.png)

- 실습 완료 후 삭제

  - ```bash
    kubectl delete deploy boto3-pod
    ```

    - 

- **대응 방안** :  EC2 메타데이터 접속 제한 or AWS IRSA 사용



#### 2.2.4 기타

- 현재 ec2에 맵핑된 iam role 연동 정책(policy)를 사용할 수 있는 boto3 sdk를 사용해서 다른 서비스도 이용해보세요! ⇒ ELB 생성/삭제!
- awscli 파드를 추가하고, 획득한 IAM Role 토근을 활용해서 aws서비스를 강제 사용하는 실습을 추가해두자!



#### 2.2.5 (추가) awscli 파드를 추가하고, 획득한 IAM Role Token을 활용해서 AWS 서비스를 강제 사용

- AWS CLI v2 파드 생성 - [링크](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-cliv2-docker.html) [공식이미지링크](https://hub.docker.com/r/amazon/aws-cli)

- awscli 파드 생성

  - ```bash
    cat <<EOF | kubectl create -f -
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: awscli-pod
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: awscli-pod
      template:
        metadata:
          labels:
            app: awscli-pod
        spec:
          containers:
          - name: awscli-pod
            image: amazon/aws-cli
            command: ["tail"]
            args: ["-f", "/dev/null"]
          terminationGracePeriodSeconds: 0
    EOF
    ```

    - ![image-20230303011927464](files/img/image-20230303011927464.png)

- 파드 이름 변수 지정

  - ```bash
    APODNAME1=$(kubectl get pod -l app=awscli-pod -o jsonpath={.items[0].metadata.name})
    echo $APODNAME1
    APODNAME2=$(kubectl get pod -l app=awscli-pod -o jsonpath={.items[1].metadata.name})
    echo $APODNAME2
    ```

    - ![image-20230303011941690](files/img/image-20230303011941690.png)

- 파드에서 EC2 메타데이터 정보 확인

  - 파드1

    - ```bash
      kubectl exec -it $APODNAME1 -- curl 169.254.169.254/latest ;echo
      kubectl exec -it $APODNAME1 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/ ;echo
      kubectl exec -it $APODNAME1 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/nodes.$KOPS_CLUSTER_NAME | jq
      ```

      - ![image-20230303012324360](files/img/image-20230303012324360.png)

  - 파드2

    - ```bash
      kubectl exec -it $APODNAME2 -- curl 169.254.169.254/latest ;echo
      kubectl exec -it $APODNAME2 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/ ;echo
      kubectl exec -it $APODNAME2 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/nodes.$KOPS_CLUSTER_NAME | jq
      ```

      - ![image-20230303012245203](files/img/image-20230303012245203.png)

- 메타데이터 메모해두기 : 단 Expiration 시간이 지나면 Token은 유효하지 않음

  - ```bash
    {
      "Code": "Success",
      "LastUpdated": "2023-03-02T15:45:21Z",
      "Type": "AWS-HMAC",
      "AccessKeyId": "ASIAZ2FILNMLU2KBYJ4Q",
      "SecretAccessKey": "JrjrVVWzWjNdKSn7vrdcnyxC3Zc8Y5f7u3VPzvSG",
      "Token": "IQoJb3JpZ2luX2VjEAAaCXVzLWVhc3QtMSJHMEUCIQDaiskr3mHtaNN0/WFSMUv2Ole4zxizOeQBLP2h2pZ7zAIgJmTySF7gino+lzFnXx/xE3aptK/7arVheJPWWGfNX8oqxQUIqf//////////ARABGgw2NzQ2NjI5MzUzMTkiDAXCj999T5Mcmtkf+yqZBSJBfNctXHcp76DeoNYkP45rQWu9T/KTradQeBL+isBCrpJtj1IivkP0zel4MnjJNkPuhGb0bjMN880Fp3TO2VrcLoaMJM+lQAdAwYdC8hEexnilhdGSqbLz6HLXiWABWv0PTqdpBkEIEf/6QllZxM+ODc6bDjjwFfQqu+1cKOnEaa9iLRVbKkOVbzd2u3AJlYugGQNbwFkZF1/bxo1QbwPLHYxVthlz6Hx4ubpMwcvoJFN5OQmh5HsnG1dimXtWEJnVaoIHA+n+TRSQsceuhL4zH5uQ5MZ6v4vYsnLQwM3elYag46uJXuofNnkitDHvZqWcetACx6jGkzdAhnlSrK85xloG+0XP1wv/czBldQsr8Jl1Ja9dTw+DHUJUU0rP8sXA49ZrzPMtc2taNTJKlxsnlPnUvJn5j7DHddyLJ35HN2MmoxoBklpo0VLCLPiEIgXn3CHcXDrS/KGJcYEJWRzJKvyStUSM/zVMz8yqiiyAZoqf19xdDnrxrvXgxG6SrHzW9ZRa5dSghtVyS9p8fIl1Grj/HSJFlwrGj46iuta4H9E8fielWFP4ggO6uU1APj+4qa5vJKicwy2IeN/kU2OjvDlotWGdfYKRK8npPE0DERekzYIc1Ap7NAeRECuLnemwEnr02Cj9bFV0PilKKSFGaeaULhmH/DO6YOk0bxq/r9YUz+85HHRQufK8AMxtghX5yI6QLsSRi6P8aK/Uck1YvOLqhinWMmvCNvm2UmXETmzQc3pJiC8xSx/L9oOmp211OGMb6FKrQNGSNkmQEtg6wetCqBI7MNbJ8dqR5CpeDCkWFTy4I7eb9CQUZzVKeyDx8pnkcPT2fBe4zajHS+h8rN3l6KRGcUG7WK8QwIuA+quKtuqBRwJ7MIGJg6AGOrEBTP3tE8CzcmyPoh8VkhHLQIlyWCJSOAYhaPWaK7vgQbjlP6wqs7POo98iNGzbq2CM+AWfVyD6iVKuk0qOgbjANtWaJnMKeaqZrS54S26Ppu23mnpUNiAJbTDjHHEJKKp9bhG0CR4IiIqdQc3B00/YCQPng90gaBI72i/+S2w4v5JMpf2SL5CB9/skMxfM3YjIOUE5mAp+WvLjioFftoPgyDSUGGbSZgWiCyA22uQzMbok",
      "Expiration": "2023-03-02T22:20:05Z"
    }
    ```

- 파드 bash 실행 : 임시자격증명을 직접 입력하는 거라서 아무곳에서나 해도됨

  - ```bash
    # kubectl exec -it $APODNAME1 -- bash
    kubectl exec -it $APODNAME2 -- bash
    ```

    - 

- 위에서 출력된 AccessKeyId , SecretAccessKey , SessionToken 으로 임시자격증명 적용

  - ```bash
    export AWS_ACCESS_KEY_ID="ASIAZ2FILNMLU2KBYJ4Q"
    export AWS_SECRET_ACCESS_KEY="JrjrVVWzWjNdKSn7vrdcnyxC3Zc8Y5f7u3VPzvSG"
    export AWS_SESSION_TOKEN="IQoJb3JpZ2luX2VjEAAaCXVzLWVhc3QtMSJHMEUCIQDaiskr3mHtaNN0/WFSMUv2Ole4zxizOeQBLP2h2pZ7zAIgJmTySF7gino+lzFnXx/xE3aptK/7arVheJPWWGfNX8oqxQUIqf//////////ARABGgw2NzQ2NjI5MzUzMTkiDAXCj999T5Mcmtkf+yqZBSJBfNctXHcp76DeoNYkP45rQWu9T/KTradQeBL+isBCrpJtj1IivkP0zel4MnjJNkPuhGb0bjMN880Fp3TO2VrcLoaMJM+lQAdAwYdC8hEexnilhdGSqbLz6HLXiWABWv0PTqdpBkEIEf/6QllZxM+ODc6bDjjwFfQqu+1cKOnEaa9iLRVbKkOVbzd2u3AJlYugGQNbwFkZF1/bxo1QbwPLHYxVthlz6Hx4ubpMwcvoJFN5OQmh5HsnG1dimXtWEJnVaoIHA+n+TRSQsceuhL4zH5uQ5MZ6v4vYsnLQwM3elYag46uJXuofNnkitDHvZqWcetACx6jGkzdAhnlSrK85xloG+0XP1wv/czBldQsr8Jl1Ja9dTw+DHUJUU0rP8sXA49ZrzPMtc2taNTJKlxsnlPnUvJn5j7DHddyLJ35HN2MmoxoBklpo0VLCLPiEIgXn3CHcXDrS/KGJcYEJWRzJKvyStUSM/zVMz8yqiiyAZoqf19xdDnrxrvXgxG6SrHzW9ZRa5dSghtVyS9p8fIl1Grj/HSJFlwrGj46iuta4H9E8fielWFP4ggO6uU1APj+4qa5vJKicwy2IeN/kU2OjvDlotWGdfYKRK8npPE0DERekzYIc1Ap7NAeRECuLnemwEnr02Cj9bFV0PilKKSFGaeaULhmH/DO6YOk0bxq/r9YUz+85HHRQufK8AMxtghX5yI6QLsSRi6P8aK/Uck1YvOLqhinWMmvCNvm2UmXETmzQc3pJiC8xSx/L9oOmp211OGMb6FKrQNGSNkmQEtg6wetCqBI7MNbJ8dqR5CpeDCkWFTy4I7eb9CQUZzVKeyDx8pnkcPT2fBe4zajHS+h8rN3l6KRGcUG7WK8QwIuA+quKtuqBRwJ7MIGJg6AGOrEBTP3tE8CzcmyPoh8VkhHLQIlyWCJSOAYhaPWaK7vgQbjlP6wqs7POo98iNGzbq2CM+AWfVyD6iVKuk0qOgbjANtWaJnMKeaqZrS54S26Ppu23mnpUNiAJbTDjHHEJKKp9bhG0CR4IiIqdQc3B00/YCQPng90gaBI72i/+S2w4v5JMpf2SL5CB9/skMxfM3YjIOUE5mAp+WvLjioFftoPgyDSUGGbSZgWiCyA22uQzMbok"
    ```

    - ![image-20230303012708823](files/img/image-20230303012708823.png)

- ec2 정보 확인

  - ```bash
    aws ec2 describe-instances --region us-east-1 | head
    ```

    - ![image-20230303012738856](files/img/image-20230303012738856.png)

- vpc 정보 확인

  - ```bash
    aws ec2 describe-vpcs --region us-east-1 | head
    ```

    - ![image-20230303012801759](files/img/image-20230303012801759.png)

- 빠져나오기

  - ```bash
    exit
    ```

    - ![image-20230303012843449](files/img/image-20230303012843449.png)
    - ![image-20230303012921626](files/img/image-20230303012921626.png)



#### 2.2.6 AWS IRSA : 파드별 IAM Role 부여 - [kOps](https://kops.sigs.k8s.io/cluster_spec/#service-account-issuer-discovery-and-aws-iam-roles-for-service-accounts-irsa)

- Pod 별로 IAM 권한을 분리하는 경우와 그렇지 않은 경우
  - ![image-20230302232920985](files/img/image-20230302232920985.png)
    - 출처: https://tech.devsisters.com/posts/pod-iam-role/
- AWS EKS Service Account에 AWS IAM Role 부여
  - https://ssup2.github.io/theory_analysis/AWS_EKS_Service_Account_IAM_Role/



### 2.3 과제1

- 목표: 파드에서 EC2 메타데이터의 IAM Role 토큰 정보를 활용하여(boto3), 스터디에서 소개한 것 이외의 다른 AWS 서비스(혹은 Action)를 사용 후 코드나 스샷을 올려주세요

- 수행결과 및 과정

  - 노드 권한 확인

    - ![image-20230303021357830](files/img/image-20230303021357830.png)

    - AWSLoadBalancerControllerIAMPolicy 정책

      - ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "iam:CreateServiceLinkedRole"
                    ],
                    "Resource": "*",
                    "Condition": {
                        "StringEquals": {
                            "iam:AWSServiceName": "elasticloadbalancing.amazonaws.com"
                        }
                    }
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "ec2:DescribeAccountAttributes",
                        "ec2:DescribeAddresses",
                        "ec2:DescribeAvailabilityZones",
                        "ec2:DescribeInternetGateways",
                        "ec2:DescribeVpcs",
                        "ec2:DescribeVpcPeeringConnections",
                        "ec2:DescribeSubnets",
                        "ec2:DescribeSecurityGroups",
                        "ec2:DescribeInstances",
                        "ec2:DescribeNetworkInterfaces",
                        "ec2:DescribeTags",
                        "ec2:GetCoipPoolUsage",
                        "ec2:DescribeCoipPools",
                        "elasticloadbalancing:DescribeLoadBalancers",
                        "elasticloadbalancing:DescribeLoadBalancerAttributes",
                        "elasticloadbalancing:DescribeListeners",
                        "elasticloadbalancing:DescribeListenerCertificates",
                        "elasticloadbalancing:DescribeSSLPolicies",
                        "elasticloadbalancing:DescribeRules",
                        "elasticloadbalancing:DescribeTargetGroups",
                        "elasticloadbalancing:DescribeTargetGroupAttributes",
                        "elasticloadbalancing:DescribeTargetHealth",
                        "elasticloadbalancing:DescribeTags"
                    ],
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "cognito-idp:DescribeUserPoolClient",
                        "acm:ListCertificates",
                        "acm:DescribeCertificate",
                        "iam:ListServerCertificates",
                        "iam:GetServerCertificate",
                        "waf-regional:GetWebACL",
                        "waf-regional:GetWebACLForResource",
                        "waf-regional:AssociateWebACL",
                        "waf-regional:DisassociateWebACL",
                        "wafv2:GetWebACL",
                        "wafv2:GetWebACLForResource",
                        "wafv2:AssociateWebACL",
                        "wafv2:DisassociateWebACL",
                        "shield:GetSubscriptionState",
                        "shield:DescribeProtection",
                        "shield:CreateProtection",
                        "shield:DeleteProtection"
                    ],
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "ec2:AuthorizeSecurityGroupIngress",
                        "ec2:RevokeSecurityGroupIngress"
                    ],
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "ec2:CreateSecurityGroup"
                    ],
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "ec2:CreateTags"
                    ],
                    "Resource": "arn:aws:ec2:*:*:security-group/*",
                    "Condition": {
                        "StringEquals": {
                            "ec2:CreateAction": "CreateSecurityGroup"
                        },
                        "Null": {
                            "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
                        }
                    }
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "ec2:CreateTags",
                        "ec2:DeleteTags"
                    ],
                    "Resource": "arn:aws:ec2:*:*:security-group/*",
                    "Condition": {
                        "Null": {
                            "aws:RequestTag/elbv2.k8s.aws/cluster": "true",
                            "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                        }
                    }
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "ec2:AuthorizeSecurityGroupIngress",
                        "ec2:RevokeSecurityGroupIngress",
                        "ec2:DeleteSecurityGroup"
                    ],
                    "Resource": "*",
                    "Condition": {
                        "Null": {
                            "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                        }
                    }
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "elasticloadbalancing:CreateLoadBalancer",
                        "elasticloadbalancing:CreateTargetGroup"
                    ],
                    "Resource": "*",
                    "Condition": {
                        "Null": {
                            "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
                        }
                    }
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "elasticloadbalancing:CreateListener",
                        "elasticloadbalancing:DeleteListener",
                        "elasticloadbalancing:CreateRule",
                        "elasticloadbalancing:DeleteRule"
                    ],
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "elasticloadbalancing:AddTags",
                        "elasticloadbalancing:RemoveTags"
                    ],
                    "Resource": [
                        "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*",
                        "arn:aws:elasticloadbalancing:*:*:loadbalancer/net/*/*",
                        "arn:aws:elasticloadbalancing:*:*:loadbalancer/app/*/*"
                    ],
                    "Condition": {
                        "Null": {
                            "aws:RequestTag/elbv2.k8s.aws/cluster": "true",
                            "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                        }
                    }
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "elasticloadbalancing:AddTags",
                        "elasticloadbalancing:RemoveTags"
                    ],
                    "Resource": [
                        "arn:aws:elasticloadbalancing:*:*:listener/net/*/*/*",
                        "arn:aws:elasticloadbalancing:*:*:listener/app/*/*/*",
                        "arn:aws:elasticloadbalancing:*:*:listener-rule/net/*/*/*",
                        "arn:aws:elasticloadbalancing:*:*:listener-rule/app/*/*/*"
                    ]
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "elasticloadbalancing:ModifyLoadBalancerAttributes",
                        "elasticloadbalancing:SetIpAddressType",
                        "elasticloadbalancing:SetSecurityGroups",
                        "elasticloadbalancing:SetSubnets",
                        "elasticloadbalancing:DeleteLoadBalancer",
                        "elasticloadbalancing:ModifyTargetGroup",
                        "elasticloadbalancing:ModifyTargetGroupAttributes",
                        "elasticloadbalancing:DeleteTargetGroup"
                    ],
                    "Resource": "*",
                    "Condition": {
                        "Null": {
                            "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                        }
                    }
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "elasticloadbalancing:RegisterTargets",
                        "elasticloadbalancing:DeregisterTargets"
                    ],
                    "Resource": "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "elasticloadbalancing:SetWebAcl",
                        "elasticloadbalancing:ModifyListener",
                        "elasticloadbalancing:AddListenerCertificates",
                        "elasticloadbalancing:RemoveListenerCertificates",
                        "elasticloadbalancing:ModifyRule"
                    ],
                    "Resource": "*"
                }
            ]
        }

  - boto3 사용을 위한 파드 생성

    - ```bash
      cat <<EOF | kubectl create -f -
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: boto3-pod
      spec:
        replicas: 2
        selector:
          matchLabels:
            app: boto3
        template:
          metadata:
            labels:
              app: boto3
          spec:
            containers:
            - name: boto3
              image: jpbarto/boto3
              command: ["tail"]
              args: ["-f", "/dev/null"]
            terminationGracePeriodSeconds: 0
      EOF
      ```

      - ![image-20230303010557155](files/img/image-20230303010557155.png)

  - 파드 이름 변수 지정

    - ```bash
      PODNAME1=$(kubectl get pod -l app=boto3 -o jsonpath={.items[0].metadata.name})
      echo $PODNAME1
      PODNAME2=$(kubectl get pod -l app=boto3 -o jsonpath={.items[1].metadata.name})
      echo $PODNAME2
      ```

      - ![image-20230303010645287](files/img/image-20230303010645287.png)

  - 파드1에서 boto3 사용 - ec2 메타정보를 가져오는 보안 비활성화

    - Describe Regions and Availability Zones

      - https://boto3.amazonaws.com/v1/documentation/api/latest/guide/ec2-example-regions-avail-zones.html#describe-regions-and-availability-zones

    - ```bash
      kubectl exec -it $PODNAME1 -- sh
      # ------------
      cat <<EOF> az.py
      import boto3
      
      ec2 = boto3.client('ec2', region_name='us-east-1')
      
      # Retrieves all regions/endpoints that work with EC2
      response = ec2.describe_regions()
      print('Regions:', response['Regions'])
      EOF
      
      python az.py
      exit
      # ------------
      ```

      - ![image-20230303025504353](files/img/image-20230303025504353.png)



## 3. Kubescape

### 3.1 개요

- 참고 링크
  - Github - https://github.com/kubescape/kubescape
  - 서비스 - https://www.armosec.io/kubescape/
  - 활용사례 - https://devocean.sk.com/experts/techBoardDetail.do?ID=164199
  - Docs - https://hub.armosec.io/docs
  - Blog - https://www.armosec.io/blog/

- 소개 및 사용 : 보안 권고 사항 기반 현재 쿠버네티스 클러스터(YAML, Helm chart)의 취약점을 점검
  - Kubescape Component architecture
    - ![image-20230302233725379](files/img/image-20230302233725379.png)
      - 출처: https://github.com/kubescape/kubescape/blob/master/docs/architecture.md
  - Kubescape is an open-source Kubernetes security platform for your **IDE**, **CI/CD pipelines**, and **clusters**. It includes risk analysis, security, compliance, and misconfiguration scanning, saving Kubernetes users and administrators precious time, effort, and resources.
  - Kubescape scans clusters, YAML files, and Helm charts. It detects misconfigurations according to multiple frameworks (including [NSA-CISA](https://www.armosec.io/blog/kubernetes-hardening-guidance-summary-by-armo/?utm_source=github&utm_medium=repository), [MITRE ATT&CK®](https://www.microsoft.com/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/) and the [CIS Benchmark](https://www.armosec.io/blog/cis-kubernetes-benchmark-framework-scanning-tools-comparison/?utm_source=github&utm_medium=repository)).
  - Kubescape was created by [ARMO](https://www.armosec.io/?utm_source=github&utm_medium=repository) and is a Cloud Native Computing Foundation (CNCF) sandbox project.
  - *Did you know you can use Kubescape in all these places?*
    - ![image-20230302233828862](files/img/image-20230302233828862.png)
      - 출처: https://github.com/kubescape/kubescape



### 3.2 실습

- 설치

  - ```bash
    curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | /bin/bash
    ```

    - ![image-20230303025833598](files/img/image-20230303025833598.png)

- Download all artifacts and save them in the default path (~/.kubescape)

  - ```bash
    kubescape download artifacts
    tree ~/.kubescape/
    cat ~/.kubescape/attack-tracks.json | jq
    ```

    - ![image-20230303025900231](files/img/image-20230303025900231.png)
    - ![image-20230303025911791](files/img/image-20230303025911791.png)

- 제공하는 보안 프레임워크 확인

  - ```bash
    kubescape list frameworks --format json | jq '.[]'
    ```

    - ![image-20230303025932359](files/img/image-20230303025932359.png)

- 제공하는 통제 정책 확인

  - ```bash
    kubescape list controls
    ```

    - ![image-20230303025954558](files/img/image-20230303025954558.png)
    - ![image-20230303030000628](files/img/image-20230303030000628.png)

- 모니터링

  - ```bash
    watch kubectl get pod -A
    ```

    - ![image-20230303030029582](files/img/image-20230303030029582.png)

- 클러스터 스캔

  - ```bash
    # Deploy Kubescape host-sensor daemonset in the scanned cluster. Deleting it right after we collecting the data. 
    # Required to collect valuable data from cluster nodes for certain controls. 
    # Yaml file: https://github.com/kubescape/kubescape/blob/master/core/pkg/hostsensorutils/hostsensor.yaml
    kubescape scan --help
    kubescape scan --enable-host-scan --verbose
    ```

    - ![image-20230303030059937](files/img/image-20230303030059937.png)
    - ![image-20230303030411214](files/img/image-20230303030411214.png)
    - ![image-20230303030417818](files/img/image-20230303030417818.png)

- (옵션1) armo 웹 사용 : 책 398~400 참고
- (옵션2) Prometheus Exporter - [링크](https://hub.armosec.io/docs/prometheus-exporter) [dashboard](https://github.com/dwertent/kubescape/blob/master/httphandler/examples/prometheus/grafana-kubescape-dashboard.json) [crontab](https://crontab.guru/)
- (옵션3) VSCODE - [링크](https://hub.armosec.io/docs/visual-studio-code)



### 3.3 과제2

- 목표: 책 398~400페이지 - kubescape armo 웹 사용 후 관련 스샷을 올려주세요

- 수행결과 및 과정

  - 웹 브라우저에서 `portal.armo.cloud` 주소로 접근하여 회원 가입

    - https://cloud.armosec.io/
    - ![image-20230303030830161](files/img/image-20230303030830161.png)

  - 실행스크립트 확인 후 실행

    - ```bash
      helm repo add kubescape https://kubescape.github.io/helm-charts/
      helm repo update
      helm upgrade --install kubescape kubescape/kubescape-cloud-operator -n kubescape --create-namespace --set account=2b976a0e-afb8-4cf9-8a15-102c80060ac1 --set clusterName=`kubectl config current-context`
      ```

      - ![image-20230303030923083](files/img/image-20230303030923083.png)

  - `I ran the script` 클릭

    - ![image-20230303031032787](files/img/image-20230303031032787.png)
    - ![image-20230303031115916](files/img/image-20230303031115916.png)
    - ![image-20230303031127431](files/img/image-20230303031127431.png)
    - ![image-20230303031246067](files/img/image-20230303031246067.png)



## 4. Polaris

### 4.1 개요

- 참고 링크
  - Github - https://github.com/FairwindsOps/polaris
  - 서비스 - https://www.fairwinds.com/polaris
  - 적용검증 - https://waspro.tistory.com/766
  - Docs - https://polaris.docs.fairwinds.com/
  - Blog - https://www.fairwinds.com/blog

- 소개 및 설치 : 오픈소스 보안 점검 도구, Polaris is an open source policy engine for Kubernetes that validates and remediates resource configuration - [helm](https://artifacthub.io/packages/helm/fairwinds-stable/polaris)
  - It includes 30+ built in configuration policies, as well as the ability to build custom policies with JSON Schema.
  - When run on the command line or as a mutating **webhook**, Polaris can automatically remediate issues based on policy criteria.
  - ![image-20230302234234240](files/img/image-20230302234234240.png)
    - 출처: https://github.com/FairwindsOps/polaris
  - As a [dashboard](https://polaris.docs.fairwinds.com/dashboard) - Validate Kubernetes resources against policy-as-code ⇒ **실습 범위**
  - As an [admission controller](https://polaris.docs.fairwinds.com/admission-controller) - Automatically reject or modify workloads that don't adhere to your organization's policies ⇒ **실습 범위**
      - Validating Webhook : 새 워크로드 배포 시 체크 항목 만족 되지 않을 경우, 배포 차단 ⇒ helm 배포 때 webhook 활성화 필요
  - As a [command-line tool](https://polaris.docs.fairwinds.com/infrastructure-as-code) - Incorporate policy-as-code into the CI/CD process to test local YAML files.



### 4.2 실습

#### 4.2.1 설치 및 배포

- 네임스페이스 생성

  - ```bash
    kubectl create ns polaris
    ```

    - ![image-20230303031719070](files/img/image-20230303031719070.png)

- 

  - ```bash
    cat <<EOT > polaris-values.yaml
    dashboard:
      replicas: 1
      service:
        type: LoadBalancer
    EOT
    ```

    - ![image-20230303031729323](files/img/image-20230303031729323.png)

- 배포

  - ```bash
    helm repo add fairwinds-stable https://charts.fairwinds.com/stable
    helm install polaris fairwinds-stable/polaris --namespace polaris --version 5.7.2 -f polaris-values.yaml
    ```

    - ![image-20230303031755803](files/img/image-20230303031755803.png)

- CLB에 ExternanDNS 로 도메인 연결

  - ```bash
    kubectl annotate service polaris-dashboard "external-dns.alpha.kubernetes.io/hostname=polaris.$KOPS_CLUSTER_NAME" -n polaris
    ```

    - ![image-20230303031804127](files/img/image-20230303031804127.png)
    - ![image-20230303032450223](files/img/image-20230303032450223.png)
    - ![image-20230303032501572](files/img/image-20230303032501572.png)

- 웹 접속 주소 확인 및 접속

  - ```bash
    echo -e "Polaris Web URL = http://polaris.$KOPS_CLUSTER_NAME"
    ```

    - ![image-20230303032511590](files/img/image-20230303032511590.png)
    - ![image-20230303032246379](files/img/image-20230303032246379.png)

- **Score** : 모범 사례 대비 전체 클러스터 구성 내역 점수, 권고 사항 만족이 많을 수록 점수 상승

- **Passing/Warning/Dangerous** Checks : 위험 등급 분류, 위험 단계 취약점은 조치를 권고

- 측정 범위

   : Efficiency, Reliability, Security

  - Security 보안성 : 보안 관련 구성 확인 - [링크](https://polaris.docs.fairwinds.com/checks/security/)
  - Efficiency 효율성 : CPU,Memory 리소스 사용 관련 - [링크](https://polaris.docs.fairwinds.com/checks/efficiency/)
  - Reliability 신뢰성 : 안정성 구성 여부 확인 - [링크](https://polaris.docs.fairwinds.com/checks/reliability/)

- 검사 항목 상세 - [링크](https://github.com/FairwindsOps/polaris/tree/master/checks)



#### 4.2.2 보안취약점 점검

- polaris 웹 접속 → 중간 Filter by Namespace 에서 default 네임스페이스 클릭 후 Apply

  - ![image-20230303032556661](files/img/image-20230303032556661.png)

- Namespaces : default 아래 netshoot-pod 부분 ⇒ 아래 항목의 ? 클릭으로 상세 정보 확인

  - ![image-20230303032715590](files/img/image-20230303032715590.png)

- Image tag 조치 - [링크](https://hub.docker.com/r/nicolaka/netshoot/tags)

  - 기존 netshoot-pod 삭제

    - ```bash
      kubectl delete deploy netshoot-pod
      ```

      - ![image-20230303032735384](files/img/image-20230303032735384.png)

  - netshoot-pod 1대 생성

    - ```bash
      cat <<EOF | kubectl create -f -
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: netshoot-pod
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: netshoot-pod
        template:
          metadata:
            labels:
              app: netshoot-pod
          spec:
            containers:
            - name: netshoot-pod
              image: nicolaka/netshoot:v0.9
              command: ["tail"]
              args: ["-f", "/dev/null"]
            terminationGracePeriodSeconds: 0
      EOF
      ```

      - ![image-20230303032748991](files/img/image-20230303032748991.png)

  - 대시보드 새로고침 후 확인 : 해결됬지만, 상단에 replica 인데 1개라 Warning 확인
    - 

  - deployment 의 replica 조치 후 확인

    - ```bash
      kubectl scale deployment netshoot-pod --replicas 2
      ```

      - ![image-20230303032804450](files/img/image-20230303032804450.png)

  - 대시보드 새로고침 후 확인 : 해결!

    - ![image-20230303032949516](files/img/image-20230303032949516.png)



#### 4.2.3 보안 모범 사례 적용 후 재점검

- 폴라리스 파드의 설정 참고

  - 

    - ```bash
      kubectl krew install neat
      kubectl get deploy/polaris-dashboard -n polaris -o yaml | kubectl neat > polaris-pod.yaml
      ```

      - ![image-20230303033030464](files/img/image-20230303033030464.png)

  - 

    - ```bash
      cat polaris-pod.yaml | yh
      
      # ...
      # spec:
      #   template:
      #     spec:
      #       containers:
      #         imagePullPolicy: Always   # 이미지를 항상 리포지토리에서 가져오게 설정
      #         resources:                # 리소스 자원 사용 보장 및 제한  
      #           limits:
      #             cpu: 150m
      #             memory: 512Mi
      #           requests:
      #             cpu: 100m
      #             memory: 128Mi
      #         securityContext:          # 컨테이너, 파드의 권한 및 접근 제어
      #           allowPrivilegeEscalation: false   # 컨테이너의 하위 프로세스가 상위 프로세스보다 많은 권한을 얻을 수 없게 설정
      #           capabilities:                     # 호스트 커널 레벨 수정이 필요 시 root 권한으로 전체 커널을 수정하지 않고 특정 커널 권한만 부여 후 사용
      #             drop:
      #             - ALL
      #           privileged: false                 # true인 경우 호스트의 모든 장치에 접근 권한 가짐, 컨테이너의 root권한이더라도 namespace/cgroup으로 격리되어 호스트의 다른 장치 접근 불가
      #           readOnlyRootFilesystem: true      # 컨테이너의 root 파일시스템에 읽기 전용
      #           runAsNonRoot: true                # root 권한으로 컨테이너를 실행하지 않음
      # ...
      ```

      - ![image-20230303033101096](files/img/image-20230303033101096.png)
      - ![image-20230303033127144](files/img/image-20230303033127144.png)

- netshoot-pod 에 보안 모범 사례 적용

  - 삭제

    - ```bash
      kubectl delete deploy netshoot-pod
      ```

      - ![image-20230303033147297](files/img/image-20230303033147297.png)

  - netshoot-pod 생성

    - ```bash
      cat <<EOF | kubectl create -f -
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: netshoot-pod
      spec:
        replicas: 2
        selector:
          matchLabels:
            app: netshoot-pod
        template:
          metadata:
            labels:
              app: netshoot-pod
          spec:
            containers:
            - name: netshoot-pod
              image: nicolaka/netshoot:v0.9
              command: ["tail"]
              args: ["-f", "/dev/null"]
              imagePullPolicy: Always
              resources:
                limits:
                  cpu: 150m
                  memory: 512Mi
                requests:
                  cpu: 100m
                  memory: 128Mi
              securityContext:
                allowPrivilegeEscalation: false
                capabilities:
                  drop:
                  - ALL
                privileged: false
                readOnlyRootFilesystem: true
                #runAsNonRoot: true
            terminationGracePeriodSeconds: 0
      EOF
      ```

      - ![image-20230303033202336](files/img/image-20230303033202336.png)

  - 폴라리스 웹 확인! → 더 필요한 부분은 조치를 해보자

    - ![image-20230303033248120](files/img/image-20230303033248120.png)

  - 다음 실습을 위해 삭제

    - ```bash
      kubectl delete deploy netshoot-pod
      ```

      - ![image-20230303033305974](files/img/image-20230303033305974.png)



#### 4.2.4 webhook 활성화 후 테스트 \- [링크](https://polaris.docs.fairwinds.com/admission-controller/)

- webhook 활성화 적용

  - ```bash
    helm upgrade polaris fairwinds-stable/polaris --namespace polaris --version 5.7.2 --reuse-values --set webhook.enable=true
    kubectl get pod,svc -n polaris
    ```

    - ![image-20230303033335126](files/img/image-20230303033335126.png)

- 취약한 파드 배포 시도

  - ```bash
    # netshoot-pod 1대 생성
    cat <<EOF | kubectl create -f -
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: netshoot-pod
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: netshoot-pod
      template:
        metadata:
          labels:
            app: netshoot-pod
        spec:
          containers:
          - name: netshoot-pod
            image: nicolaka/netshoot
            command: ["tail"]
            args: ["-f", "/dev/null"]
          terminationGracePeriodSeconds: 0
    EOF
    ```

    - ![image-20230303033410542](files/img/image-20230303033410542.png)



#### 4.2.5 IaC 활용 cli - [링크](https://polaris.docs.fairwinds.com/infrastructure-as-code/)

- 다운로드

  - ```bash
    wget https://github.com/FairwindsOps/polaris/releases/download/7.3.2/polaris_linux_amd64.tar.gz
    tar zxvf polaris_linux_amd64.tar.gz
    mv polaris /usr/local/bin
    polaris audit --help
    ```

    - ![image-20230303033423846](files/img/image-20230303033423846.png)
    - ![image-20230303033444648](files/img/image-20230303033444648.png)

- 마리오 파드 yaml 점검

  - ```bash
    polaris audit --audit-path ~/pkos/1/mario.yaml --format=pretty
    ```

    - ![image-20230303033458604](files/img/image-20230303033458604.png)
    - ![image-20230303033503652](files/img/image-20230303033503652.png)



#### 4.2.6 기타

- (참고) Github Action 에  polaris actoin 수행 - [링크](https://polaris.docs.fairwinds.com/infrastructure-as-code/#running-in-a-ci-pipeline)

- 다음 실습을 위해 polaris 삭제:  

  - ```bash
    helm uninstall -n polaris polaris
    ```

    - ![image-20230303035030127](files/img/image-20230303035030127.png)

- 컨테이너 보안 컨텍스트 : 정리 예정 - 책 p590 - [링크](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-pod)

- 파드 보안 컨텍스트 : 정리 예정 - 책 p592 - [링크](https://freedeveloper.tistory.com/427) [링크2](https://www.inflearn.com/questions/74754/securitycontext-관련)



### 4.3 과제3

- 목표: polaris 관련 실습(아무거나) 후 관련 스샷을 올려주세요

- 수행결과 및 과정

  - 문제 확인

    - ![image-20230303034130356](files/img/image-20230303034130356.png)
      - https://polaris.docs.fairwinds.com/checks/security/
    - ![image-20230303034339494](files/img/image-20230303034339494.png)

  - 문제 해결을 위한 netshoot-pod 생성

    - ```bash
      cat <<EOF | kubectl create -f -
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: netshoot-pod
      spec:
        replicas: 2
        selector:
          matchLabels:
            app: netshoot-pod
        template:
          metadata:
            labels:
              app: netshoot-pod
          spec:
            containers:
            - name: netshoot-pod
              image: nicolaka/netshoot:v0.9
              command: ["tail"]
              args: ["-f", "/dev/null"]
              imagePullPolicy: Always
              resources:
                limits:
                  cpu: 150m
                  memory: 512Mi
                requests:
                  cpu: 100m
                  memory: 128Mi
              securityContext:
                allowPrivilegeEscalation: false
                capabilities:
                  drop:
                  - ALL
                privileged: false
                readOnlyRootFilesystem: true
                runAsNonRoot: true # root 권한으로 컨테이너를 실행하지 않음
            terminationGracePeriodSeconds: 0
      EOF
      ```

      - ![image-20230303034417047](files/img/image-20230303034417047.png)

    - 폴라리스 웹 확인

      - ![image-20230303034436048](files/img/image-20230303034436048.png)

    - 다음 실습을 위해 삭제

      - ```bash
        kubectl delete deploy netshoot-pod
        ```

        - ![image-20230303034449375](files/img/image-20230303034449375.png)





## 5. K8S 인증/인가 & RBAC

### 5.1 개요

- K8S(API 접근) 인증/인가 소개 : 출처 - 김태민 기술 블로그 - [링크](https://kubetm.github.io/k8s/07-intermediate-basic-resource/authentication/) [링크2](https://kubetm.github.io/k8s/07-intermediate-basic-resource/authorization/)
  - **서비스 어카운트(**Service Account)
  - **API 서버 사용** : kubectl(config, 다수 클러스터 관리 가능), 서비스 어카운트, https(x.509 Client Certs) ⇒ `X.509 발음`을 어떻게 하시나요? - [링크](https://youglish.com/pronounce/x.509/english)
  - **API 서버 접근 과정** : **인증** → **인가** → **Admission Control**(API 요청 검증, 필요 시 변형 - 예. ResourceQuota, LimitRange) - [참고](https://blog.naver.com/alice_k106/221546328906)
    - ![image-20230302235801399](files/img/image-20230302235801399.png)
      - 출처: PKOS 스터디 7주차(CloudNet@ Blog)

- 인증(Authentication)
  - **X.509 Client Certs** : kubeconfig 에 **CA crt**(발급 기관 인증서) , **Client crt**(클라이언트 인증서) , **Client key**(클라이언트 개인키) 를 통해 인증
  - **kubectl** : 여러 클러스터(**kubeconfig**)를 관리 가능 - **contexts** 에 클러스터와 유저 및 **인증서**/**키** 참고
  - **Service Account** : 기본 서비스 어카운트(default) - 시크릿(CA crt 와 token)
  - ![image-20230302235949503](files/img/image-20230302235949503.png)
    - 출처: PKOS 스터디 7주차(CloudNet@ Blog)

- 인가(Authorization)
  - **인가 방식** : **RBAC(Role, RoleBinding)**, ABAC, Webhook, Node Authorization⇒ `RBAC 발음`을 어떻게 하시나요?
  - RBAC : `역할` 기반의 권한 관리, `사용자`와 역할을 별개로 선언 후 두가지를 `조합`(binding)해서 사용자에게 권한을 부여하여 kubectl or API로 관리 가능
    - Namespace/Cluster - Role/ClusterRole, RoleBinding/ClusterRoleBinding, Service Account
    - Role(롤) - (RoleBinding 롤 바인딩) - Service Account(서비스 어카운트) : 롤 바인딩은 롤과 서비스 어카운트를 연결
    - Role(네임스페이스내 자원의 권한) vs ClusterRole(클러스터 수준의 자원의 권한)
    - ![image-20230303000046706](files/img/image-20230303000046706.png)
      - 출처: PKOS 스터디 7주차(CloudNet@ Blog)

- .kube/config 파일 내용

  - `clusters` : kubectl 이 사용할 쿠버네티스 `API 서버의 접속 정보 목록`. 원격의 쿠버네티스 API 서버의 주소를 추가해 사용 가능

  - `users` : 쿠버네티스의 API 서버에 접속하기 위한 `사용자 인증 정보 목록`. (서비스 어카운트의 토큰, 혹은 인증서의 데이터 등)

  - `contexts` : cluster 항목과 users 항목에 정의된 값을 조합해 최종적으로 사용할 쿠버네티스 클러스터의 정보(컨텍스트)를 설정.

    - 예를 들어 clusters 항목에 클러스터 A,B 가 정의돼 있고, users 항목에 사용자 a,b 가 정의돼 있다면 `cluster A + user a 를 조합해, 'cluster A 에 user a 로 인증해 쿠버네티스를 사용한다`' 라는 새로운 컨텍스트를 정의할 수 있습니다.
    - kubectl 을 사용하려면 여러 개의 컨텍스트 중 하나를 선택.

  - ```bash
    cat .kube/config
    ```

    - ![image-20230303031451036](files/img/image-20230303031451036.png)

### 5.2 실습

#### 5.2.1 실습환경

- 쿠버네티스에 사용자를 위한 서비스 어카운트(Service Account, SA)를 생성 : dev-k8s, infra-k8s
- 사용자는 각기 다른 권한(Role, 인가)을 가짐 : dev-k8s(dev-team 네임스페이스 내 모든 동작) , infra-k8s(dev-team 네임스페이스 내 모든 동작)
- 각각 별도의 kubectl 파드를 생성하고, 해당 파드에 SA 를 지정하여 권한에 대한 테스트를 진행
- EKS 대신 저희는 kOps 를 사용합니다!
  - ![image-20230303000407163](files/img/image-20230303000407163.png)
    - 출처: PKOS 스터디 7주차(CloudNet@ Blog)



#### 5.2.2 네임스페이스와 서비스 어카운트 생성 후 확인

- 파드 기동 시 서비스 어카운트 한 개가 할당되며, 서비스 어카운트 기반 인증/인가를 함, 미지정 시 기본 서비스 어카운트가 할당
- 서비스 어카운트에 자동 생성된 시크릿에 저장된 토큰으로 쿠버네티스 API에 대한 인증 정보로 사용 할 수 있다 ← 1.23 이전 버전의 경우에만 해당

- 네임스페이스(Namespace, NS) 생성 및 확인

  - ```bash
    kubectl create namespace dev-team
    kubectl create ns infra-team
    ```

    - ![image-20230303031525291](files/img/image-20230303031525291.png)

- 네임스페이스 확인

  - ```bash
    kubectl get ns
    ```

    - ![image-20230303031539216](files/img/image-20230303031539216.png)

- 네임스페이스에 각각 서비스 어카운트 생성 : serviceaccounts 약자(=sa)

  - ```bash
    kubectl create sa dev-k8s -n dev-team
    kubectl create sa infra-k8s -n infra-team
    ```

    - ![image-20230303031554289](files/img/image-20230303031554289.png)

- 서비스 어카운트 정보 확인

  - ```bash
    kubectl get sa -n dev-team
    kubectl get sa dev-k8s -n dev-team -o yaml | yh
    ```

    - ![image-20230303031606539](files/img/image-20230303031606539.png)

  - ```bash
    kubectl get sa -n infra-team
    kubectl get sa infra-k8s -n infra-team -o yaml | yh
    ```

    - ![image-20230303031619212](files/img/image-20230303031619212.png)



#### 5.2.3 (심화 참고) dev-k8s 서비스어카운트의 토큰 정보 확인  - https://jwt.io/ → Bearer type - JWT(JSON Web Token) - [링크](https://coffeewhale.com/kubernetes/authentication/http-auth/2020/05/03/auth02/)

- JSON Web Token (jwt)
  - ![image-20230303001143871](files/img/image-20230303001143871.png)
    - 출처: https://coffeewhale.com/kubernetes/authentication/http-auth/2020/05/03/auth02/



- dev-k8s 서비스어카운트의 토큰 정보 확인 

  - ```bash
    DevTokenName=$(kubectl get sa dev-k8s -n dev-team -o jsonpath="{.secrets[0].name}")
    DevToken=$(kubectl get secret -n dev-team $DevTokenName -o jsonpath="{.data.token}" | base64 -d)
    echo $DevToken
    ```

    - ![image-20230303034650539](files/img/image-20230303034650539.png)

- **Bearer type - JWT(JSON Web Token)**

  - Bearer type 경우, 서버에서 지정한 어떠한 문자열도 입력할 수 있습니다. 하지만 굉장히 허술한 느낌을 받습니다.

  - 이를 보완하고자 쿠버네티스에서 Bearer 토큰을 전송할 때 주로 **JWT** (JSON Web Token) 토큰을 사용합니다.

  - **JWT**는 X.509 Certificate와 마찬가지로 private key를 이용하여 토큰을 서명하고 public key를 이용하여 서명된 메세지를 검증합니다.

  - 이러한 메커니즘을 통해 해당 토큰이 쿠버네티스를 통해 생성된 valid한 토큰임을 인증할 수 있습니다.

  - `X.509 Certificate`의 `lightweight JSON` 버전이라고 생각하면 편리합니다.
    - jwt는 JSON 형태로 토큰 형식을 정의한 스펙입니다. jwt는 쿠버네티스에서 뿐만 아니라 다양한 웹 사이트에서 인증, 권한 허가, 세션관리 등의 목적으로 사용합니다.
      - Header: 토큰 형식와 암호화 알고리즘을 선언합니다.
      - Payload: 전송하려는 데이터를 JSON 형식으로 기입합니다.
      - Signature: Header와 Payload의 변조 가능성을 검증합니다.

  - 각 파트는 base64 URL 인코딩이 되어서 `.`으로 합쳐지게 됩니다.



#### 5.2.4 서비스 어카운트를 지정하여 파드 생성 후 권한 테스트

- ![image-20230303001709090](files/img/image-20230303001709090.png)
  - 출처: https://kubetm.github.io/k8s/07-intermediate-basic-resource/authentication/#3-service-account

- 각각 네임스피이스에 kubectl 파드 생성 - [컨테이너이미지](https://hub.docker.com/r/bitnami/kubectl/)

  - ```bash
    # docker run --rm --name kubectl -v /path/to/your/kube/config:/.kube/config bitnami/kubectl:latest
    cat <<EOF | kubectl create -f -
    apiVersion: v1
    kind: Pod
    metadata:
      name: dev-kubectl
      namespace: dev-team
    spec:
      serviceAccountName: dev-k8s
      containers:
      - name: kubectl-pod
        image: bitnami/kubectl:1.24.10
        command: ["tail"]
        args: ["-f", "/dev/null"]
      terminationGracePeriodSeconds: 0
    EOF
    ```

    - ![image-20230303035133483](files/img/image-20230303035133483.png)

  - ```bash
    cat <<EOF | kubectl create -f -
    apiVersion: v1
    kind: Pod
    metadata:
      name: infra-kubectl
      namespace: infra-team
    spec:
      serviceAccountName: infra-k8s
      containers:
      - name: kubectl-pod
        image: bitnami/kubectl:1.24.10
        command: ["tail"]
        args: ["-f", "/dev/null"]
      terminationGracePeriodSeconds: 0
    EOF
    ```

    - ![image-20230303035141974](files/img/image-20230303035141974.png)

- 확인

  - ```bash
    kubectl get pod -A
    kubectl get pod -o dev-kubectl -n dev-team -o yaml
    ```

    - ![image-20230303035221560](files/img/image-20230303035221560.png)
    - ![image-20230303035243880](files/img/image-20230303035243880.png)
    - ![image-20230303035300726](files/img/image-20230303035300726.png)

  - ```bash
    kubectl get pod -o infra-kubectl -n infra-team -o yaml
    ```

    - ![image-20230303035315907](files/img/image-20230303035315907.png)
    - ![image-20230303035321411](files/img/image-20230303035321411.png)

- 파드에 기본 적용되는 서비스 어카운트(토큰) 정보 확인

  - ```bash
    kubectl exec -it dev-kubectl -n dev-team -- ls /run/secrets/kubernetes.io/serviceaccount
    kubectl exec -it dev-kubectl -n dev-team -- cat /run/secrets/kubernetes.io/serviceaccount/token
    kubectl exec -it dev-kubectl -n dev-team -- cat /run/secrets/kubernetes.io/serviceaccount/namespace
    kubectl exec -it dev-kubectl -n dev-team -- cat /run/secrets/kubernetes.io/serviceaccount/ca.crt
    ```

    - ![image-20230303035409171](files/img/image-20230303035409171.png)

- 각각 파드로 Shell 접속하여 정보 확인 : 단축 명령어(alias) 사용

  - ```bash
    alias k1='kubectl exec -it dev-kubectl -n dev-team -- kubectl'
    alias k2='kubectl exec -it infra-kubectl -n infra-team -- kubectl'
    ```

    - ![image-20230303035432717](files/img/image-20230303035432717.png)

- 권한 테스트

  - ```bash
    k1 get pods # kubectl exec -it dev-kubectl -n dev-team -- kubectl get pods 와 동일한 실행 명령이다!
    k1 run nginx --image nginx:1.20-alpine
    k1 get pods -n kube-system
    ```

    - ![image-20230303035507391](files/img/image-20230303035507391.png)
    - ![image-20230303035450883](files/img/image-20230303035450883.png)
    - ![image-20230303035525015](files/img/image-20230303035525015.png)
    - 

  - ```bash
    k2 get pods # kubectl exec -it infra-kubectl -n infra-team -- kubectl get pods 와 동일한 실행 명령이다!
    k2 run nginx --image nginx:1.20-alpine
    k2 get pods -n kube-system
    ```

    - ![image-20230303035545856](files/img/image-20230303035545856.png)
    - ![image-20230303035634677](files/img/image-20230303035634677.png)
    - ![image-20230303035729988](files/img/image-20230303035729988.png)

- (옵션) kubectl auth can-i 로 kubectl 실행 사용자가 특정 권한을 가졌는지 확인

  - ```bash
    k1 auth can-i get pods
    ```

    - ![image-20230303035738640](files/img/image-20230303035738640.png)



#### 5.2.5 각각 네임스페이스에 롤(Role)를 생성 후 서비스 어카운트 바인딩

- **롤(Role)** : apiGroups 와 resources 로 지정된 리소스에 대해 **verbs** 권한을 인가

- **실행 가능한 조작(verbs)** : *(모두 처리), create(생성), delete(삭제), get(조회), list(목록조회), patch(일부업데이트), update(업데이트), watch(변경감시)

  - ![image-20230303002228308](files/img/image-20230303002228308.png)
    - 출처: https://kubetm.github.io/k8s/07-intermediate-basic-resource/authorization/#1-%EC%9E%90%EC%8B%A0%EC%9D%98-namespace-%EB%82%B4%EC%97%90-pod%EB%93%A4%EB%A7%8C-%EC%A1%B0%ED%9A%8C%ED%95%A0-%EC%88%98-%EC%9E%88%EB%8A%94-%EA%B6%8C%ED%95%9C

- 각각 네임스페이스내의 모든 권한에 대한 롤 생성

  - ```bash
    cat <<EOF | kubectl create -f -
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: role-dev-team
      namespace: dev-team
    rules:
    - apiGroups: ["*"]
      resources: ["*"]
      verbs: ["*"]
    EOF
    ```

    - ![image-20230303035807959](files/img/image-20230303035807959.png)

  - ```bash
    cat <<EOF | kubectl create -f -
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: role-infra-team
      namespace: infra-team
    rules:
    - apiGroups: ["*"]
      resources: ["*"]
      verbs: ["*"]
    EOF
    ```

    - ![image-20230303035816060](files/img/image-20230303035816060.png)

- 롤 확인

  - ```bash
    kubectl get roles -n dev-team
    kubectl get roles -n infra-team
    kubectl get roles -n dev-team -o yaml
    ```

    - ![image-20230303035835572](files/img/image-20230303035835572.png)

  - ```bash
    kubectl describe roles role-dev-team -n dev-team
    ```

    - ![image-20230303035845978](files/img/image-20230303035845978.png)

- 롤바인딩 생성 : '서비스어카운트 <-> 롤' 간 서로 연동

  - ```bash
    cat <<EOF | kubectl create -f -
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: roleB-dev-team
      namespace: dev-team
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: role-dev-team
    subjects:
    - kind: ServiceAccount
      name: dev-k8s
      namespace: dev-team
    EOF
    ```

    - ![image-20230303035858031](files/img/image-20230303035858031.png)

  - ```bash
    cat <<EOF | kubectl create -f -
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: roleB-infra-team
      namespace: infra-team
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: Role
      name: role-infra-team
    subjects:
    - kind: ServiceAccount
      name: infra-k8s
      namespace: infra-team
    EOF
    ```

    - ![image-20230303035906680](files/img/image-20230303035906680.png)

- 롤바인딩 확인

  - ```bash
    kubectl get rolebindings -n dev-team
    kubectl get rolebindings -n infra-team
    kubectl get rolebindings -n dev-team -o yaml
    ```

    - ![image-20230303035925824](files/img/image-20230303035925824.png)

  - ```bash
    kubectl describe rolebindings roleB-dev-team -n dev-team
    ```

    - ![image-20230303035935166](files/img/image-20230303035935166.png)



#### 5.2.6 서비스 어카운트를 지정하여 생성한 파드에서 다시 권한 테스트

- 각각 파드로 Shell 접속하여 정보 확인 : 단축 명령어(alias) 사용

  - ```bash
    alias k1='kubectl exec -it dev-kubectl -n dev-team -- kubectl'
    alias k2='kubectl exec -it infra-kubectl -n infra-team -- kubectl'
    ```

    - ![image-20230303035947130](files/img/image-20230303035947130.png)

- 권한 테스트

  - ```bash
    k1 get pods
    k1 run nginx --image nginx:1.20-alpine
    k1 get pods
    k1 delete pods nginx
    k1 get pods -n kube-system
    k1 get nodes
    ```

    - ![image-20230303040032480](files/img/image-20230303040032480.png)

  - ```bash
    k2 get pods
    k2 run nginx --image nginx:1.20-alpine
    k2 get pods
    k2 delete pods nginx
    k2 get pods -n kube-system
    k2 get nodes
    ```

    - ![image-20230303040106389](files/img/image-20230303040106389.png)

- (옵션) kubectl auth can-i 로 kubectl 실행 사용자가 특정 권한을 가졌는지 확인

  - ```bash
    k1 auth can-i get pods
    ```

    - ![image-20230303040116337](files/img/image-20230303040116337.png)

#### 5.2.7 리소스 삭제

- ```bash
  kubectl delete ns dev-team infra-team
  ```

  - ![image-20230303040129659](files/img/image-20230303040129659.png)



### 5.3 과제4

- 목표: 신규 서비스 어카운트(SA) 생성 후 '클러스터 수준(모든 네임스페이스 포함)에서 읽기 전용'의 권한을 주고 테스트 후 코드나 스샷을 올려주세요

- 수행결과 및 과정

  - 본 과제는 박정희(카이도스)님의 과제 수행 과정을 참조하여 진행했습니다.
    - https://djdakf1234.tistory.com/99

  - 네임스페이스(Namespace, NS) 생성 및 확인

    - ```bash
      kubectl create ns infra-team
      k get ns
      ```

      - ![image-20230303040847300](files/img/image-20230303040847300.png)

  - 신규 어카운트 생성

    - ```bash
      kubectl create sa infra-k8s-cluster -n infra-team
      ```

      - ![image-20230303041126658](files/img/image-20230303041126658.png)

  - 신규 POD 생성

    - ```bash
      cat <<EOF | kubectl create -f -
      apiVersion: v1
      kind: Pod
      metadata:
        name: infra-kubectl-cluster
        namespace: infra-team
      spec:
        serviceAccountName: infra-k8s-cluster
        containers:
        - name: kubectl-pod
          image: bitnami/kubectl:1.24.10
          command: ["tail"]
          args: ["-f", "/dev/null"]
        terminationGracePeriodSeconds: 0
      EOF
      ```

      - ![image-20230303041138541](files/img/image-20230303041138541.png)

  - cluster 롤 생성

    - ```bash
      cat <<EOF | kubectl create -f -
      apiVersion: rbac.authorization.k8s.io/v1
      kind: ClusterRole
      metadata:
        name: role-infra-team-cluster
        namespace: infra-team
      rules:
      - apiGroups: [""]
        resources: ["*"]
        verbs: ["get", "watch", "list"]
      EOF
      ```

      - ![image-20230303041148033](files/img/image-20230303041148033.png)

  - cluster 롤 확인

    - ```bash
      kubectl describe clusterRole role-infra-team-cluster -n infra-team
      ```

      - ![image-20230303041155927](files/img/image-20230303041155927.png)

  - 롤바인딩 생성 : '서비스어카운트 <-> 롤' 간 서로 연동

    - ```bash
      cat <<EOF | kubectl create -f -
      apiVersion: rbac.authorization.k8s.io/v1
      kind: ClusterRoleBinding
      metadata:
        name: roleB-infra-team-cluster
        namespace: infra-team
      roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: ClusterRole
        name: role-infra-team-cluster
      subjects:
      - kind: ServiceAccount
        name: infra-k8s-cluster
        namespace: infra-team
      EOF
      ```

      - ![image-20230303041205593](files/img/image-20230303041205593.png)

  - 확인

    - ```bash
      kubectl describe clusterrolebindings roleB-infra-team-cluster -n infra-team
      ```

      - ![image-20230303041217745](files/img/image-20230303041217745.png)

  - 파드로 Shell 접속하여 정보 확인 : 단축 명령어(alias) 사용

    - ```bash
      alias k1='kubectl exec -it infra-kubectl-cluster -n infra-team -- kubectl'
      ```

      - ![image-20230303041229720](files/img/image-20230303041229720.png)

  - 권한 테스트

    - ```bash
      k1 get pods
      k1 run nginx --image nginx:1.20-alpine
      k1 get nodes
      k1 get ns
      k1 create ns test
      ```

      - ![image-20230303041302081](files/img/image-20230303041302081.png)

  - (옵션) kubectl auth can-i 로 kubectl 실행 사용자가 특정 권한을 가졌는지 확인

    - ```bash
      k1 auth can-i get pods
      ```

      - ![image-20230303041313957](files/img/image-20230303041313957.png)



## 6. (실습 완료 후) 자원 삭제

- kOps 클러스터 삭제 & AWS CloudFormation 스택 삭제

  - ```bash
    kops delete cluster --yes && aws cloudformation delete-stack --stack-name mykops
    ```

    - ![image-20230303041453888](files/img/image-20230303041453888.png)
    - ![image-20230303041906689](files/img/image-20230303041906689.png)
