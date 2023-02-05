---
date: 2023-01-22T20:51:44+09:00
lastmod: 2023-01-25T03:20:44+09:00
author: "MinHo Jung"
authorLink: ""
license: "CC BY-NC-ND 4.0"
draft: false
toc: true
hiddenFromHomePage: false
hiddenFromSearch: false
linkToMarkdown: false
rssFullText: false

title: "PKOS 2주차 - 쿠버네티스 네트워크"
subtitle: "CloudFormation을 이용한 kOps 배포 및 네트워크 흐름과 NLB 실습"
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

왜 그런지 모르겠는데 이미지 위아래로 여백이 생깁니다.

에디터에서는 안그러는데 왜 그런지 좀 찾아봐야겠습니다.

본 글은 초안이기에 다듬는 과정에서 내용이 수정될 수 있습니다. 

---

# PKOS 2주차

## 들어가기전에

본 내용은 `CloudNet@` 팀에서 진행하는 `쿠버네티스 실무 실습` 스터디를 기반으로 작성된 내용입니다.

또한 개념 설명에서 사용된 이미지의 출처는 스터디 학습 자료에서 가져온 것을 밝힙니다.

- 참조: https://www.notion.so/AWS-EKS-VPC-CNI-1-POD-f89e3e5967b24f8c9aa5bfaab1a82ceb

2주차에는 CloudFormation을 이용해 kOps를 설치하고 네트워크 흐름과 NLB 실습을 진행했습니다.



### 과제 수행결과

#### 과제 1 - 파드간 통신 시 tcpdump 내용 확인

![image-20230122231413539](files/img/image-20230122231413539.png)



#### 과제 2 - 워커 노드 1대에 100대의 파드가 배포되게 설정

해결이 잘 되지 않아서 인스턴스 사양을 c5.4xlarge로 늘려 해결했습니다.

![image-20230123034245862](files/img/image-20230123034245862.png)



#### 과제 3 - 서비스(NLB)/파드 배포 시 ExternalDNS 설정해서, 각자 자신의 도메인으로 NLB를 통해 애플리케이션(파드)로 접속

![image-20230123003420678](files/img/image-20230123003420678.png)



![image-20230123003550545](files/img/image-20230123003550545.png)





#### 과제 4 - NLB에 TLS 적용하기

![image-20230123010508839](files/img/image-20230123010508839.png)



![image-20230123010308060](files/img/image-20230123010308060.png)

## 1. 실습 환경 세팅

### 1.1 구성 환경

- 사전 준비
  - AWS 계정, SSH 키 페어, IAM 계정 생성 후 키, S3 버킷
- 전체 구성도
  - 기본 구성 환경은 1주차 내용과 동일
- 실습 환경 내용
  - CloudFormation 스택 실행 시 파라미터를 기입하면, 해당 정보가 반영되어 배포됩니다.
  - VPC는 kOps 배포를 위한 EC2가 위치할 MyVPC 1개와 실제 kOps 가 배포되어 구동되는 VPC 1개로 총 2개가 생성됩니다.
  - CloudFormation 에 EC2의 UserData 부분(Script 실행)으로 AWS kOps 설치를 진행합니다
  - 마스터 노드 1대, 워커 노드는 기본은 2대로 구성됩니다



### 1.2 실습 환경

- 본 실습은 **미국 동부(버지니아 북부) `us-east-1`** 에서 진행됩니다.

#### 1.2.1 S3 버킷 생성

1. S3 URL 접속
   - https://s3.console.aws.amazon.com/s3/buckets?region=us-east-1
2. 버킷 만들기 버튼 클릭
   - ![image-20230124190808258](files/img/image-20230124190808258.png)
3. 버킷 만들기
   - 버킷 이름 설정 후 기본 설정 그대로 버킷 만들기 버튼 클릭
     - 버킷 이름 : 20230124-learn-s3-mybucket
   - ![image-20230124191042704](files/img/image-20230124191042704.png)



#### 1.2.2 Cloud Formation을 이용한 kOps 생성(이하 배포)

1. Cloud Formation URL 접속 및 스택 생성 버튼 클릭

   - https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks
   - ![image-20230124191453040](files/img/image-20230124191453040.png)

2. 스택 생성 - 1단계 스택 생성

   - 템플릿 소스 URL 입력 - Amazon S3 URL
     - https://s3.ap-northeast-2.amazonaws.com/cloudformation.cloudneta.net/K8S/kops-oneclick.yaml
   - ![image-20230124191725189](files/img/image-20230124191725189.png)

3. 스택 생성 - 2단계 스택 세부 정보 지정

   - 스택 이름

     - 스택 이름 : mkops
     - ![image-20230124191952730](files/img/image-20230124191952730.png)

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
       - ![image-20230124193414257](files/img/image-20230124193414257.png)

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
             - 20230124-learn-s3-mybucket

       - ![image-20230124204019684](files/img/image-20230124204019684.png)

     - <<<<< Region AZ >>>>>

       - 설정 설명
         - TargetRegion : kOps를 배포할 리전
         - AvailabliltyZone1 : kOps를 배포할 리전의 가용 영역
         - AvailabliltyZone2 : kOps를 배포할 리전의 가용 영역
       - 설정 내용
         - TargetRegion : us-east-1
         - AvailabliltyZone1 : us-east-1a
         - AvailabliltyZone2 : us-east-1c
       - ![image-20230124193526968](files/img/image-20230124193526968.png)

   - 다음 버튼 클릭

     - ![image-20230124193718432](files/img/image-20230124193718432.png)

4. 스택 생성 - 3단계 스택 옵션

   - 기본 설정 그대로 다음 버튼 클릭
     - ![image-20230124193718432](files/img/image-20230124193718432.png)

5. 스택 생성 - 4단계 mkops 검토

   - 기본 설정 그대로 전송 버튼 클릭
     - ![image-20230124194153632](files/img/image-20230124194153632.png)

6. 스택 생성 확인 및 접속 IP 확인

   - 스택 생성 확인
     - ![image-20230124194429711](files/img/image-20230124194429711.png)
   - 접속 IP 확인
     - KOPSEC2IP : 3.80.146.2
     - ![image-20230124211311404](files/img/image-20230124211311404.png)



#### 1.2.3 kOps 배포 확인

##### 1. 접속 후 기본 설정

- 마스터노드 SSH 접속 - Putty
  - ![image-20230124213822718](files/img/image-20230124213822718.png)

- kOps 기본 인증 : /etc/profile 에 적용되어 있음

  - ```bash
    tail -n 13 /etc/profile
    ```

    - ![image-20230124213835502](files/img/image-20230124213835502.png)

  - ```bash
    cat ~/.kube/config | yh
    ```

    - ![image-20230124213858503](files/img/image-20230124213858503.png)

- 설치 확인

  - ```bash
    kops validate cluster --wait 10m
    ```

    - ![image-20230124213934323](files/img/image-20230124213934323.png)

- 환경변수 정보 확인

  - ```bash
    export | egrep 'ACCOUNT|AWS|KOPS|KUBERNETES'
    ```

    - ![image-20230124214005912](files/img/image-20230124214005912.png)

  - ```bash
    export | egrep 'ACCOUNT|AWS|KOPS|KUBERNETES' | grep -v SECRET
    ```

    - ![image-20230124214019501](files/img/image-20230124214019501.png)

- default 네임스페이스 적용

  - ```bash
    kubectl ns default
    ```

    - ![image-20230124214032948](files/img/image-20230124214032948.png)



##### 2. kOps 정보 확인

- kOps 클러스터 정보 확인

  - ```bash
    kops get cluster
    ```

    - ![image-20230124214109059](files/img/image-20230124214109059.png)

  - ```bash
    kops get cluster -o yaml
    ```

    - ![image-20230124214153320](files/img/image-20230124214153320.png)

    - ```yaml
      apiVersion: kops.k8s.io/v1alpha2
      kind: Cluster
      metadata:
        creationTimestamp: "2023-01-24T11:44:45Z"
        name: learn-dc.link
      spec:
        api:
          dns: {}
        authorization:
          rbac: {}
        channel: stable
        cloudProvider: aws
        configBase: s3://20230124-learn-s3-mybucket/learn-dc.link
        etcdClusters:
        - cpuRequest: 200m
          etcdMembers:
          - encryptedVolume: true
            instanceGroup: master-us-east-1a
            name: a
          memoryRequest: 100Mi
          name: main
        - cpuRequest: 100m
          etcdMembers:
          - encryptedVolume: true
            instanceGroup: master-us-east-1a
            name: a
          memoryRequest: 100Mi
          name: events
        iam:
          allowContainerRegistry: true
          legacy: false
        kubelet:
          anonymousAuth: false
        kubernetesApiAccess:
        - 0.0.0.0/0
        - ::/0
        kubernetesVersion: 1.24.9
        masterPublicName: api.learn-dc.link
        networkCIDR: 172.30.0.0/16
        networking:
          amazonvpc: {}
        nonMasqueradeCIDR: 100.64.0.0/10
        sshAccess:
        - 0.0.0.0/0
        - ::/0
        subnets:
        - cidr: 172.30.32.0/19
          name: us-east-1a
          type: Public
          zone: us-east-1a
        - cidr: 172.30.64.0/19
          name: us-east-1c
          type: Public
          zone: us-east-1c
        topology:
          dns:
            type: Public
          masters: public
          nodes: public

  - ```bash
    kops get cluster -o yaml | yh
    ```

    - ![image-20230124214215077](files/img/image-20230124214215077.png)

    - ```yaml
      apiVersion: kops.k8s.io/v1alpha2
      kind: Cluster
      metadata:
        creationTimestamp: "2023-01-24T11:44:45Z"
        name: learn-dc.link
      spec:
        api:
          dns: {}
        authorization:
          rbac: {}
        channel: stable
        cloudProvider: aws
        configBase: s3://20230124-learn-s3-mybucket/learn-dc.link
        etcdClusters:
        - cpuRequest: 200m
          etcdMembers:
          - encryptedVolume: true
            instanceGroup: master-us-east-1a
            name: a
          memoryRequest: 100Mi
          name: main
        - cpuRequest: 100m
          etcdMembers:
          - encryptedVolume: true
            instanceGroup: master-us-east-1a
            name: a
          memoryRequest: 100Mi
          name: events
        iam:
          allowContainerRegistry: true
          legacy: false
        kubelet:
          anonymousAuth: false
        kubernetesApiAccess:
        - 0.0.0.0/0
        - : :/0
        kubernetesVersion: 1.24.9
        masterPublicName: api.learn-dc.link
        networkCIDR: 172.30.0.0/16
        networking:
          amazonvpc: {}
        nonMasqueradeCIDR: 100.64.0.0/10
        sshAccess:
        - 0.0.0.0/0
        - : :/0
        subnets:
        - cidr: 172.30.32.0/19
          name: us-east-1a
          type: Public
          zone: us-east-1a
        - cidr: 172.30.64.0/19
          name: us-east-1c
          type: Public
          zone: us-east-1c
        topology:
          dns:
            type: Public
          masters: public
          nodes: public
      ```

- 인스턴스그룹 정보 확인

  - ```bash
    kops get ig
    ```

    - ![image-20230124214352774](files/img/image-20230124214352774.png)

  - ```bash
    kops get ig -o yaml
    ```

    - ![image-20230124214435579](files/img/image-20230124214435579.png)

    - ```yaml
      apiVersion: kops.k8s.io/v1alpha2
      kind: InstanceGroup
      metadata:
        creationTimestamp: "2023-01-24T11:44:45Z"
        labels:
          kops.k8s.io/cluster: learn-dc.link
        name: master-us-east-1a
      spec:
        image: 099720109477/ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20221206
        instanceMetadata:
          httpPutResponseHopLimit: 3
          httpTokens: required
        machineType: t3.medium
        maxSize: 1
        minSize: 1
        role: Master
        subnets:
        - us-east-1a
      
      ---
      
      apiVersion: kops.k8s.io/v1alpha2
      kind: InstanceGroup
      metadata:
        creationTimestamp: "2023-01-24T11:44:45Z"
        labels:
          kops.k8s.io/cluster: learn-dc.link
        name: nodes-us-east-1a
      spec:
        image: 099720109477/ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20221206
        instanceMetadata:
          httpPutResponseHopLimit: 1
          httpTokens: required
        machineType: t3.medium
        maxSize: 1
        minSize: 1
        role: Node
        subnets:
        - us-east-1a
      
      ---
      
      apiVersion: kops.k8s.io/v1alpha2
      kind: InstanceGroup
      metadata:
        creationTimestamp: "2023-01-24T11:44:45Z"
        labels:
          kops.k8s.io/cluster: learn-dc.link
        name: nodes-us-east-1c
      spec:
        image: 099720109477/ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20221206
        instanceMetadata:
          httpPutResponseHopLimit: 1
          httpTokens: required
        machineType: t3.medium
        maxSize: 1
        minSize: 1
        role: Node
        subnets:
        - us-east-1c
      ```

  - ```bash
    kops get ig -o yaml | yh
    ```

    - ![image-20230124214529877](files/img/image-20230124214529877.png)

    - ```yaml
      apiVersion: kops.k8s.io/v1alpha2
      kind: InstanceGroup
      metadata:
        creationTimestamp: "2023-01-24T11:44:45Z"
        labels:
          kops.k8s.io/cluster: learn-dc.link
        name: master-us-east-1a
      spec:
        image: 099720109477/ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20221206
        instanceMetadata:
          httpPutResponseHopLimit: 3
          httpTokens: required
        machineType: t3.medium
        maxSize: 1
        minSize: 1
        role: Master
        subnets:
        - us-east-1a
      ---
      apiVersion: kops.k8s.io/v1alpha2
      kind: InstanceGroup
      metadata:
        creationTimestamp: "2023-01-24T11:44:45Z"
        labels:
          kops.k8s.io/cluster: learn-dc.link
        name: nodes-us-east-1a
      spec:
        image: 099720109477/ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20221206
        instanceMetadata:
          httpPutResponseHopLimit: 1
          httpTokens: required
        machineType: t3.medium
        maxSize: 1
        minSize: 1
        role: Node
        subnets:
        - us-east-1a
      ---
      apiVersion: kops.k8s.io/v1alpha2
      kind: InstanceGroup
      metadata:
        creationTimestamp: "2023-01-24T11:44:45Z"
        labels:
          kops.k8s.io/cluster: learn-dc.link
        name: nodes-us-east-1c
      spec:
        image: 099720109477/ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-20221206
        instanceMetadata:
          httpPutResponseHopLimit: 1
          httpTokens: required
        machineType: t3.medium
        maxSize: 1
        minSize: 1
        role: Node
        subnets:
        - us-east-1c

