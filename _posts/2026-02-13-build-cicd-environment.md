---
layout: post
title: "쿠버네티스 상에서 Github Actions로 CI/CD 환경 구축"
categories: engineer
tags: [Docker, knowledge, Engineer, build, problem]
published: true
---

오랜만에 올리는 포스팅이다.

비록 꾸준히 블로그에 올리겠다는 다짐은 지켜지지 않았으나, 꾸준히 뭔가를 하고 있는 것 같아 다행이다.

친해진 동료와 함께 프로젝트를 진행하기 위해 기초적인 인프라 작업을 내가 맡아서 진행하고 있었는데, 현재 정리해둔 내용을 올려보려고 한다.


# Github CI/CD의 기초 및 툴을 선택한 이유

## CI/CD란?

- 앱의 개발부터 배포까지 모든 단계를 자동화하여, 코드를 통합하고 검증하여 사용자에게 신속하고 안정적으로 서비스하는 파이프라인 구축 방법.
- CI(Continuous Integration)
    - 여러 사람의 코드 변경 사항을 공유 레포에 자동으로 통합하여, 앱을 빌드하고 다양한 수준의 테스트를 통해 변경 사항을 검증하는 과정.
- CD > 두 가지 경우를 모두 CD라고 부름
    - Continous Delivery
        - 테스트에 성공할 시 언제든지 배포할 준비를 마친 상태로 대기. 관리자의 승인에 따라 앱이 배포됨.
        - 누가 배포를 했는지, 점검 시간이나 배포 일정에 맞추기 용이함.
        - 배포 승인해주는 사람이 필요하며, 사람의 승인 시간이 필요해서 자동화보다는 느림.
    - Continuous Deployment
        - 테스트에 성공할 시 자동화된 파이프라인이 앱을 릴리즈.
        - 빠르게 배포가 가능하며, 사용자 반응을 실시간으로 보고 고칠 수 있음
        - 테스트 코드에서 거르지 못하는 버그가 있다면 서비스가 불안정해질 수 있고, 테스트 코드 짜는게 오래 걸림.
- 두 가지 방식의 CD방식이 있지만, 안정성과, 실무에서 더 많이 쓰일 Continuous Delivery 방식으로 접근.
- CI/CD환경에서의 선택지
    1. Cloud 환경(AWS/GCP 등)이나 외부 업체에 돈 내고 사용
        1. 스타트업이나 인프라 인력이 부족할 시 편하게 사용 가능하며, 전용 클라우드 자원을 써서 빌드 환경이 쾌적함.
        2. 하지만 사용량이 늘어나면 비용이 늘어나고, 코드가 외부 업체에서 빌드되어 보안의 문제가 있음.
    2. Jenkins
        1. 무료에, 커스터마이징이 뛰어나며, 자유도가 높음.
        2. 하지만 관리가 어렵고 ui가 불편하며, 자바 기반이라 무거움
    3. GitLab
        1. 코드 관리부터 이슈 트래킹, CI/CD까지 한 곳에서 사용하며, 도커 컨테이너 기반 빌드가 좋음
        2. 하지만 깃랩 자체가 꽤나 무겁고, 깃허브를 쓰면서 CI만 깃랩을 쓰는게 불편함
    4. Github Actions
        1. 깃허브를 쓴다면 편하게 접근 가능하며, 남들의 스크립트를 쉽게 가져올 수 있음.
        2. 퍼블릭 레포는 무료이지만, 프라이빗은 무료 초과 시 과금이 됨.
    - 이 중 깃허브 액션을 사용하기로 결정함. 무료에 구축이 편하고, YAML파일로 관리가 쉽고 최근에 높은 점유율을 차지하고 있다는 점을 보아 실무에서 자주 사용할 것으로 예상.

## Github Actions를 구축하기

### Github Actions의 기본 개념들

- Workflow
    - 자동화된 전체 프로세스. YAML파일로 작성되며, Github Repository의 .github/workflows 폴더 아래에 저장됨.
- Event
    - Workflow를 실행하는 특정 활동이나 규칙.
- Job
    - Step들로 구성된 하나의 행동 단위.다른 Job에 의존관계 혹은 병렬로 실행되기도 함.
- Step
    - 순차적으로 실행되는 Job보다 작은 프로세스의 단위. 명령을 내리거나 실행하는 단위임.
- Actions
    - 특정 작업을 수행하며 반복적인 코드의 양을 줄이는 미리 정의하고 재사용 가능한 작업 혹은 코드 집합.
