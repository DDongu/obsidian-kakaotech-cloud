---
sticker: emoji//270d-fe0f
---
ref: [ArgoCD 유튜브 강의](https://www.youtube.com/watch?v=cLgzqc_hwIg&t=23s) / [ArgoCD 강의 교재](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbDd2ai1sODFIZW1RM2lyaWNHZXpUMnZDcVNFUXxBQ3Jtc0tsdjlLQ3ZocExCSzZmdlZHc243b1BtY2JHcmw2QmVmV1dWcnA0NEx5Q0RJVzFfejU3TFYtVlRqR0d1QzIySmV1SGtzeVRwUV9BZVVidnFNRnFOSS1OWnJCTkRQVTNRVndlbFdtWTFvUkZkejJXa3FvUQ&q=https%3A%2F%2Fsungwook-choi.gitbook.io%2Fargocd%2F&v=XBcoj3-pjDA)
# 동작 방식
```
🚀 ArgoCD 동작 방식
개발자가 git commit을 날림 
-> argoCD에서 check off(현재 상태와 의도상태 차이 감지) -> 쿠버네티스에 Sync
```

# 개념
- ArgoCD는 쿠버네티스에서만 동작함
- ArgoCD는 배포만 하기 때문에 빌드하는 파이프라인(CI)을 추가로 구축해야 됨

## 단위
**Application**: ArgoCD의 동기화 설정이 담겨져 있는 단위
- ArgoCD를 생성할 때 CRD가 생성되어 CRD로 관리됨
**Project**: Application을 관리하는 단위
- Application은 Project에 무조건 속해야 함
- ArgoCd를 설치하면 default Project가 기본 생성됨

# Sync 설정
**Refresh Period**: 동기화 하는 주기
**Sync Policy**: Auto sync(자동) / Manual sync(수동)
## Self Healing 옵션
: acgocd로 동기화된 리소스를 항상 git에 있는 의도된 상태로 보장해주는 기능
- sync policy를 auto로 설정했을 때 사용 가능
- ❗만약, 콘솔에서 `kubectl` 명령어로 리소스를 직접 변경하게 된다면, argocd에서 git 상태와 동일하게 다시 바꿔버림.(즉, 콘솔에서 수정해도 소용이 없다.)
## Prune 옵션
: 동기화된 리소스 삭제 옵션.
- git에서 리소스를 삭제할 떄, 해당 리소스를 쿠버네티스에서 삭제할지 유지할지 결정하는 옵션
- ❗prune이 비활성화(default)되어 있으면, argocd에서 동기화 된 리소스는 git에서 삭제되더라고 k8s에서는 삭제되지 않고 남아있음
## Directory Recurse옵션
: git의 path를 지정했을 때, path 안에 다른 디렉터리가 있으면 그 디렉터리까지 다 반영하겠다는 옵션
- 비활성화 시, path에 있는 `파일`만 반영하고 디렉터리는 반영 x
## Ignore Difference 
: 특정 필드나 리소스를 비교대상에 제외시키는 기능
- 예를 들면, HPA를 설정했을 때 오토스케일링으로 파드 수가 증가하게 되면 이는 git 상태와 달라지게 됨. 따라서 argocd에서 이를 감지하게 되고 git 상태와 동일하게 바꾸려고 함. 이때 ignore difference 옵션을 활성화 하여 해당 파드나 hpa를 제외하게 하여 HPA를 정상적으로 적용시킬 수 있음.

# Phases
: ArgoCD sync 과정 순서 (**pre sync -> sync -> post sync**)
- phase를 활용하여 DB업데이트 ,알림 기능 등을 구현할 수 있음
## pre sync와 post sync 설정 방법
📌 PreSync 설정방법
```
apiVersion: batch/v1
kind: Job
metadata:
  generateName: schema-migrate-
  annotations:
    argocd.argoproj.io/hook: PreSync
```

📌 PostSync 설정방법
```
apiVersion: batch/v1
kind: Job
metadata:
  generateName: integration-test-
  annotations:
    argocd.argoproj.io/hook: PostSync
```

❗`argocd.argoproj.io/hook-delete-policy: HookSucceeded` 설정
	Pre,PostSync는 job이다. 따라서 기본적으로는 수행 후에 남아있게 된다. 이때 위 설정을 넣어주게 되면 정책 수행 후 job이 삭제된다.

💡 [ArcoCD github 기본 예제](https://github.com/argoproj/argocd-example-apps)

# Project
- 어플리케이션을 그룹화하고 관리하는데 사용
**Source Repositories**: git 레퍼지토리를 제한하는 설정
**Destinations**: 네임스페이스를 제한하는 설정
**Cluster resource allow list**: 허용하는 리소스를 제한하는 설정

# 기존 리소스에 ArgoCD 연결

❓**기존 k8s에 리소스가 있고, 그 리소스에 대한 application을 생성하면 application만 생성될까? 리소스와 연결이 될까?**
A. 정답은 생성 시 리소스와 연결이 된다.
=> 그 이유는, application을 생성하고 sync를 하게 되면, argocd에서 해당 리소스의 라벨에 argocd의 관리 라벨을 추가하여 리소스가 연결이 된다.

❓**기존에 있던 리소스가 application에 연결이 되었고, 그 application을 삭제한다면 리소스는 어떻게 될까?**
A. 정답은 리소스도 같이 삭제된다.
=> application을 삭제하게 되면, 그와 연결된 리소스 뿐만 아니라 네임스페이스도 삭제가 된다. 따라서 운영 단계에서, 실수로 기존 리소스 또는 네임스페이스와 연결되는 application 만들고 이를 삭제하게 되면, *삭제되면 안되는 리소스가 삭제되는 참사가 일어날 수 있다.*

### Non Cascade 옵션을 통한 삭제
: Application을 삭제할 때 쿠버네티스 리소스는 유지하도록 설정하는 옵션
- Application 삭제 시 `Non Cascade` 옵션을 선택하게 된다면, application만 삭제가 되고 k8s의 리소스는 그대로 남아있게 됨

#  App of Apps
: ArgoCD application을 그룹으로 관리하는 방법
![[Pasted image 20250314161246.png]]

📌예제
```json
apiVersion: argoproj.io/v1alpha1
kind: Application                                 # Application으로 정의되어 있음
metadata:
  name: helm-guestbook
  namespace: argocd
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  destination:
    namespace: helm-guestbook
    server: {{ .Values.spec.destination.server }}
  project: default
  source:
    path: helm-guestbook                           # helm-guestbook라는 디렉터리는 k8s의 리소스가 정의되어 있음
    repoURL: {{ .Values.spec.source.repoURL }}
    targetRevision: {{ .Values.spec.source.targetRevision }}
```
=> 해당 yaml 파일이 배포되게 되면 application이 올라오게 되고,  path에 해당되는 application이 생성됨
(*Application -> Application을 가리키는 구조가 만들어짐*)

💡어떻게 사용될까?
```
**Cluster Bootstrapping에 사용**
=> 새로운 클러스터를 생성할 때, 해당 패턴의 파일을 만들어둔 뒤 빠르게 생성하게 됨
=> ArgoCD user가 사용하기 보다, admin을 위한 방식임
```

❗**주의**
*App of Apps를 삭제하게 된다면, 위에서 말했던 이유처럼 연동되어 있는 application이 다 삭제되어 날릴 수 있음*