- 인스턴스 정보 확인

  - ```bash
    kops get instances
    ```

    - ![image-20230124214558218](files/img/image-20230124214558218.png)

  - ```bash
    kops get instances -o yaml | yh
    ```

    - ![image-20230124214620237](files/img/image-20230124214620237.png)

    - ```yaml
      - id: i-07141eab6cc9bb2e4
        instanceGroup: master-us-east-1a.masters.learn-dc.link
        internalIP: 172.30.37.165
        machineType: t3.medium
        nodeName: i-07141eab6cc9bb2e4
        roles:
        - master
        state: ""
        status: UpToDate
      - id: i-0d3406b74013db109
        instanceGroup: nodes-us-east-1a.learn-dc.link
        internalIP: 172.30.46.72
        machineType: t3.medium
        nodeName: i-0d3406b74013db109
        roles:
        - node
        state: ""
        status: UpToDate
      - id: i-0e94665f9bcb7353d
        instanceGroup: nodes-us-east-1c.learn-dc.link
        internalIP: 172.30.89.212
        machineType: t3.medium
        nodeName: i-0e94665f9bcb7353d
        roles:
        - node
        state: ""
        status: UpToDate



##### 3. k8s 정보 확인 & krew

- 클러스터 정보 확인

  - ```bash
    kubectl cluster-info
    ```

    - ![image-20230124214654259](files/img/image-20230124214654259.png)

  - ```bash
    kubectl cluster-info dump
    ```

    - ![image-20230124214807696](files/img/image-20230124214807696.png)
    - ![image-20230124214813339](files/img/image-20230124214813339.png)

- 노드 정보 확인

  - ```bash
    kubectl get nodes -o wide
    ```

    - ![image-20230124214828693](files/img/image-20230124214828693.png)

  - ```bash
    kubectl get nodes -v6
    ```

    - ![image-20230124214848949](files/img/image-20230124214848949.png)

- 파드 정보 확인

  - ```bash
    kubectl get pod -A
    ```

    - ![image-20230124214912495](files/img/image-20230124214912495.png)

- krew 플러그인 설치 확인

  - ```bash
    kubectl krew list
    ```

    - ![image-20230124214934801](files/img/image-20230124214934801.png)

- kube-system 오브젝트 전체 확인

  - ```bash
    kubectl get-all -n kube-system
    ```

    - ![image-20230124215019655](files/img/image-20230124215019655.png)

- 모니터링 (플러그인)

  - ```bash
    kubectl ktop
    ```

    - ![image-20230124215044561](files/img/image-20230124215044561.png)
    - ![image-20230124215108289](files/img/image-20230124215108289.png)

- 노드 인스턴스 타입 확인

  - ```bash
    kubectl describe nodes | grep "node.kubernetes.io/instance-type"
    ```

    - ![image-20230124215131621](files/img/image-20230124215131621.png)



##### 4. EC2 정보 확인

- 노드 IP 확인

  - ```bash
    aws ec2 describe-instances --output table
    ```

    - ![image-20230124215216158](files/img/image-20230124215216158.png)
    - ![image-20230124215224326](files/img/image-20230124215224326.png)

  - ```bash
    aws ec2 describe-instances --query "Reservations[*].Instances[*].{PublicIPAdd:PublicIpAddress,PrivateIPAdd:PrivateIpAddress,InstanceName:Tags[?Key=='Name']|[0].Value,Status:State.Name}" --filters Name=instance-state-name,Values=running --output table
    ```



##### 5. AWS Route53 도메인 정보 확인

- 자신의 도메인 변수 지정 : 소유하고 있는 자신의 도메인을 입력하시면 됩니다.

  - ```bash
    # MyDomain=<자신의 도메인>
    MyDomain=learn-dc.link

- 자신의 Route 53 도메인 ID 조회 및 변수 지정

  - ```bash
    aws route53 list-hosted-zones-by-name --dns-name "${MyDomain}." | jq
    ```

  - ```bash
    MyDnzHostedZoneId=`aws route53 list-hosted-zones-by-name --dns-name "${MyDomain}." --query "HostedZones[0].Id" --output text`

- A 레코드 타입 조회

  - ```bash
    aws route53 list-resource-record-sets --hosted-zone-id "${MyDnzHostedZoneId}" --query "ResourceRecordSets[?Type == 'A']" | jq

- A 레코드 값 반복 조회

  - ```bash
    while true; do aws route53 list-resource-record-sets --hosted-zone-id "${MyDnzHostedZoneId}" --query "ResourceRecordSets[?Type == 'A']" | jq ; date ; echo ; sleep 1; done
    ```

    - ![image-20230124215304173](files/img/image-20230124215304173.png)



##### 6. (옵션) 노드에 배포된 컨테이너 정보 확인

- Containerd clients 3종 : ctr, nerdctl, crictl
  - ![출처: PKOS 스터디(Containerd clients 3종)](files/img/image-20230124185204453.png)
    - https://iximiuz.com/en/posts/containerd-command-line-clients/

- 아래는 마스터 노드 정보를 확인

- ctr 버전 확인

  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME ctr --version
    ```

    - ![image-20230124215336943](files/img/image-20230124215336943.png)

- ctr help

  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME ctr
    ```

    - ![image-20230124215438595](files/img/image-20230124215438595.png)

- 네임스페이스 확인

  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME sudo ctr ns list
    ```

    - ![image-20230124215458414](files/img/image-20230124215458414.png)

- 컨테이너 리스트 확인

  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME sudo ctr -n k8s.io container list
    ```

    - ![image-20230124215519200](files/img/image-20230124215519200.png)

- 컨테이너 이미지 확인

  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME sudo ctr -n k8s.io image list --quiet
    ```

    - ![image-20230124215533757](files/img/image-20230124215533757.png)

- 태스크 리스트 확인

  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME sudo ctr -n k8s.io task list
    ```

    - ![image-20230124215606336](files/img/image-20230124215606336.png)

- 예시) 각 태스크의 PID(5923) 확인

  - ```bash
    # ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME sudo ps -c <PID>
    ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME sudo ps -c 5789
    ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME sudo ps -c 5051 5871 6064
    ```

    - ![image-20230124215718739](files/img/image-20230124215718739.png)
    - ![image-20230124215818361](files/img/image-20230124215818361.png)





## 2. 쿠버네티스 네트워크

### 2.1 AWS VPC CNI(Container Network Interface) 소개

#### 2.1.1 개념

##### 1. K8S CNI

- CNI(Container Network Interface)는 k8s 네트워크 환경을 구성해주며 다양한 플러그인이 존재함
  - https://kubernetes.io/docs/concepts/cluster-administration/networking/
  - https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy

##### 2. AWS VPC CNI

- 파드의 IP를 할당해주며, 파드의 IP 네트워크 대역과 노드(워커)의 IP 대역이 같아서 직접 통신이 가능함
  - https://github.com/aws/amazon-vpc-cni-k8s
  - https://github.com/aws/amazon-vpc-cni-k8s/blob/master/docs/cni-proposal.md
- supports native VPC networking with the Amazon VPC Container Network Interface(CNI) plugin for Kubernetes.
- VPC와 통합
  - VPC Flow logs, VPC 라우팅 정책, ~~보안 그룹(SG;Security Group)~~ 사용 가능
    - kOps는 SG for Pod 미지원
      - https://reaperes.medium.com/security-group-%EC%9D%84-pod-%EB%8B%A8%EC%9C%84%EB%A1%9C-%ED%95%A0%EB%8B%B9%ED%95%98%EA%B8%B0-bef922065684
- This Plugin assigns an IP address from your VPC to each pod.
- VPC ENI(Elastic Network Interface)에 미리 할당된 IP를 파드에서 사용할 수 있음

##### 3. K8S Calico CNI 와 AWS VPC CNI 차이

- 네트워크 통신의 최적화(성능, 지연)를 위해 노드와 파드의 네트워크 대역을 동일하게 설정함
  - ![출처: PKOS 스터디(K8S Calico CNI 와 AWS VPC CNI 차이) - 일반적인 K8S CNI 플러그인(Calico) 와 AWS VPC CNI 간 노드와 파드 네트워크 대역 비교](files/img/image-20230124161458533.png)
- 파드간 통신 시 일반적으로 `k8s CNI`는 오버레이(VXLAN, IP-IP 등) 통신을 하고, `AWS VPC CNI`는 동일 대역으로 직접 통신함
  - ![출처: PKOS 스터디(K8S Calico CNI 와 AWS VPC CNI 차이)](files/img/image-20230124162255148.png)

##### 4. 워커 노드에 생성 가능한 최대 파드 갯수

- Secondary IPv4 addresses
  - 인스턴스 유형에 최대 ENI 갯수와 할당 가능 IP 수를 조합하여 선정
    - https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/using-eni.html
- IPv4 Prefix Delegation
  - IPv4 28bit 서브넷(prefix)를 위임하여 할당 가능 IP 수와 인스턴스 유형에 권장하는 최대 갯수로 선정

![출처: PKOS 스터디(워커 노드에 생성 가능한 최대 파드 갯수)](files/img/image-20230124163014278.png)



#### 2.1.2 실습

##### 1. 네트워크 기본 정보 확인

- CNI 정보 확인

  - ```bash
    kubectl describe daemonset aws-node --namespace kube-system | grep Image | cut -d "/" -f 2
    ```
    
    - ![image-20230124215920509](files/img/image-20230124215920509.png)

- 노드 IP 확인

  - ```bash
    aws ec2 describe-instances --query "Reservations[*].Instances[*].{PublicIPAdd:PublicIpAddress,PrivateIPAdd:PrivateIpAddress,InstanceName:Tags[?Key=='Name']|[0].Value,Status:State.Name}" --filters Name=instance-state-name,Values=running --output table
    ```
    
    - ![image-20230124215936514](files/img/image-20230124215936514.png)

- 파드 IP 확인

  - ```bash
    kubectl get pod -n kube-system -o=custom-columns=NAME:.metadata.name,IP:.status.podIP,STATUS:.status.phase
    ```
    
    - ![image-20230124220001151](files/img/image-20230124220001151.png)

- 파드 이름 확인

  - ```bash
    kubectl get pod -A -o name
    ```
    
    - ![image-20230124220018194](files/img/image-20230124220018194.png)