- Runner
    - Github Action Runner가 설치된 머신으로, Workflow가 실행될 인스턴스.
- ARC
    - Github Actions팀의 self-hosted runner를 쿠버네티스 위에서 오토스케일링하며 관리해주는 오퍼레이터. 이를 통해 동시에 코드가 들어와도 빠르게 필요한 워커를 할당하여 처리 가능.
- ArgoCD
    - 깃 저장소의 상태 변경을 관측하고 있다가, 변경된 내용을 감지하여 배포를 도와주는 툴.
- 작업 흐름 순서
    - Trigger > Runner Allocation > Job Start > Post-Build > Clean up
    Local Computer > Github Shared Repository > Github Actions > ARC > Kaniko > Container Registry > ArgoCD > Real Deployment
    - Trigger - 코드를 수정하고 git push를 침.
    - Runner Allocation - ARC가 코드가 변경됨을 감지하고, 쿠버네티스 내부에 새로운 (일회용) pod를 생성.
    - Job Start
        - Check-Out - 깃허브 저장소에 있는 코드를 불러옴.
        - Build & Push(Kaniko) - Dockerfile을 읽고, 도커 데몬 없이 이미지 레이어를 만든 후에, 이것을 하나의 이미지로 스냅샷을 뜨고, 도커 허브에 전송.
    - Post-Build - 일반적으로는 쿠버네티스에 새 버전으로 업데이트 시키는 명령어를 사용하지만, ArgoCD를 사용하는 GitOps 방식 사용 시, 배포용 저장소의 이미지 태그를 v1에서 v2로 수정하여 commit & push 진행.
    - Clean up - 사용한 포드를 제거.

### Build Tools에 대한 고찰

|  | Kaniko | Buildkit | Buildah | DinD |
| --- | --- | --- | --- | --- |
| 데몬 요구 여부 | N | Y | N | Y |
| 권한 요구 | Rootless | Rootless | Rootless | Privileged |
| 빌드 속도 | Medium | Fast | Slow | Medium |
| 설정 난이도 | Medium | High | Medium | Low |
| 커뮤니티 활성도 | High | High | Medium | High |

위의 표만 본다면 BuildKit > Kaniko > Buildah > DinD 순으로 선호도가 높아보이지만, Buildkit의 경우 로컬에 캐시를 저장해두고 재사용해서 빠른것인데, 이를 위해 PVC(Persistent Volume Claims)를 설정해야하고, 데몬을 사용하지 않고 돌릴 수 있지만, 이를 위해 포드 시작할 때 `buildkitd`를 백그라운드에 띄우고, 소켓을 연결하고 빌드를 끝나면 다시 죽이는 과정이 필요하여, 설정이 상대적으로 복잡함. 따라서 설정도 상대적으로 쉬운 Kaniko를 선택!

### CD Tools에 대한 고찰

|  | ArgoCD | Flux | Spinnaker | Jenkins |
| --- | --- | --- | --- | --- |
| 방식 | GitOps | GitOps | Pipeline | Script |
| UI | Good | Bad | Medium | Medium |
| 설치 난이도 | Medium | Good | Very Bad | Bad |
| 리소스 소모 정도 | Medium | Low | Very High | High |

우선 젠킨스는 깃허브 액션을 쓰기로 해서 탈락. 

Spinnaker 방식은 외부에서 클러스터 자격 증명을 가지고 접속해서 보안에 취약한 방식에, 필요한 서비스가 많고 무거워 탈락.

ArgoCD와 Flux는 같은 GitOps 방식을 사용하지만, Flux에 비해 편하게 토폴로지 모니터링이 가능한 ArgoCD를 선택. 

설치할 툴 및 설치 과정

1. Cert-Manager > Github와의 통신을 위해서 제일 먼저 설치
    1. 호환되는 버전을 찾기 위해 공식 홈페이지에 들어갔고, 쿠버네티스 버전을 확인.
    `kubectl version` / `kubeclt get nodes`
    결과 클라이언트는 1.35.0, 서버는 1.30.14, 워커 노드는 1.29.15의 버전 차이가 발생.
    쿠버네티스는 공식적으로 +- 1까지만 호환성을 보장하기 때문에 클라이언트 버전을 낮춤.
    2. 1.29부터 1.33까지 보장하는 1.18버전으로 설치.
    `kubectl apply -f [https://github.com/cert-manager/cert-manager/releases/download/v1.18.0/cert-manager.yaml](https://github.com/cert-manager/cert-manager/releases/download/v1.18.0/cert-manager.yaml)` \