- 파드 갯수 확인

  - ```bash
    kubectl get pod -A -o name | wc -l
    ```
    
    - ![image-20230124220034594](files/img/image-20230124220034594.png)

- 파드 갯수 확인(플러그인)

  - https://github.com/vladimirvivien/ktop

  - ```bash
    kubectl ktop
    ```
    
    - ![image-20230124220050136](files/img/image-20230124220050136.png)
    - ![image-20230124215108289](files/img/image-20230124215108289.png)



- [master node] aws vpc cni log

  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME ls /var/log/aws-routed-eni
    ```

    - ![image-20230124220151155](files/img/image-20230124220151155.png)



##### 2. master node에 SSH 접속 후 확인

- [mater node] SSH 접속

  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME
    ```
    
    - ![image-20230124220207630](files/img/image-20230124220207630.png)



- 툴 설치

  - ```bash
    sudo apt install -y tree jq net-tools
    ```
    
    - ![image-20230124220228165](files/img/image-20230124220228165.png)
    - ![image-20230124220235397](files/img/image-20230124220235397.png)




- CNI 정보 확인

  - ```bash
    ls /var/log/aws-routed-eni
    ```
    
    - ![image-20230124220257165](files/img/image-20230124220257165.png)
    
  - ```bash
    cat /var/log/aws-routed-eni/plugin.log | jq
    ```
  
    - ![image-20230124220331543](files/img/image-20230124220331543.png)
    - ![image-20230124220310788](files/img/image-20230124220310788.png)
    
  - ```bash
    cat /var/log/aws-routed-eni/ipamd.log | jq
    ```
    
    - ![image-20230124220348197](files/img/image-20230124220348197.png)



- 네트워크 정보 확인 : eniY는 pod network 네임스페이스와 veth pair

  - ```bash
    ip -br -c addr
    ```

    - ![image-20230124220421338](files/img/image-20230124220421338.png)
    
  - ```bash
    ip -c addr
    ```
  
    - ![image-20230124220438823](files/img/image-20230124220438823.png)
    
  - ```bash
    ip -c route
    ```

    - ![image-20230124220450531](files/img/image-20230124220450531.png)
    
  - ```bash
    sudo iptables -t nat -S
    ```
  
    - ![image-20230124220507254](files/img/image-20230124220507254.png)
    - ![image-20230124220512686](files/img/image-20230124220512686.png)
    
  - ```bash
    sudo iptables -t nat -L -n -v
    ```
    
    - ![image-20230124220532471](files/img/image-20230124220532471.png)
    - ![image-20230124220525432](files/img/image-20230124220525432.png)



- 빠져나오기

  - ```bash
    exit
    ```
    
    - ![image-20230124220547791](files/img/image-20230124220547791.png)



##### 3. 워커 노드에 SSH 접속 후 확인 : 워커 노드의 public ip로 SSH 접속

- 워커 노드 public IP 확인

  - ```bash
    aws ec2 describe-instances --query "Reservations[*].Instances[*].{PublicIPAdd:PublicIpAddress,InstanceName:Tags[?Key=='Name']|[0].Value}" --filters Name=instance-state-name,Values=running --output table
    ```
    
    - ![image-20230124220608973](files/img/image-20230124220608973.png)



- 워커 노드 Public IP 변수 지정

  - ```bash
    # W1PIP=<워커 노드 1 Public IP>
    W1PIP=3.86.36.119
    # W2PIP=<워커 노드 2 Public IP>
    W2PIP=35.168.13.194
    
    echo $W1PIP
    echo $W2PIP
    ```
    
    - ![image-20230124220804221](files/img/image-20230124220804221.png)



- 워커 노드 SSH 접속 확인

  - 워커 노드1

    - ```bash
      ssh -i ~/.ssh/id_rsa ubuntu@$W1PIP
      exit
      ```
      
      - ![image-20230124220830860](files/img/image-20230124220830860.png)
      - ![image-20230124220845197](files/img/image-20230124220845197.png)
  
  - 워커 노드2
  
    - ```bash
      ssh -i ~/.ssh/id_rsa ubuntu@$W2PIP
      exit
      ```
      
      - ![image-20230124220904006](files/img/image-20230124220904006.png)
      - ![image-20230124220912465](files/img/image-20230124220912465.png)



- [워커 노드1, 워커 노드2] SSH 접속 후 툴 설치 및 정보 확인

  - SSH 접속

    - ```bash
      # ssh -i ~/.ssh/id_rsa ubuntu@$W1PIP
      ssh -i ~/.ssh/id_rsa ubuntu@$W2PIP
      ```
      
      - ![image-20230124221117904](files/img/image-20230124221117904.png)
      - ![image-20230124221127097](files/img/image-20230124221127097.png)

  
  
  - 툴 설치

    - ```bash
      sudo apt install -y tree jq net-tools
      ```
      
      - ![image-20230124221144789](files/img/image-20230124221144789.png)
      - ![image-20230124221203525](files/img/image-20230124221203525.png)
      - ![image-20230124221210623](files/img/image-20230124221210623.png)

  
  
  - CNI 정보 확인

    - ```bash
      ls /var/log/aws-routed-eni
      ```

      - ![image-20230124221224408](files/img/image-20230124221224408.png)
      - ![image-20230124221241892](files/img/image-20230124221241892.png)
    
    - ```bash
      cat /var/log/aws-routed-eni/plugin.log | jq
      ```
    
      - ![image-20230124221332380](files/img/image-20230124221332380.png)
    
      - ![image-20230124221257784](files/img/image-20230124221257784.png)
    
      - ![image-20230124221407359](files/img/image-20230124221407359.png)
    
      - ![image-20230124221303729](files/img/image-20230124221303729.png)
    
      - ``` json
        {
          "level": "info",
          "ts": "2023-01-24T11:48:47.337Z",
          "caller": "routed-eni-cni-plugin/cni.go:119",
          "msg": "Constructed new logger instance"
        }
        {
          "level": "info",
          "ts": "2023-01-24T11:48:47.337Z",
          "caller": "routed-eni-cni-plugin/cni.go:128",
          "msg": "Received CNI add request: ContainerID(c7eb1fca15b325f1586072de859212e5b4f120ecdb6b7f510cb150afcb40a863) Netns(/var/run/netns/cni-009d6902-2a84-8d87-ad75-c3012ccccc43) IfName(eth0) Args(IgnoreUnknown=1;K8S_POD_NAMESPACE=kube-system;K8S_POD_NAME=coredns-autoscaler-5685d4f67b-v7vfl;K8S_POD_INFRA_CONTAINER_ID=c7eb1fca15b325f1586072de859212e5b4f120ecdb6b7f510cb150afcb40a863;K8S_POD_UID=02872caf-9307-4599-a808-77fdc99d1d7e) Path(/opt/cni/bin) argsStdinData({\"cniVersion\":\"0.4.0\",\"mtu\":\"9001\",\"name\":\"aws-cni\",\"pluginLogFile\":\"/var/log/aws-routed-eni/plugin.log\",\"pluginLogLevel\":\"DEBUG\",\"podSGEnforcingMode\":\"strict\",\"type\":\"aws-cni\",\"vethPrefix\":\"eni\"})"
        }
        {
          "level": "debug",
          "ts": "2023-01-24T11:48:47.337Z",
          "caller": "routed-eni-cni-plugin/cni.go:128",
          "msg": "Prev Result: <nil>\n"
        }
        {
          "level": "debug",
          "ts": "2023-01-24T11:48:47.337Z",
          "caller": "routed-eni-cni-plugin/cni.go:128",
          "msg": "MTU value set is 9001:"
        }
        {
          "level": "info",
          "ts": "2023-01-24T11:48:47.343Z",
          "caller": "routed-eni-cni-plugin/cni.go:128",
          "msg": "Received add network response from ipamd for container c7eb1fca15b325f1586072de859212e5b4f120ecdb6b7f510cb150afcb40a863 interface eth0: Success:true  IPv4Addr:\"172.30.37.39\"  VPCv4CIDRs:\"172.30.0.0/16\""
        }
        {
          "level": "debug",
          "ts": "2023-01-24T11:48:47.343Z",
          "caller": "routed-eni-cni-plugin/cni.go:237",
          "msg": "SetupPodNetwork: hostVethName=eni02955105579, contVethName=eth0, netnsPath=/var/run/netns/cni-009d6902-2a84-8d87-ad75-c3012ccccc43, v4Addr=172.30.37.39/32, v6Addr=<nil>, deviceNumber=0, mtu=9001"
        }
        {
          "level": "debug",
          "ts": "2023-01-24T11:48:47.400Z",
          "caller": "driver/driver.go:281",
          "msg": "Successfully disabled IPv6 RA and ICMP redirects on hostVeth eni02955105579"
        }
        {
          "level": "debug",
          "ts": "2023-01-24T11:48:47.408Z",
          "caller": "driver/driver.go:297",
          "msg": "Successfully setup container route, containerAddr=172.30.37.39/32, hostVeth=eni02955105579, rtTable=main"
        }
        {
          "level": "debug",
          "ts": "2023-01-24T11:48:47.408Z",
          "caller": "driver/driver.go:297",
          "msg": "Successfully setup toContainer rule, containerAddr=172.30.37.39/32, rtTable=main"
        }
        parse error: Invalid numeric literal at line 10, column 5
        ```
    
    - ```bash
      cat /var/log/aws-routed-eni/ipamd.log | jq
      ```
    
      - ![image-20230124221523980](files/img/image-20230124221523980.png)
      - ![image-20230124221536816](files/img/image-20230124221536816.png)
  
  
  
  - 네트워크 정보 확인 : eniY는 pod network 네임스페이스와 veth pair
  
    - ```bash
      ip -br -c addr
      ```
  
      - ![image-20230124221554923](files/img/image-20230124221554923.png)
      - ![image-20230124221559555](files/img/image-20230124221559555.png)
      
    - ```bash
      ip -c addr
      ```
    
      - ![image-20230124221708476](files/img/image-20230124221708476.png)
      - ![image-20230124221713531](files/img/image-20230124221713531.png)
      
    - ```bash
      ip -c route
      ```
    
      - ![image-20230124221737695](files/img/image-20230124221737695.png)
      - ![image-20230124221742802](files/img/image-20230124221742802.png)
      
    - ```bash
      sudo iptables -t nat -S
      ```
    
      - ![image-20230124221810068](files/img/image-20230124221810068.png)
      - ![image-20230124221816915](files/img/image-20230124221816915.png)
      - ![image-20230124221821983](files/img/image-20230124221821983.png)
      - ![image-20230124221826656](files/img/image-20230124221826656.png)
      
    - ```bash
      sudo iptables -t nat -L -n -v
      ```
      
      - ![image-20230124221848105](files/img/image-20230124221848105.png)
      - ![image-20230124221854273](files/img/image-20230124221854273.png)
      - ![image-20230124221902177](files/img/image-20230124221902177.png)
      - ![image-20230124221911009](files/img/image-20230124221911009.png)
  
  
  
  - 빠져나오기
  
    - ```bash
      exit
      ```
      
      - ![image-20230124221937876](files/img/image-20230124221937876.png)
      - ![image-20230124221941861](files/img/image-20230124221941861.png)




### 2.2 노트에서 기본 네트워크 정보 확인

#### 2.2.1 개념

##### 1. 워커 노드1 기본 네트워크 구성 : 워커 노드2는 구성이 유사하여 생략

![출처: PKOS 스터디(워커 노드 기본 네트워크 구성)](files/img/image-20230124164936061.png)

- Network NS(Name Space)는 호스트(Root)와 파드 별(Per Pod)로 구분

- 특정한 파드(kube-proxy, aws-node)는 호스트(Root)의 IP를 그대로 사용

- t3.medium의 경우 ENI에 최대 6개의 IP를 가질 수 있음

  - | 인스턴스 유형 | 최대 네트워크 인터페이스 수 | 인터페이스당 프라이빗 IPv4 주소 수 | 인터페이스당 IPv6 주소 수 |
    | :------------ | :-------------------------- | :--------------------------------- | :------------------------ |
    | `t3.medium`   | 3                           | 6                                  | 6                         |

    - https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/using-eni.html

- ENI0, ENI1으로 2개의 ENI는 자신의 IP 이외에 추가적으로 5개의 보조 프라이빗 IP를 가질 수 있음
- coredns 파드는 veth으로 호스트에는 eniY@ifN 인터페이스와 파드에 eth0과 연결되어 있음



##### 2. 워커 노드1 인스턴스의 네트워크 정보 확인