2. ARC
    1. 보안을 위해 깃허브에서 토큰 발급
    2. 쿠버네티스 상에 네임스페이스 생성 및 secret에 토큰 저장
    `kubectl create ns actions-runner-system`
    `kubectl create secret generic controller-manager -n actions-runner-system —-from-literal=github_tokern’토큰’`
    3. helm 저장소 등록
    `helm repo add actions-runner-controller [https://actions-runner-controller.github.io/actions-runner-controller](https://actions-runner-controller.github.io/actions-runner-controller)
    helm repo update`
    4. 설치에 자꾸 에러가 뜸! K8s Worker인 windows에서  방화벽 문제가 있어 해결.
    마스터 노드와의 통신 허용\
    `New-NetFirewallRule -DisplayName "K8s Master Allow" -Direction Inbound -RemoteAddress x.x.x.x -Action Allow`
    \NodePort 개방\
    `New-NetFirewallRule -DisplayName "K8s VXLAN UDP" -Direction Inbound -Protocol UDP -LocalPort 8472 -Action Allow
    New-NetFirewallRule -DisplayName "K8s Kubelet TCP" -Direction Inbound -Protocol TCP -LocalPort 10250 -Action Allow
    New-NetFirewallRule -DisplayName "K8s NodePort TCP" -Direction Inbound -Protocol TCP -LocalPort 30000-32767 -Action Allow`
    5. 다시 설치\
    `helm install actions-runner-controller actions-runner-controller/actions-runner-controller \
    --namespace actions-runner-system \
    --create-namespace \
    --set authSecret.create=false \
    --set authSecret.name=controller-manager`
    6. 단..이렇게 하니까 윈도우에서 자꾸 실행되며 방화벽 문제가 생김. 그래서 찾다보니 윈도우즈에서 wsl2로 돌리는 노드는 워커로만 써야한다고 함. cert-manager도 linux에서만 돌아가는데 wsl2를 돌리다보니 리눅스로 판단하지만, 실제로는 윈도우즈를 거친 후에 linux에서 실행되는 것과 같기때문에, 에러가 발생. 따라서 윈도우 노드에 windows라고 라벨을 넣어주려 했지만, 쿠버네티스 설정 상 실패. 따라서 필요한 잡들만 마스터에서 작동하도록 설정.
    
    `cert-manager` 웹훅을 마스터에서 작동.\
    `kubectl patch deployment -n cert-manager cert-manager-webhook \
    --patch '{"spec": {"template": {"spec": {"nodeSelector": {"[node-role.kubernetes.io/control-plane](http://node-role.kubernetes.io/control-plane)": ""}}}}}'
    
    ARC도 마스터에서 작동하게 upgrade\
    helm upgrade --install actions-runner-controller actions-runner-controller/actions-runner-controller \
    --namespace actions-runner-system \
    --create-namespace \
    --set authSecret.create=false \
    --set authSecret.name=controller-manager \
    --set nodeSelector."node-role\.kubernetes\.io/control-plane"=""`
    7. 이후에 깃허브 레포 테스트용으로 하나 만들어서 연동하고 테스트 성공\
3. ArgoCD
    1. helm 저장소 등록\
    `helm repo add argo https://argoproj.github.io/argo-helm
    helm repo update`
    2. 버전 확인\
    `helm search repo argo/argo-cd --versions`
    3. 마스터 노드에서 작동하도록 설치\
    `helm upgrade --install argocd argo/argo-cd \
    --namespace argocd \
    --create-namespace \
    --version 8.2.7 \
    --set global.nodeSelector."node-role\.kubernetes\.io/control-plane"=""`
    4. 설치 완료 후 유저 추가 및 권한 설정\
    `KUBE_EDITOR="nano" kubectl edit configmap argocd-cm -n argocd`
    `data:` 아래에 아래와 같이 추가
    `accounts.계정명: apiKey, login
    accounts.계정명.enabled: ‘true’`
    
    권한추가\
    `KUBE_EDITOR="nano" kubectl edit configmap argocd-rbac-cm -n argocd`
    `data:
      policy.csv: |
        g, ronny, role:admin
        g, alledeli, role:admin`
    5. 클라우드 플레어 설정
    터널이름 : argocd
    하위 도메인 : argocd
    도메인 : self.ronny.com
    서비스 형식: https
    url : argocd-server.argocd.svc
    
    추가 설정
    TLS 확인 없음 체크(argocd는 가짜 인증서로 웹사이트를 구동하기 때문)
    6. 외부에서 접속 가능하도록 yaml 파일 생성 및 쿠버네티스 상에 설치
        - yaml 파일 내용
            
            ```yaml
            apiVersion: apps/v1
            
            kind: Deployment
            
            metadata:
            
              name: cloudflared
            
              namespace: infra
            
              labels:
            
                app: cloudflared
            
            spec:
            
              replicas: 1
            
              selector:
            
                matchLabels:
            
                  app: cloudflared
            
              template:
            
                metadata:
            
                  labels:
            
                    app: cloudflared
            
                spec:
            
                  containers:
            
                  - name: cloudflared
            
                    image: cloudflare/cloudflared:latest
            
                    args:
            
                    - tunnel
            
                    # - --config
            
                    # - /etc/cloudflared/config.yml
            
                    - run
            
                    env:
            
                    - name: TUNNEL_TOKEN
            
                      valueFrom:
            
                        secretKeyRef:
            
                          name: tunnel-credentials
            
                          key: argo-tunnel-token
            
                    livenessProbe:
            
                      httpGet:
            
                        path: /ready
            
                        port: 2000
            
                      initialDelaySeconds: 1
            
                      periodSeconds: 10
            ```
            
        
        `kubectl apply -f cloudflared-argocd.yaml`
        