![출처: PKOS 스터디(워커 노드 인스턴스의 네트워크 정보 확인)](files/img/image-20230124165014979.png)





#### 2.2.2 실습

##### 1. 보조 IPv4 주소를 파드가 사용하는지 확인

- ebs-csi-node 파드 IP 정보 확인

  - ```bash
    kubectl get pod -n kube-system -l app=ebs-csi-node -owide
    ```
    
    - ![image-20230124222044009](files/img/image-20230124222044009.png)



- 노드의 라우팅 정보 확인 >> EC2 네트워크 정보의 '보조 프라이빗 IPv4 주소'와 비교해보자

  - ```bash
    echo $KOPS_CLUSTER_NAME
    echo $W1PIP
    echo $W2PIP
    ```

    - ![image-20230124222551478](files/img/image-20230124222551478.png)
  
  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@api.$KOPS_CLUSTER_NAME ip -c route
    ```
  
    - ![image-20230124222201469](files/img/image-20230124222201469.png)
    - ![image-20230124222252325](files/img/image-20230124222252325.png)
  
  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@$W1PIP ip -c route
    ```
  
    - ![image-20230124222132924](files/img/image-20230124222132924.png)
    - ![image-20230124222310141](files/img/image-20230124222310141.png)
  
  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@$W2PIP ip -c route
    ```
  
    - ![image-20230124222147927](files/img/image-20230124222147927.png)
    - ![image-20230124222347484](files/img/image-20230124222347484.png)
  



##### 2. 테스트용 파드 생성 - [nicolaka/netshoot](https://github.com/nicolaka/netshoot)

- [터미널1, 터미널2] 워커 노드1, 워커 노드2 모니터링

  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@$W1PIP
    watch -d "ip link | egrep 'ens5|eni' ;echo;echo "[ROUTE TABLE]"; route -n | grep eni"
    ```

    - ![image-20230124223102232](files/img/image-20230124223102232.png)
    
  - ```bash
    ssh -i ~/.ssh/id_rsa ubuntu@$W2PIP
    watch -d "ip link | egrep 'ens5|eni' ;echo;echo "[ROUTE TABLE]"; route -n | grep eni"
    ```
    
    - ![image-20230124223055327](files/img/image-20230124223055327.png)



- 작업용 EC2 파드 2개 생성

  - ```bash
    cat ~/pkos/2/netshoot-2pods.yaml | yh
    ```
    
    - ![image-20230124223150975](files/img/image-20230124223150975.png)
  
  - ```bash
    kubectl apply -f ~/pkos/2/netshoot-2pods.yaml
    ```
  
    - ![image-20230124223326699](files/img/image-20230124223326699.png)
    - ![image-20230124223348864](files/img/image-20230124223348864.png)



- 파드 확인

  - ```bash
    kubectl get pod -o wide
    ```
  
    - ![image-20230124223411409](files/img/image-20230124223411409.png)
    
  - ```bash
    kubectl get pod -o=custom-columns=NAME:.metadata.name,IP:.status.podIP
    ```
  
    - ![image-20230124223422678](files/img/image-20230124223422678.png)
    
  - 만약 파드 2개가 모두 1개의 노드에 배치 시 강제로 파드 1개를 다른 노드에 배치
  
    - 불편하게 nodeName 설정하지 않게 해당 실습을 netshoot 데몬셋으로 변경할 것
  
    - 파드 1개 삭제
  
      - ```bash
        kubectl delete pod pod-2
  
    - yaml 파일 편집
  
      - ```bash
        vim ~/pkos/2/netshoot-2pods.yaml
        # ...
        # spec:
        #   nodeName: i-0d3406b74013db109  <- 각자 자신의 노드의 이름을 직접 입력
        #   containers:
        #   - name: netshoot-pod
        # ...
  
    - 재배포
  
      - ```bash
        kubectl apply -f ~/pkos/2/netshoot-2pods.yaml
  
    - 파드 배포 확인
  
      - ```bash
        kubectl get pod -o wide
  



- 파드가 생성되면 **워커 노드에 eniY@ifN 추가**되고 라우팅 테이블에도 정보가 추가됨

- 테스트용 파드 eniY 정보 확인 - 워커 노드 EC2

  - 노드에서 네트워크 인터페이스 정보 확인
  
    - ```bash
      ip -br -c addr show
      ```
  
      - ![image-20230124223554227](files/img/image-20230124223554227.png)
      
    - ```bash
      ip -c link
      ```
  
      - ![image-20230124223604661](files/img/image-20230124223604661.png)
      
    - ```bash
      ip -c addr
      ```
  
      - ![image-20230124223613079](files/img/image-20230124223613079.png)
      
    - ```bash
      # 또는 route -n
      ip -c route
      ```
      
      - ![image-20230124223625760](files/img/image-20230124223625760.png)
      - ![image-20230124223709964](files/img/image-20230124223709964.png)
  
  - 마지막 생성된 네임스페이스 정보 출력 -t net(네트워크 타입)
  
    - ```bash
      sudo lsns -o PID,COMMAND -t net | awk 'NR>2 {print $1}' | tail -n 1
      ```
  
  - 마지막 생성된 네임스페이스 net PID 정보 출력 -t net(네트워크 타입)을 변수 지정
  
    - ```bash
      MyPID=$(sudo lsns -o PID,COMMAND -t net | awk 'NR>2 {print $1}' | tail -n 1)
      ```
  
  - PID 정보로 파드 정보 확인
  
    - ```bash
      sudo nsenter -t $MyPID -n ip -c addr
      ```
  
    - ```bash
      sudo nsenter -t $MyPID -n ip -c route
      ```



- 테스트용 파드 접속

  - 테스트용 파드 접속(exec) 후 Shell 실행

    - ```bash
      # kubectl exec -it pod-2 -- zsh
      kubectl exec -it pod-1 -- zsh
      ```
      
      - ![image-20230124224201371](files/img/image-20230124224201371.png)
  
  - 아래부터는 pod-1 Shell 에서 실행 : 네트워크 정보 확인
  
    - ```bash
      ip -c addr
      ```
  
      - ![image-20230124224217446](files/img/image-20230124224217446.png)
      
    - ```bash
      ip -c route
      ```
  
      - ![image-20230124224234519](files/img/image-20230124224234519.png)
      
    - ```bash
      route -n
      ```
      
      - ![image-20230124224249908](files/img/image-20230124224249908.png)
      
    - ```bash
      # ping -c 1 <pod-2 IP>
      ping -c 1 172.30.62.233
      ```
      
      - ![image-20230124223422678](files/img/image-20230124223422678.png)
      - ![image-20230124224438024](files/img/image-20230124224438024.png)
      
    - ```bash
      ps
      ```
      
      - ![image-20230124224446743](files/img/image-20230124224446743.png)
      
    - ```bash
      cat /etc/resolv.conf
      ```
      
      - ![image-20230124224457730](files/img/image-20230124224457730.png)
      
    - ```bash
      exit
      ```
      
      - ![image-20230124224512825](files/img/image-20230124224512825.png)
  
  - 파드2 Shell 실행 후 위와 동일하게 실행
  
    - ```bash
      kubectl exec -it pod-2 -- zsh
      ```
    
      - ![image-20230124224558510](files/img/image-20230124224558510.png)






### 2.3 노드 간 파드 통신

#### 2.3.1 개념

##### 1. 파드간 통신 흐름 : 별도의 오버레이(Overlay) 통신 기술 없이, VPC Native 하게 파드간 직접 통신이 가능하다

- ![출처: PKOS 스터디(파드간 통신 흐름)](files/img/image-20230124171420219.png)
- (참고)파드간 통신 시 과정
  - ![출처: PKOS 스터디(파드간 통신 시 과정)](files/img/image-20230124171456224.png)



#### 2.3.2 실습

##### 1. 파드간 통신 테스트 및 확인 : 별도의 NAT 동작 없이 통신 가능!

- 파드 IP 변수 지정

  - ```bash
    POD1=$(kubectl get pod pod-1 -o jsonpath={.status.podIP})
    echo $POD1
    ```
  
    - ![image-20230124224805539](files/img/image-20230124224805539.png)
    
  - ```bash
    POD2=$(kubectl get pod pod-2 -o jsonpath={.status.podIP})
    echo $POD2
    ```
    
    - ![image-20230124224817724](files/img/image-20230124224817724.png)



- 파드1 Shell 에서 파드2로 ping 테스트

  - ```bash
    kubectl exec -it pod-1 -- ping -c 2 $POD2
    ```
    
    - ![image-20230124224832728](files/img/image-20230124224832728.png)

- 파드2 Shell 에서 파드1로 ping 테스트

  - ```bash
    kubectl exec -it pod-2 -- ping -c 2 $POD1
    ```
    
    - ![image-20230124224849841](files/img/image-20230124224849841.png)

- 워커 노드 EC2 : TCPDUMP 확인 - ens6 에서 패킷 덤프 확인이 되나요?

  - ```bash
    sudo tcpdump -i any -nn icmp
    ```
  
    - ![image-20230124225033575](files/img/image-20230124225033575.png)
    
  - ```bash
    sudo tcpdump -i ens5 -nn icmp
    ```
    
    - ![image-20230124225121591](files/img/image-20230124225121591.png)
    
  - ```bash
    sudo tcpdump -i ens6 -nn icmp
    ```
    
    - ![image-20230124225152251](files/img/image-20230124225152251.png)



- [워커 노드1]

  - routing policy database management 확인
  
    - ```bash
      ip rule
      ```
      
      - ![image-20230124225218728](files/img/image-20230124225218728.png)
  
  - routing table management 확인
  
    - ```bash
      ip route show table local
      ```
      
      - ![image-20230124225228811](files/img/image-20230124225228811.png)
  
  - 디폴트 네트워크 정보를 ens5를 통해 빠져나감
  
    - ```bash
      ip route show table main
      ```
      
      - ![image-20230124225241336](files/img/image-20230124225241336.png)
  



#### 2.3.3 과제1

##### 1. 과제 내용

목표 : 파드간 통신 시 tcpdump 내용을 확인하고 관련 스샷을 올려주세요



##### 2. 과제 수행내용

- [워커 노드1, 워커 노드2] 모니터링

  - ```bash
    sudo tcpdump -i ens5 -nn icmp
    ```
    
    - ![image-20230124225549999](files/img/image-20230124225549999.png)

- 워커 노드1 에서 워커 노드 2로 핑 전송

  - ```bash
    kubectl exec -it pod-1 -- ping -c 2 $POD2
    ```
    
    - ![image-20230124225611185](files/img/image-20230124225611185.png)





### 2.4 파드에서 외부 통신

#### 2.4.1 개념

##### 1. 파드에서 외부 통신 흐름 : iptable에 SNAT을 통하여 노드의 eth0 IP로 변경되어서 외부와 통신됨

- VPC CNI의 External source network address translation(SNAT) 설정에 따라, 외부(인터넷) 통신 시 SNAT 하거나 혹은 SNAT 없이 통신 할 수 있음
  - https://docs.aws.amazon.com/eks/latest/userguide/external-snat.html
- ![출처: PKOS 스터디(파드에서 외부 통신 흐름)](files/img/image-20230124171949415.png)



#### 2.4.2 실습

##### 1. 파드에서 외부 통신 테스트 및 확인

- 파드 shell 실행 후 외부로 ping 테스트 & 워커 노드에서 tcpdump 및 iptables 정보 확인

  - 작업용 EC2 : pod-1 Shell 에서 외부로 ping
  
    - ```bash
      kubectl exec -it pod-1 -- ping -c 1 www.google.com
      ```
  
      - ![image-20230124225726718](files/img/image-20230124225726718.png)
      
    - ```bash
      kubectl exec -it pod-1 -- ping -i 0.1 www.google.com
      ```
      
      - ![image-20230124225758562](files/img/image-20230124225758562.png)
  
  - 워커 노드 EC2 : public IP 확인, tcpdump 확인
  
    - ```bash
      curl -s ipinfo.io/ip ; echo
      ```
    
      - ![image-20230124225910165](files/img/image-20230124225910165.png)
      
    - ```bash
      sudo tcpdump -i any -nn icmp
      ```
      
      - ![image-20230124225926636](files/img/image-20230124225926636.png)
      
    - ```bash
      sudo tcpdump -i ens5 -nn icmp
      ```
      
      - ![image-20230124225938875](files/img/image-20230124225938875.png)
    
  - 작업용 EC2 : pod-1 Shell 에서 외부 접속 확인 - 공인 IP는 어떤 주소인가?
  
    - ```bash
      kubectl exec -it pod-1 -- curl -s ipinfo.io/ip ; echo
      ```
    
      - ![image-20230124230009566](files/img/image-20230124230009566.png)
      
    - ```bash
      kubectl exec -it pod-1 -- curl -s wttr.in/seoul
      ```
      
      - ![image-20230124230028906](files/img/image-20230124230028906.png)
      
    - ```bash
      kubectl exec -it pod-1 -- curl -s wttr.in/seoul?format=3
      ```
      
      - ![image-20230124230040800](files/img/image-20230124230040800.png)
      
    - ```bash
      kubectl exec -it pod-1 -- curl -s wttr.in/Moon
      ```
      
      - ![image-20230124230059001](files/img/image-20230124230059001.png)
      
    - ```bash
      kubectl exec -it pod-1 -- curl -s wttr.in/:help
      ```
      
      - ![image-20230124230129095](files/img/image-20230124230129095.png)
      - ![image-20230124230145707](files/img/image-20230124230145707.png)
    
  - 워커 노드 EC2
  
    - 출력된 결과를 보고 어떻게 빠져나가는지 고민해보자!
  
      - ```bash
        ip rule
        ```
      
        - ![image-20230124230231397](files/img/image-20230124230231397.png)
        
      - ```bash
        ip route show table main
        ```
      
        - ![image-20230124230248004](files/img/image-20230124230248004.png)
        
      - ```bash
        sudo iptables -L -n -v -t nat
        ```
      
        - ![image-20230124230259218](files/img/image-20230124230259218.png)
        - ![image-20230124230308411](files/img/image-20230124230308411.png)
        
      - ```bash
        sudo iptables -t nat -S
        ```
        
        - ![image-20230124230320446](files/img/image-20230124230320446.png)
        - ![image-20230124230325360](files/img/image-20230124230325360.png)
      
    - 파드가 외부와 통신시에는 아래 처럼 'AWS-SNAT-CHAIN-0, AWS-SNAT-CHAIN-1' 룰(rule)에 의해서 SNAT 되어서 외부와 통신!
    
      - 참고로 뒤 IP는 eth0(ENI 첫번째)의 IP 주소이다
      
      - --random-fully 동작
      
        - https://ssup2.github.io/issue/Linux_TCP_SYN_Packet_Drop_SNAT_Port_Race_Condition/
        - https://ssup2.github.io/issue/Kubernetes_TCP_Connection_Delay_VXLAN_CNI_Plugin/
      
      - ```bash
        sudo iptables -t nat -S | grep 'A AWS-SNAT-CHAIN'
        ```
        
        - ![image-20230124230514612](files/img/image-20230124230514612.png)
        - ![image-20230124230357508](files/img/image-20230124230511828.png)
      
    - 아래 'mark 0x4000/0x4000' 매칭되지 않아서 RETURN 됨!
    
      - ```bash
        # -A KUBE-POSTROUTING -m mark ! --mark 0x4000/0x4000 -j RETURN
        # -A KUBE-POSTROUTING -j MARK --set-xmark 0x4000/0x0
        # -A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -j MASQUERADE --random-fully
        # ...
        ```
    
    - 카운트 확인 시 'AWS-SNAT-CHAIN-0, AWS-SNAT-CHAIN-1'에 매칭되어, 목적지가 '172.30.0.0/16'이 아닌 외부로 빠져나갈 떄 SNAT 172.30.85.242 변경되어 나감
    
      - ```bash
        sudo iptables -t filter --zero; sudo iptables -t nat --zero; sudo iptables -t mangle --zero; sudo iptables -t raw --zero
        ```
      
      - ```bash
        watch -d 'sudo iptables -v --numeric --table nat --list AWS-SNAT-CHAIN-0; echo ; sudo iptables -v --numeric --table nat --list AWS-SNAT-CHAIN-1; echo ; sudo iptables -v --numeric --table nat --list KUBE-POSTROUTING'
        ```
        
        - ![image-20230124230605144](files/img/image-20230124230605144.png)
        - ![image-20230124230552252](files/img/image-20230124230602663.png)
      
    - conntrack 확인
    
      - ```bash
        # sudo conntrack -L -n |grep -v '169.254.169'
        sudo conntrack -L -n
        ```
        
        - ![image-20230124230917455](files/img/image-20230124230917455.png)
  
- 다음 실습을 위해 파드 삭제

  - ```bash
    kubectl delete pod pod-1 pod-2
    ```
    
    - ![image-20230124230945714](files/img/image-20230124230945714.png)





### 2.5 노드에서 파드 생성 갯수 제한

#### 2.5.1 개념

##### 1. 워커 노드의 인스턴스 타입 별 파드 생성 갯수 제한

- Secondary IPv4 addresses : 인스턴스 유형에 최대 ENI 갯수와 할당 가능 IP 수를 조합하여 선정

- 인스턴스 타입  별 ENI 최대 갯수와 할당 가능한 최대 IP 갯수에 따라서 파드 배치 갯수가 결정됨

  - 단, aws-node 와 kube-proxy 파드는 호스트의 IP를 사용함으로 최대 갯수에서 제외함

- 인스턴스별 유형별 네트워크 인터페이스당 IP 주소

  - https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/using-eni.html

  - ```bash
    # AWS CLI 명령을 이용한 조회 - 예시:c5.*
    aws ec2 describe-instance-types --filters "Name=instance-type,Values=c5.*" --query "InstanceTypes[].{Type: InstanceType, MaxENI: NetworkInfo.MaximumNetworkInterfaces, IPv4addr: NetworkInfo.Ipv4AddressesPerInterface}" --output table
    
    # ---------------------------------------
    # |        DescribeInstanceTypes        |
    # +----------+----------+---------------+
    # | IPv4addr | MaxENI   |     Type      |
    # +----------+----------+---------------+
    # |  30      |  8       |  c5.4xlarge   |
    # |  50      |  15      |  c5.24xlarge  |
    # |  15      |  4       |  c5.xlarge    |
    # |  30      |  8       |  c5.12xlarge  |
    # |  10      |  3       |  c5.large     |
    # |  15      |  4       |  c5.2xlarge   |
    # |  50      |  15      |  c5.metal     |
    # |  30      |  8       |  c5.9xlarge   |
    # |  50      |  15      |  c5.18xlarge  |
    # +----------+----------+---------------+
    ```

- 최대 파드 생성 갯수 : (Number of network interfaces for the instance type × (the number of IP addressess per network interface - 1)) + 2

- ![출처: PKOS 스터디(노드에 파드 생성 갯수 제한)](files/img/image-20230124174629406.png)



#### 2.5.2 실습

##### 1. 워커 노드의 인스턴스 정보 확인 : 현재 t3.medium 사용 중

- t3 타입의 정보(필터) 확인

  - ```bash
    aws ec2 describe-instance-types --filters Name=instance-type,Values=t3.* \
     --query "InstanceTypes[].{Type: InstanceType, MaxENI: NetworkInfo.MaximumNetworkInterfaces, IPv4addr: NetworkInfo.Ipv4AddressesPerInterface}" \
     --output table
    ```
    
    - ![image-20230124231044545](files/img/image-20230124231044545.png)



- 파드 사용 가능 계산 예시 : aws-node와 kube-proxy 파드는 host-networking 사용으로 IP 2개 남음

  - ```bash
    # ((MaxENI * (IPv4addr-1)) + 2)
    # t3.medium 경우 : ((3 * (6 - 1) + 2 ) = 17개 >> aws-node 와 kube-proxy 2개 제외하면 15개
    ```



- 워커 노드 상세 정보 확인 : 노드 상세 정보의 Allocatable에 pods에 17개 정보 확인

  - ```bash
    kubectl describe node | grep Allocatable: -A6
    ```
    
    - ![image-20230124231056198](files/img/image-20230124231056198.png)



##### 2. 최대 파드 생성 및 확인

- 모니터링

  - 워커 노드 EC2 - 모니터링

    - ```bash
      watch -d "ip link | egrep 'ens|eni'"
      # while true; do ip -br -c addr show && echo "--------------" ; date "+%Y-%m-%d %H:%M:%S" ; sleep 1; done
      ```
      
      - ![image-20230124231700689](files/img/image-20230124231700689.png)


  - 작업용 EC2 - 터미널1

    - ```bash
      watch -d 'kubectl get pods -o wide'
      ```
      
      - ![image-20230124231714010](files/img/image-20230124231714010.png)




- 디플로이먼트 생성

  - ```bash
    cat ~/pkos/2/nginx-dp.yaml | yh
    ```
    
    - ![image-20230124231822762](files/img/image-20230124231822762.png)

  - ```bash
    kubectl apply -f ~/pkos/2/nginx-dp.yaml
    ```

    - ![image-20230124231859364](files/img/image-20230124231859364.png)


- 파드 확인

  - ```bash
    kubectl get pod -o wide

  - ```bash
    kubectl get pod -o=custom-columns=NAME:.metadata.name,IP:.status.podIP
    ```

  - ```bash
    kubectl ktop
    ```
    
    - ![image-20230124231946379](files/img/image-20230124231946379.png)
    - ![image-20230124231938048](files/img/image-20230124231938048.png)

- 파드 증가 테스트 >> 파드 정상 생성 확인, 워커 노드에서 eth, eni 갯수 확인

  - ```bash
    kubectl scale deployment nginx-deployment --replicas=8
    ```
    
    - ![image-20230124232043088](files/img/image-20230124232043088.png)

- 파드 증가 테스트 >> 파드 정상 생성 확인, 워커 노드에서 eth, eni 갯수 확인 >> 어떤일이 벌어 졌는가?

  - ```bash
    kubectl scale deployment nginx-deployment --replicas=22
    ```
    
    - ![image-20230124232121839](files/img/image-20230124232121839.png)
    - ![image-20230124232156536](files/img/image-20230124232156536.png)

- 파드 증가 테스트 >> 파드 정상 생성 확인, 워커 노드에서 eth, eni 갯수 확인 >> 어떤일이 벌어 졌는가?

  - ```bash
    kubectl scale deployment nginx-deployment --replicas=30
    ```
    
    - ![image-20230124232244381](files/img/image-20230124232244381.png)
    - ![image-20230124232252022](files/img/image-20230124232252022.png)

- 파드 생성 실패!

  - ```bash
    kubectl get pods | grep Pending

  - ```bash
    # kubectl describe pod <Pending 파드> | grep Events: -A5
    kubectl describe pod nginx-deployment-6fb79bc456-7m8n2 | grep Events: -A5
    kubectl describe pod nginx-deployment-6fb79bc456-cgq4b | grep Events: -A5
    ```
    
    - ![image-20230124232449969](files/img/image-20230124232449969.png)

- 디플로이먼트 삭제

  - ```bash
    kubectl delete deploy nginx-deployment
    ```
    
    - ![image-20230124232617849](files/img/image-20230124232617849.png)



##### 3. 해결 방안 : Prefix Delegation, WARM & MIN IP/Prefix Targets

- 참고
  - https://www.notion.so/4-f89e3e5967b24f8c9aa5bfaab1a82ceb#33243514f4684462bd8d657d00684b6f
  - [류승현]님 VPC CNI 옵션 설정
    - https://www.notion.so/ed085ee58b454f76afd6e0d94b049983
  - [류승현]님 kOps Docs
    - https://kops.sigs.k8s.io/networking/aws-vpc/#configuration
- ![출처: PKOS 스터디(POD 통신)](files/img/image-20230124175542054.png)

#### 2.5.3 과제2

##### 1. 과제 내용

목표 : 워커 노드 1대에 100대이상의 파드가 배포되게 설정하고 관련 스샷을 올려주세요

참고 :

- ```bash
  # 실습 참고참고
  kubectl apply -f ~/pkos/2/nginx-dp.yaml
  kubectl scale deployment nginx-deployment --replicas=110
  ```



##### 2. 과제 수행내용

- LimitRange, Kubelet 의 max-pods args, AWS VPC CNI(ENABLE_PREFIX_DELEGATION|WARM_PREFIX_TARGET)