### 잘 모르는 용어 정리

- CNCF(Cloud Native Compution Foundations) - 리눅스 제단 산하의 비영리 단체로, 클라우드 네이티브 오픈소스를 관리하는 곳. Sandbox > Incubating > Graduated 순으로 기술 등급을 매겨주는데, Graduated 수준이면 기업에서 사용해도 될 만큼의 성숙도가 있다고 판단하는 것임.
- Topology - 쿠버네티스 상에서 앱을 구성하는 리소스(Pod, Service, Ingress 등)들이 어떻게 연결되어있는지 보여주는 구조도.
- GitOps - Git을 원천으로 삼아 인프라를 운영하는 방법론
1. 선언적(Declarative) - 시스템이 원하는 상태가 YAML로 선언되어 있어야 함.
2. 버전 관리(Versioned) - 각 버전 파일들이 GIt에 저장되어 이력 관리(버전 관리)가 되어야 함.
3. 자동 적용(Automated) - Git에 코드가 올라가면 자동으로 클러스터에 반영.
4. 자가 치유(Self-Healing) - 실제 서버 상태가 git과 달라지면, 도구가 감지하고 Git 상태로 강제로 복구.

참고자료

| URL | 내용 |
| --- | --- |
| [https://www.redhat.com/ko/topics/devops/what-is-ci-cd](https://www.redhat.com/ko/topics/devops/what-is-ci-cd) | CI/CD의 기본적인 정의 |
| [https://nangman14.tistory.com/92](https://nangman14.tistory.com/92) | 컨테이너 빌드 도구 비교 |
| [https://wlsdn3004.tistory.com/37](https://wlsdn3004.tistory.com/37) | ArgoCD의 개념 및 설치 |
| [https://channel.io/ko/team/blog/articles/GitHub-Actions-도입기1-ARC-구축-및-Container-Jobs-지원-55fa0670](https://channel.io/ko/team/blog/articles/GitHub-Actions-%EB%8F%84%EC%9E%85%EA%B8%B01-ARC-%EA%B5%AC%EC%B6%95-%EB%B0%8F-Container-Jobs-%EC%A7%80%EC%9B%90-55fa0670) | ARC의 정의 및 설치 |
| [https://nwblog06.tistory.com/630](https://nwblog06.tistory.com/630) | Github Labs VS Actions |
| [https://docs.github.com/ko](https://docs.github.com/ko) | 깃허브 액션 공식문서 |
| [https://cert-manager.io/docs/releases/](https://cert-manager.io/docs/releases/) | Cert-manager 공식 릴리즈 문서 |
| [https://docs.github.com/en/actions/tutorials/use-actions-runner-controller/quickstart](https://docs.github.com/en/actions/tutorials/use-actions-runner-controller/quickstart) | ARC 공식 문서 |
| [https://kubernetes.io/docs/concepts/windows/intro/](https://kubernetes.io/docs/concepts/windows/intro/) | 쿠버네티스 윈도우즈 공식 문서 |
| [https://argo-cd.readthedocs.io/en/release-3.0/operator-manual/installation/#supported-versions](https://argo-cd.readthedocs.io/en/release-3.0/operator-manual/installation/#supported-versions) | ArgoCD 3.0 버전 릴리즈 호환 확인 |

[기존에 적어둔 내용](https://www.notion.so/300b76172ccb80a98af8e892edd9135e?pvs=21)