1. (옵션) 동작 확인의 편리를 위해서 워커노드 c5.large 를 1대 배포하자

   - Nitro 인스턴스 유형 확인

     - ```bash
       aws ec2 describe-instance-types --filters Name=hypervisor,Values=nitro --query "InstanceTypes[*].[InstanceType]" --output text | sort | egrep 't3\.|c5\.'
       ```

       - ![image-20230125012621547](files/img/image-20230125012621547.png)

   - 워커노드 인스턴스 타입 변경(WorkerNodeInstanceType=t3.xlarge), 워커노드 1대 배포(WorkerNodeCount=1)

     - kops 인스턴스 그룹 확인

       - ```bash
         kops get ig
         ```

         - ![image-20230125015535618](files/img/image-20230125015535618.png)

     - kops 인스턴스 그룹 수정

       - ```bash
         kops edit ig nodes-us-east-1a
         
         # spec:
         #   machineType: c5.large
         ```

         - ![image-20230125015618319](files/img/image-20230125015618319.png)
         - ![image-20230125015746294](files/img/image-20230125015746294.png)

     - 인스턴스 그룹 삭제

       - ```bash
         kops delete ig nodes-us-east-1c
         ```

         - ![image-20230125020703289](files/img/image-20230125020703289.png)

       - ```bash
         kops get ig
         kops get instances
         ```
       
         - ![image-20230125020929069](files/img/image-20230125020929069.png)

     - kops 클러스터 업데이트

       - ```bash
         kops update cluster --yes && echo && sleep 5 && kops rolling-update cluster --yes
         ```

         - ![image-20230125021524714](files/img/image-20230125021524714.png)
         - ![image-20230125021732127](files/img/image-20230125021732127.png)

2. 기본 정보 확인

   - 노드 인스턴스 타입 확인

     - ```bash
       kubectl describe nodes | grep "node.kubernetes.io/instance-type"
       ```
       
       - ![image-20230125021800678](files/img/image-20230125021800678.png)

   - 노드 상세 정보 확인 : 노드 상세 정보의 Allocatable 에 노드별 최대 생성 가능한 pods 정보 확인 - 각각 마스터 노드, 워커 노드

     - ```bash
       kubectl describe node | grep Allocatable: -A6
       ```
     
       - ![image-20230125022028007](files/img/image-20230125022028007.png)

3. 파드 30개 배포 후 확인

   - 파드 배포 및 갯수 늘리기

     - ```bash
       cat ~/pkos/2/nginx-dp.yaml | yh
       ```

       - ![image-20230125022123355](files/img/image-20230125022123355.png)

     - ```bash
       kubectl apply -f ~/pkos/2/nginx-dp.yaml
       ```

       - ![image-20230125022152386](files/img/image-20230125022152386.png)

     - ```bash
       kubectl scale deployment nginx-deployment --replicas=30
       ```

       - ![image-20230125022205581](files/img/image-20230125022205581.png)
       - ![image-20230125022238808](files/img/image-20230125022238808.png)

   - 파드 배포 확인

     - ```bash
       kubectl get pod
       ```

       - ![image-20230125022305401](files/img/image-20230125022305401.png)

     - ```bash
       kubectl get pod | grep Pending | wc -l
       ```

       - ![image-20230125022336840](files/img/image-20230125022336840.png)

     - ```bash
       kubectl get replicasets
       ```

       - ![image-20230125022427123](files/img/image-20230125022427123.png)

     - ```bash
       kubectl events | tail -n 2
       ```

       - ![image-20230125022438132](files/img/image-20230125022438132.png)

   - 노드 정보 확인 : 현재 워커 노드 29개 최대 파드 생성할 수 있으지만, CPU 리소스 부족으로 22개가 동작

     - ```bash
       kubectl describe node
       ```

       - ![image-20230125022503114](files/img/image-20230125022503114.png)
       - ![image-20230125022508276](files/img/image-20230125022508276.png)

   - LimitRanges 기본 정책 확인 : 컨테이너는 기본적으로 0.1CPU(=100m vcpu)를 최소 보장(개런티)

     - ```bash
       kubectl describe limitranges
       ```
     
       - ![image-20230125022527527](files/img/image-20230125022527527.png)

4. limitranges 정책 삭제 후 파드 30개 배포 후 확인

   - 파드 갯수 0으로 줄이기

     - ```bash
       kubectl scale deployment nginx-deployment --replicas=0
       ```

       - ![image-20230125022554071](files/img/image-20230125022554071.png)
       - ![image-20230125022645793](files/img/image-20230125022645793.png)

   - LimitRanges 기본 정책 삭제

     - ```bash
       kubectl delete limitranges limits
       kubectl get limitranges
       ```

       - ![image-20230125022626973](files/img/image-20230125022626973.png)

   - 파드 갯수 30으로 늘리기

     - ```bash
       kubectl scale deployment nginx-deployment --replicas=30
       ```

       - ![image-20230125022705746](files/img/image-20230125022705746.png)
       - ![image-20230125022751416](files/img/image-20230125022751416.png)

   - 이전 17개 보다는 8개 많은 25개 파드가 배포됨 >> 하지만 아직도 5대의 파드가 배치되지 않음

     - ```bash
       kubectl get replicasets
       ```

       - ![image-20230125022814828](files/img/image-20230125022814828.png)

   - 노드 정보 확인 : 현재 워커 노드 29개 최대 파드 생성할 수 있으며, 29개 파드 동작 중

     - ```bash
       kubectl describe node
       ```
     
       - ![image-20230125022951063](files/img/image-20230125022951063.png)
       - ![image-20230125023057907](files/img/image-20230125023057907.png)
       - ![image-20230125023104415](files/img/image-20230125023104415.png)

5. kOps 클러스터와 인스턴스 그룹에 파라미터 수정

   - 파드 갯수 0으로 줄이기

     - ```bash
       kubectl scale deployment nginx-deployment --replicas=0
       ```

       - ![image-20230125023133759](files/img/image-20230125023133759.png)
       - ![image-20230125023204195](files/img/image-20230125023204195.png)

   - 수정 전 env 정보 확인 : WARM_PREFIX_TARGET은 기본값이 1로 이미 되어 있음

     - ```bash
       kubectl describe ds -n kube-system aws-node | grep ADDITIONAL_ENI_TAGS: -A22
       ```

       - ![image-20230125023214193](files/img/image-20230125023214193.png)

   - 적용 전 노드(인스턴스)의 External-IP 확인

     - ```bash
       kubectl get node -owide
       ```

       - ![image-20230125023228514](files/img/image-20230125023228514.png)

   - kOps 클러스터와 인스턴스 룹에 파라미터 수정 : 노드에 kubelet에 maxPods 110개로 수정, Prefix Assign 활성화

     - ```bash
       kops edit cluster
       
       # ...
       #   kubelet:
       #     anonymousAuth: false
       #     maxPods: 110
       # ...
       #   networking:
       #     amazonvpc:
       #       env:
       #       - name: ENABLE_PREFIX_DELEGATION
       #         value: "true"
       # ...
       ```

       - ![image-20230125023351245](files/img/image-20230125023351245.png)
       - ![image-20230125023533720](files/img/image-20230125023533720.png)
       - ![image-20230125023549963](files/img/image-20230125023549963.png)

   - 적용을 위해서 반드시 노드 롤링 업데이트 필요 : 삭제 -> 재생성, 이 과정에서 matser node의 IP(Public, Private)가 자동으로 변경됨 >> 10분 정도 소요

     - ```bash
       kops update cluster --yes && echo && sleep 5 && kops rolling-update cluster --yes
       ```

       - ![image-20230125024238722](files/img/image-20230125024238722.png)
       - ![image-20230125030729443](files/img/image-20230125030729443.png)

   - 롤링 업데이트 완료 후 노드 정보 확인 : External-IP 변경 확인, 참고로 맨 아래 워커노드는 현재 삭제된 EC2로 잠시 시간이 지나면 워커 노드 목록에서 삭제됨

     - ```bash
       kubectl get node -owide
       ```

       - ![image-20230125030754038](files/img/image-20230125030754038.png)

   - 변경 정보 반영된 env 정보 확인

     - ```bash
       kubectl describe ds -n kube-system aws-node | grep ADDITIONAL_ENI_TAGS: -A22
       ```

       - ![image-20230125030840567](files/img/image-20230125030840567.png)

     - ```bash
       kubectl describe daemonsets.apps -n kube-system aws-node | egrep 'ENABLE_PREFIX_DELEGATION|WARM_PREFIX_TARGET'
       ```

       - ![image-20230125030857532](files/img/image-20230125030857532.png)

   - 노드 상세 정보 확인 : 노드 상세 정보의 Allocatable에 노드별 최대 생성 가능한 pods 정보 확인 - 각각 마스터 노드, 워커 노드

     - ```bash
       kubectl describe node | grep Allocatable: -A6
       ```

       - ![image-20230125030917098](files/img/image-20230125030917098.png)

   - Route53에 A 레코드 값을 신규 Master EC2의 IP(Public, Private)로 자동으로 변경됨

     - 변경전
       - ![image-20230125011523575](files/img/image-20230125011523575.png)
     - 변경후
       - ![image-20230125030952459](files/img/image-20230125030952459.png)

6. 파드 110 생성 후 확인

   - 파드 갯수 110으로 늘이기

     - ```bash
       kubectl scale deployment nginx-deployment --replicas=110
       ```

       - ![image-20230125031023167](files/img/image-20230125031023167.png)
       - ![image-20230125031215091](files/img/image-20230125031215091.png)
   
     - ```bash
       kubectl get replicasets
       ```
   
       - ![image-20230125031206348](files/img/image-20230125031206348.png)
   
   - 노드 정보 확인 : 현재 워커 노드에 기존 6개 파드 + 106개 nginx 파드 = 총 110개 파드 배포
   
     - ```bash
       kubectl describe node
       ```
   
       - ![image-20230125031241405](files/img/image-20230125031241405.png)
       - ![image-20230125031411502](files/img/image-20230125031411502.png)
       - ![image-20230125031426465](files/img/image-20230125031426465.png)
   
   - 삭제
   
     - ```bash
       kubectl delete deploy nginx-deployment
       ```
     
       - ![image-20230125031505071](files/img/image-20230125031505071.png)



### 2.6 서비스 소개

#### 2.6.1 개념

##### 1. 서비스 종류

- ClusterIP 타입
  - ![출처: PKOS 스터디(서비스 종류)](files/img/image-20230124232722560.png)

- NodePort 타입
  - ![출처: PKOS 스터디(서비스 종류)](files/img/image-20230124232732447.png)

- LoadBalancer 타입(기본 모드) : NLB 인스턴스 유형
  - ![출처: PKOS 스터디(서비스 종류)](files/img/image-20230124232748605.png)

- Service (LoadBalancer Controller) : AWS Load Balancer Controller + NLB IP 모드 동작
  - ![출처: PKOS 스터디(서비스 종류)](files/img/image-20230124232759591.png)


##### 2. NLB(Network Load Balancer) 모드 전체 정리

- 인스턴스 유형
  1. externalTrafficPolicy
     - ClusterIP => 2번 분산 및 SNAT으로 Client IP 확인 불가능 <- LoadBalancer 타입 (기본 모드) 동장
  2. externalTrafficPolicy
     - Local => 1번 분산 및 ClientIP 유지, 워커 노드의 iptables 사용함
- - 인스턴스 유형 상세 설명
    - 요약 : 외부 클라이언트가 '로드밸런서' 접속 시 부하분산 되어 노드 도달 후 iptables 룰로 목적지 파드와 통신됨
      - ![출처: PKOS 스터디(NLB 모드 전체 정리)](files/img/image-20230124180624756.png)
      - ![출처: PKOS 스터디(NLB 모드 전체 정리)](files/img/image-20230124181011548.png)
    - 노드는 외부에 공개되지 않고 로드밸런서만 외부에 공개되어, 외부 클라이언트는 로드밸런서에 접속을 할 뿐 내부 노드의 정보를 알 수 없음
    - 로드밸런서가 부하분산하여 파드가 존재하는 노드들에게 전달
      - iptalbes 룰에서는 자신의 노드에 있는 파드만 연결(`externalTrafficPolicy: local`)
    - DNAT(Destination NAT) 2번 동작 : 
      - 첫번째 : 로드밸런서 접속 후 빠져 나갈때
      - 두번째 : 노드의 iptables 룰에서 파드 IP 전달 시
    - 외부 클라이언트 IP 보존(유지) : 
      - AWS NLB는 타겟이 인스턴스일 경우 클라이언트 IP를 유지
      - iptables 룰 경우도 `externalTrafficPolicy`로 클라이언트 IP를 보존
    - 부하분산 최적화 : 
      - 노드에 파드가 없을 경우 '로드밸런서'에서 노드에 헬스 체크(상태 검사)가 실패하여 해당 노드로는 외부 요청 트래픽을 전달하지 않음
        - ![출처: PKOS 스터디(NLB 모드 전체 정리)](files/img/image-20230124181257403.png)
        - ![출처: PKOS 스터디(NLB 모드 전체 정리)](files/img/image-20230124181302209.png)



- IP 유형 => 반드시 AWS LoadBalancer 컨트롤러 파드 및 정책 설정이 필요함

- - https://www.notion.so/gasidaseo/AWS-NLB-Client-IP-Proxy-protocol-57827e2c83fc474992b37e65db81f669

  1. Proxy Protocol v2 비활성화
     - NLB에서 바로 파드로 인입, 단 ClientIP가 NLB로 SNAT 되어 ClientIP 확인 불가능
  2. Proxy Protocol v2 활성화
     - NLB에서 바로 파드로 인입 및 ClientIP 확인 가능
       - 단, PPv2를 애플리케이션이 인지할 수 있게 설정 필요

#### 2.6.2 실습

##### 1. EC2 instance profiles 설정 및 AWS Load Balancer 배포

- 기본 EC2 profile 권한 확인

  - ```bash
    aws elbv2 describe-load-balancers
    ```
    
    - ![image-20230124233020414](files/img/image-20230124233020414.png)



- 마스터/워커 노드에 EC2 IAM Role에 Policy (AWSLoadBalancerControllerIAMPolicy) 추가

  - IAM Policy 정책 생성

    - ```bash
      curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.5/docs/install/iam_policy.json
      aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
      ```
      
      - ![image-20230124233204655](files/img/image-20230124233204655.png)




- 계정 Account ID 변수 지정

  - ```bash
    ACCOUNT_ID=`aws sts get-caller-identity --query 'Account' --output text`
    echo $ACCOUNT_ID
    ```
    
    - ![image-20230124233224965](files/img/image-20230124233224965.png)



- 생성된 IAM Policy Arn 확인

  - ```bash
    aws iam list-policies --scope Local
    ```
  
    - ![image-20230124233253778](files/img/image-20230124233253778.png)
    - ![image-20230124233301083](files/img/image-20230124233301083.png)
    
  - ```bash
    aws iam get-policy --policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy
    ```
    
    - ![image-20230124233314304](files/img/image-20230124233314304.png)
    
  - ```bash
    aws iam get-policy --policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy --query 'Policy.Arn'
    ```
    
    - ![image-20230124233324711](files/img/image-20230124233324711.png)



- EC2 Instance profiles 이름 확인

  - ```bash
    aws iam list-instance-profiles
    ```
  
    - ![image-20230124233400207](files/img/image-20230124233400207.png)
    - ![image-20230124233406507](files/img/image-20230124233406507.png)
    
  - ```bash
    aws iam list-instance-profiles --query 'InstanceProfiles[*].InstanceProfileName'
    ```
    
    - ![image-20230124233418150](files/img/image-20230124233418150.png)



- EC2 instance profiles에 IAM Policy 추가(attach)

  - ```bash
    aws iam attach-role-policy --policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy --role-name masters.$KOPS_CLUSTER_NAME
    ```
  
  - ```bash
    aws iam attach-role-policy --policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy --role-name nodes.$KOPS_CLUSTER_NAME
    ```
    
    - ![image-20230124233445749](files/img/image-20230124233445749.png)

- kOps 클러스터 편집 : 아래 내용 추가

  - ```bash
    ## 인증서 구성을 웹훅에 삽입할 수 있도록 cert-manager를 설치합니다. 
    ## Cert-manager는 쿠버네티스 클러스터 내에서 TLS인증서를 자동으로 프로비저닝 및 관리하는 오픈 소스입니다
    kops edit cluster --name ${KOPS_CLUSTER_NAME}
    ```
  
    - ![image-20230124233508702](files/img/image-20230124233508702.png)
    
  - ```bash
    # ...
    # spec:
    #   certManager:
    #     enabled: true
    #   awsLoadBalancerController:
    #     enabled: true
    # ...
    ```
    
    - ![image-20230124233554929](files/img/image-20230124233554929.png)
    - ![image-20230124233821587](files/img/image-20230124233821587.png)



- 업데이트 적용 : 적용이 안될 경우 한번 더 아래 명령 실행

  - ```bash
    kops update cluster --yes && echo && sleep 5 && kops rolling-update cluster
    ```
    
    - ![image-20230124233846626](files/img/image-20230124233846626.png)
    - ![image-20230124233854185](files/img/image-20230124233854185.png)



- aws-load-balancer-controller 파드 확인 : 서비스 정상 상태로 변경까지 대략 2분 30초 정도 소요

  - ```bash
    watch kubectl get pod -A
    ```
    
    - ![image-20230124234256683](files/img/image-20230124234256683.png)



- 버전 확인 : v2.4.3

  - ```bash
    kubectl describe deploy -n kube-system aws-load-balancer-controller | grep Image | cut -d "/" -f 2
    ```
    
    - ![image-20230124234315515](files/img/image-20230124234315515.png)

- crds 확인

  - ```bash
    kubectl get crd
    ```
    
    - ![image-20230124234325620](files/img/image-20230124234325620.png)



##### 2. 서비스/파드 배포 테스트 with NLB

- https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/guide/service/nlb/

- - 작업용 EC2 - 디플로이먼트 & 서비스 생성

    - ```bash
      cat ~/pkos/2/echo-service-nlb.yaml | yh
      kubectl apply -f ~/pkos/2/echo-service-nlb.yaml
      ```
      
      - ![image-20230124234426490](files/img/image-20230124234426490.png)
      - ![image-20230124234431282](files/img/image-20230124234431282.png)
  
  
  - 확인
  
    - ```bash
      kubectl get deploy,pod
      ```
  
      - ![image-20230124234444804](files/img/image-20230124234444804.png)
      
    - ```bash
      kubectl get svc,ep,targetgroupbindings
      ```
  
      - ![image-20230124234454098](files/img/image-20230124234454098.png)
      
    - ```bash
      kubectl get targetgroupbindings -o json | jq
      ```
      
      - ![image-20230124234516698](files/img/image-20230124234516698.png)
      - ![image-20230124234505223](files/img/image-20230124234505223.png)
  
  - AWS ELB(NLB) 정보 확인
  
    - ```bash
      aws elbv2 describe-load-balancers | jq
      ```
  
      - ![image-20230124234536003](files/img/image-20230124234536003.png)
      - ![image-20230124234527906](files/img/image-20230124234527906.png)
      
    - ```bash
      aws elbv2 describe-load-balancers --query 'LoadBalancers[*].State.Code' --output text
      ```
      
      - ![image-20230124234603976](files/img/image-20230124234603976.png)
      - ![image-20230124234708948](files/img/image-20230124234708948.png)
      - ![image-20230124235148839](files/img/image-20230124235148839.png)
      - ![image-20230124235256893](files/img/image-20230124235256893.png)
  
  - 웹 접속 주소 확인
  
    - ```bash
      kubectl get svc svc-nlb-ip-type -o jsonpath={.status.loadBalancer.ingress[0].hostname} | awk '{ print "Pod Web URL = http://"$1 }'
      ```
      
      - ![image-20230124234642288](files/img/image-20230124234642288.png)
      - ![image-20230124234717257](files/img/image-20230124234717257.png)
      - ![image-20230124234725942](files/img/image-20230124234725942.png)
  
  - 파드 로깅 모니터링
  
    - ```bash
      kubectl mtail app=deploy-websrv
      ```
      
      - ![image-20230124234742197](files/img/image-20230124234742197.png)
      - ![image-20230124234753012](files/img/image-20230124234753012.png)
  
  - 분산 접속 확인
  
    - ```bash
      NLB=$(kubectl get svc svc-nlb-ip-type -o jsonpath={.status.loadBalancer.ingress[0].hostname})
      echo $NLB
      ```
      
      - ![image-20230124234836777](files/img/image-20230124234836777.png)
      
    - ```bash
      curl -s $NLB
      ```
  
      - ![image-20230124234854717](files/img/image-20230124234854717.png)
      - ![image-20230124234907913](files/img/image-20230124234907913.png)
      
    - ```bash
      for i in {1..100}; do curl -s $NLB | grep Hostname ; done | sort | uniq -c | sort -nr
      ```
      
      - ![image-20230124234920058](files/img/image-20230124234920058.png)
  
  - 지속적인 접속 시도 : 아래 상세 동작 확인 시 유용(패킷 덤프 등)
  
    - ```bash
      while true; do curl -s --connect-timeout 1 $NLB | egrep 'Hostname|client_address'; echo "----------" ; date "+%Y-%m-%d %H:%M:%S" ; sleep 1; done
      ```
      
      - ![image-20230124235034547](files/img/image-20230124235034547.png)
  



- AWS NLB의 대상 그룹 확인 : IP를 확인해보자

- 파드 2개 -> 1개 -> 3개 설정 시 동작 : auto discovery <- 어떻게 가능할까?

  - 작업용 EC2 - 파드 1개 설정

    - ```bash
      kubectl scale deployment deploy-echo --replicas=1
      ```
      
      - ![image-20230124235344555](files/img/image-20230124235344555.png)
      - ![image-20230124235412718](files/img/image-20230124235412718.png)
  
  - 확인

    - ```bash
      kubectl get deploy,pod,svc,ep
      ```
  
      - ![image-20230124235424762](files/img/image-20230124235424762.png)
      
    - ```bash
      for i in {1..100}; do curl -s $NLB | grep Hostname ; done | sort | uniq -c | sort -nr
      ```
      
      - ![image-20230124235501458](files/img/image-20230124235501458.png)
  
  - 작업용 EC2 - 파드 3개 설정

    - ```bash
      kubectl scale deployment deploy-echo --replicas=3
      ```
      
      - ![image-20230124235530432](files/img/image-20230124235530432.png)
      - ![image-20230124235905962](files/img/image-20230124235905962.png)
  
  - 확인
  
    - ```bash
      kubectl get deploy,pod,svc,ep
      ```
  
      - ![image-20230124235918658](files/img/image-20230124235918658.png)
      
    - ```bash
      for i in {1..100}; do curl -s $NLB | grep Hostname ; done | sort | uniq -c | sort -nr
      ```
      
      - ![image-20230124235929770](files/img/image-20230124235929770.png)
  
  - [AWS LB Ctrl] 클러스터 롤 바인딩 정보 확인
  
    - ```bash
      kubectl describe clusterrolebindings.rbac.authorization.k8s.io aws-load-balancer-controller-rolebinding
      ```
      
      - ![image-20230124235945715](files/img/image-20230124235945715.png)
  
  - [AWS LB Ctrl] 클러스터 롤 확인
  
    - ```bash
      kubectl describe clusterroles.rbac.authorization.k8s.io aws-load-balancer-controller-role
      ```
      
      - ![image-20230125000007377](files/img/image-20230125000007377.png)



##### 3. 실습 리소스 삭제

- ```bash
  kubectl delete deploy deploy-echo; kubectl delete svc svc-nlb-ip-type
  ```
  
  - ![image-20230125000110480](files/img/image-20230125000110480.png)



### 2.7 ExternalDNS

#### 2.7.1 개념

##### 1. ExternalDNS 소개

- k8s 서비스/인그레스 생성 시 도메인을 설정하면, AWS(Route 53), Azure(DNS), GCP(Cloud DNS)에 A 레코드(TXT 레코드)로 자동 생성/삭제

- ![출처: PKOS 스터디(ExternalDNS)](files/img/image-20230124182939941.png)
  - https://edgehog.blog/a-self-hosted-external-dns-resolver-for-kubernetes-111a27d6fc2c



#### 2.7.2 실습

##### 1. 설치

- 모니터링

  - ```bash
    watch -d kubectl get pod -A
    ```
    
    - ![image-20230125000155859](files/img/image-20230125000155859.png)

- dns 관련 파드 갯수 줄이기(2개)

  - ```bash
    kubectl delete deploy -n kube-system coredns-autoscaler
    ```

  - ```bash
    kubectl scale deploy -n kube-system coredns --replicas 1
    ```
    
    - ![image-20230125000224814](files/img/image-20230125000224814.png)

- 정책 생성 -> 마스터/워커 노드에 정책 연결

  - ```bash
    curl -s -O https://s3.ap-northeast-2.amazonaws.com/cloudformation.cloudneta.net/AKOS/externaldns/externaldns-aws-r53-policy.json
    ```

  - ```bash
    aws iam create-policy --policy-name AllowExternalDNSUpdates --policy-document file://externaldns-aws-r53-policy.json
    ```
    
    - ![image-20230125000328069](files/img/image-20230125000328069.png)

- example : arn:aws:iam::XXXXXXXXXXXX:policy/AllowExternalDNSUpdates

  - ```bash
    aws iam list-policies --query 'Policies[?PolicyName==`AllowExternalDNSUpdates`].Arn' --output text
    ```

  - ```bash
    export POLICY_ARN=$(aws iam list-policies --query 'Policies[?PolicyName==`AllowExternalDNSUpdates`].Arn' --output text)
    ```
    
    - ![image-20230125000347295](files/img/image-20230125000347295.png)

- EC2 instance profiles에 IAM Policy 추가(attach)

  - ```bash
    aws iam attach-role-policy --policy-arn $POLICY_ARN --role-name masters.$KOPS_CLUSTER_NAME
    ```

  - ```bash
    aws iam attach-role-policy --policy-arn $POLICY_ARN --role-name nodes.$KOPS_CLUSTER_NAME
    ```
    
    - ![image-20230125000403559](files/img/image-20230125000403559.png)
    - ![image-20230125000511388](files/img/image-20230125000511388.png)
    - ![image-20230125000529818](files/img/image-20230125000529818.png)

- 설치

  - ```bash
    kops edit cluster
    # ...
    # spec:
    #   externalDns:
    #     provider: external-dns
    # ...
    ```
    
    - ![image-20230125000548194](files/img/image-20230125000548194.png)
    - ![image-20230125000659427](files/img/image-20230125000659427.png)
    - ![image-20230125000706698](files/img/image-20230125000706698.png)

- 업데이트 적용

  - ```bash
    kops update cluster --yes && echo && sleep 5 && kops rolling-update cluster
    ```
    
    - ![image-20230125000723747](files/img/image-20230125000723747.png)
    - ![image-20230125000730540](files/img/image-20230125000730540.png)
    - ![image-20230125000821309](files/img/image-20230125000821309.png)

- 버전 확인 : v0.12.2

  - ```bash
    kubectl describe deploy -n kube-system external-dns | grep Image | cut -d "/" -f 3
    ```
    
    - ![image-20230125000832738](files/img/image-20230125000832738.png)

- externalDNS 컨트롤러 파드 확인

  - ```bash
    kubectl get pod -n kube-system -l k8s-app=external-dns
    ```
    
    - ![image-20230125000846782](files/img/image-20230125000846782.png)



##### 2. 서비스(NLB)/파드 배포 시 ExternalDNS 설정해보기

- ExternalDNS 로그 모니터링

  - ```bash
    kubectl logs -n kube-system -f $(kubectl get po -n kube-system | egrep -o 'external-dns[A-Za-z0-9-]+')
    ```
    
    - ![image-20230125000945932](files/img/image-20230125000945932.png)



- 서비스/파드 배포

  - ```bash
    kubectl apply -f ~/pkos/2/echo-service-nlb.yaml
    ```
    
    - ![image-20230125001049609](files/img/image-20230125001049609.png)



- 확인

  - ```bash
    kubectl get deploy,pod
    ```

    - ![image-20230125001123945](files/img/image-20230125001123945.png)
    
  - ```bash
    kubectl get svc,ep,targetgroupbindings
    ```
    
    - ![image-20230125001134187](files/img/image-20230125001134187.png)

- 각자 자신의 도메인 정보 입력

  - ```bash
    # MyDOMAIN1=<각자 자신의 nginx 도메인 지정>
    MyDOMAIN1=nginx.learn-dc.link
    echo $MyDOMAIN1
    ```
    
    - ![image-20230125001154194](files/img/image-20230125001154194.png)
    
  - ```bash
    kubectl annotate service svc-nlb-ip-type "external-dns.alpha.kubernetes.io/hostname=$MyDOMAIN1."
    ```
  
    - ![image-20230125001208991](files/img/image-20230125001208991.png)
    - ![image-20230125001220017](files/img/image-20230125001220017.png)
    - ![image-20230125001319340](files/img/image-20230125001319340.png)
    
  - ```bash
    kubectl describe svc svc-nlb-ip-type | grep Annotations: -A5
    ```
    
    - ![image-20230125001337307](files/img/image-20230125001337307.png)



- 확인

  - ```bash
    dig +short $MyDOMAIN1
    ```
    
    - ![image-20230125001347375](files/img/image-20230125001347375.png)



- 분산 접속 확인

  - ```bash
    curl -s $MyDOMAIN1
    ```
  
    - ![image-20230125001358380](files/img/image-20230125001358380.png)
    
  - ```bash
    for i in {1..100}; do curl -s $MyDOMAIN1 | grep Hostname ; done | sort | uniq -c | sort -nr
    ```
    
    - ![image-20230125001409486](files/img/image-20230125001409486.png)



- 지속적입 접속 시도 : 아래 상세 동작 확인 시 유용(패킷 덤프 등)

  - ```bash
    while true; do curl -s --connect-timeout 1 $MyDOMAIN1 | egrep 'Hostname|client_address'; echo "----------" ; date "+%Y-%m-%d %H:%M:%S" ; sleep 1; done
    ```
    
    - ![image-20230125001431924](files/img/image-20230125001431924.png)

- AWS Route53 A 레코드 확인

  - ![image-20230125001319340](files/img/image-20230125001319340.png)




##### 3. 실습 리소스 삭제

- 과제3 진행 후 삭제

  - ```bash
    kubectl delete deploy deploy-echo; kubectl delete svc svc-nlb-ip-type
    ```

    - ![image-20230125003015192](files/img/image-20230125003015192.png)
    - A 레코드 자동 삭제 확인
      - ![image-20230125003051784](files/img/image-20230125003051784.png)




#### 2.7.3 과제3

##### 1. 과제 내용

목표 : 서비스(NLB)/파드 배포 시 ExternalDNS 설정해서, 각자 자신의 도메인으로 NLB를 통해 애플리케이션(파드)로 접속해보고 관련 스샷을 올려주세요

- 퍼블릭 도메인이 없는 멤버분들은, ExternalDNS 제외하고 NLB 도메인으로 접속한 결과를 올려주세요



##### 2. 과제 수행내용

- 분산 접속 확인

  - ```bash
    echo $MyDOMAIN1
    for i in {1..100}; do curl -s $MyDOMAIN1 | grep Hostname ; done | sort | uniq -c | sort -nr
    ```
    
    - ![image-20230125001640255](files/img/image-20230125001640255.png)

- 웹에서 확인

  - ![image-20230125001755568](files/img/image-20230125001755568.png)
    
    

### 2.8 과제4

목표 : 아래 활용 기능 중 1개를 선택해서 실습 후 결과 내용을 올려주세요

#### 1. NLB에 TLS 적용하기

- 참고

  - https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/guide/use_cases/nlb_tls_termination/

- 실습

  - ACM(AWs Certificate Manager)에서 us-east-1(버지니아 북부) 인증서 발급

    - https://us-east-1.console.aws.amazon.com/acm/home?region=us-east-1#/certificates/list
    
  - 클러스터 네임 확인

    - ```bash
      echo $KOPS_CLUSTER_NAME
      ```
      
      - ![image-20230125002634771](files/img/image-20230125002634771.png)
  
  - 사용 리전(us-east-1)의 인증서 ARN 확인

    - ```bash
      aws acm list-certificates
      ```
      
      - ![image-20230125002652367](files/img/image-20230125002652367.png)
    
    - ```bash
      aws acm list-certificates --max-items 10
      ```
    
      - ![image-20230125002704923](files/img/image-20230125002704923.png)
    
    - ```bash
      aws acm list-certificates --query 'CertificateSummaryList[].CertificateArn[]' --output text
      ```
    
      - ![image-20230125002716630](files/img/image-20230125002716630.png)
    
    - ```bash
      CERT_ARN=`aws acm list-certificates --query 'CertificateSummaryList[].CertificateArn[]' --output text`
      echo $CERT_ARN
      ```
    
      - ![image-20230125002730692](files/img/image-20230125002730692.png)
    
  - 자신의 도메인 변수 지정
  
    - ```bash
      # MyDomain=<자신의 도메인>
      MyDomain=websrv.learn-dc.link
      echo $MyDomain
      ```
      
      - ![image-20230125002815056](files/img/image-20230125002815056.png)
  
  - 생성
  
    - ```bash
      cat <<EOF | kubectl create -f -
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: deploy-echo
      spec:
        replicas: 2
        selector:
          matchLabels:
            app: deploy-websrv
        template:
          metadata:
            labels:
              app: deploy-websrv
          spec:
            terminationGracePeriodSeconds: 0
            containers:
            - name: akos-websrv
              image: k8s.gcr.io/echoserver:1.5
              ports:
              - containerPort: 8080
      ---
      apiVersion: v1
      kind: Service
      metadata:
        name: svc-nlb-ip-type
        annotations:
          external-dns.alpha.kubernetes.io/hostname: "${MyDomain}"
          service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
          service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
          service.beta.kubernetes.io/aws-load-balancer-healthcheck-port: "8080"
          service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
          service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "https"
          service.beta.kubernetes.io/aws-load-balancer-ssl-cert: ${CERT_ARN}
          service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
      spec:
        ports:
          - port: 80
            targetPort: 8080
            protocol: TCP
            name: http
          - port: 443
            targetPort: 8080
            protocol: TCP
            name: https
        type: LoadBalancer
        loadBalancerClass: service.k8s.aws/nlb
        selector:
          app: deploy-websrv
      EOF
      ```
      
      - ![image-20230125003206684](files/img/image-20230125003206684.png)
      - ![image-20230125003157587](files/img/image-20230125003157587.png)
      - ![image-20230125003806251](files/img/image-20230125003806251.png)
      - ![image-20230125003821906](files/img/image-20230125003821906.png)
  
  - 모니터링
  
    - ```bash
      watch -d kubectl get svc,ep
      ```
      
      - ![image-20230125004106091](files/img/image-20230125004106091.png)
  
  - 확인
  
    - ```bash
      kubectl get svc,ep
      ```
      
      - ![image-20230125004119861](files/img/image-20230125004119861.png)
    
    - ```bash
      kubectl describe svc svc-nlb-ip-type
      ```
    
      - ![image-20230125004155932](files/img/image-20230125004155932.png)
    
    - ```bash
      kubectl describe svc svc-nlb-ip-type | grep Annotations: -A8
      ```
    
      - ![image-20230125004209659](files/img/image-20230125004209659.png)
  
  - (외부) 접속 테스트
  
    - ```bash
      # curl -s http://<접속 도메인 주소> | grep Hostname
      curl -s -k http://$MyDomain | grep Hostname
      ```
      
      - ![image-20230125003532869](files/img/image-20230125003532869.png)
    
    - ```bash
      # curl -s -k https://<접속 도메인 주소> | grep Hostname
      curl -s -k https://$MyDomain | grep Hostname
      ```
    
      - ![image-20230125004244583](files/img/image-20230125004244583.png)
      - ![image-20230125004056823](files/img/image-20230125004056823.png)
    
  - 삭제
  
    - ```bash
      kubectl delete deploy deploy-echo; kubectl delete svc svc-nlb-ip-type
      ```
      
      - ![image-20230125004316593](files/img/image-20230125004316593.png)
      - ![image-20230125004331849](files/img/image-20230125004331849.png)
      - ![image-20230125004344576](files/img/image-20230125004344576.png)
  
  

#### 2. NLB 대상타겟을 Instance mode로 설정해보기



#### 3. kOps dns-controller compatibility mode

- 참고
  - https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/kops-dns-controller.md



#### 4. NLB IP Target & Proxy Protocol v2 활성화 : NLB에서 바로 파드로 인입 및 CLient IP 확인 설정

- 참고
  - https://www.notion.so/57827e2c83fc474992b37e65db81f669#a599aa7f26774fecb73bbd1f42c5e52a
  - https://hub.docker.com/r/gasida/httpd/tags
  - ![출처: PKOS 스터디(과제4)](files/img/image-20230124184009637.png)

#### 5. AWS kOps Private Cluster 설정하기

- 참고
  - https://kops.sigs.k8s.io/topology/
  - https://kops.sigs.k8s.io/operations/high_availability/#example-2-private-topology





## 3. 실습 완료 후 자원 삭제

- kOps 클러스터 삭제 & AWS CloudFormation 스택 삭제

  - ```bash
    kops delete cluster --yes && aws cloudformation delete-stack --stack-name mykops
    ```
    
    - ![image-20230125031542717](files/img/image-20230125031542717.png)
    - ![image-20230125031550359](files/img/image-20230125031550359.png)
    - ![image-20230125032026739](files/img/image-20230125032026739.png)

