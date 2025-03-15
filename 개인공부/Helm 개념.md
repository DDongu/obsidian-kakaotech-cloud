---
sticker: emoji//270d-fe0f
---
Refer: [Helm 이론 유튜브 강의](https://www.youtube.com/watch?v=w51lDVuRWuk)
# Helm
: Kubernetes에서 어플리케이션을 패키징, 관리 및 배포하는 도구

# Component
**1. Helm Chart**: k8s 어플리케이션을 배포하기 위한 템플릿화된 패키지
- Deployment, service, ConfigMap, Ingress 등 여러 개의 k8s 리소스를 하나의 Chart로 묶어서 관리함

**2. Configs Values**: Helm Chart에서 사용하는 기본 설정 파일
- `values.yaml` 파일을 통해 어플리케이션 동작을 커스터마이징 가능 
	-> 기본 템플릿을 두고 동적으로 변경되는 값을 통해 다른 어플리케이션을 배포 가능

**3. Releases**: Helm Chart가 k8s에 배포된 인스턴스(실행중)
- 동일한 Chart라도 여러 개의 release를 배포할 수 있음
- release에 설치, 업그레이드, 롤백 수행

📌 예시 **(`templates/deployment.yaml`)**
```sh
# 예시
helm install my-nginx nginx
```

**4. Repository**: Helm Chart를 저장하고 배포하는 저장소
- [Helm 공식 레포지토리](https://artifacthub.io/) 또는 사용자 정의 레포지토리 가능

📌 예시 **(`templates/deployment.yaml`)**
```sh
helm repo add stable https://charts.helm.sh/stable
helm search repo nginx
```

**5. Templating**: Go 템플릿 엔진을 사용하여 동적인  YAML 파일을 생성하는 것
- `templates/` 폴더 내의 `.yaml.` 파일을 동적으로 생성
- `{{ .Values }}` 문법을 사용하여 values.yaml 값을 적용 가능

📌 예시 **(`templates/deployment.yaml`)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: my-container
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
```

# Helm Chart 기본 구조
📌 기본 디렉터리 구조
```sh
my-helm-chart/        # Chart 이름
│── charts/           # (서브 차트) 다른 Chart를 의존성으로 포함 가능
│── templates/        # (템플릿) 실제 Kubernetes 리소스 정의 (필수❗)
│   ├── NOTES.txt     # (배포 메모) 차트 설치 또는 status 명령 시에 콘솔에 인쇄
│   ├── configmap.yaml   # ConfigMap 정의
│   ├── deployment.yaml  # Deployment 정의
│   ├── service.yaml     # Service 정의
│   ├── ingress.yaml     # (선택) Ingress 정의
│   ├── _helpers.tpl     # 템플릿 헬퍼 함수
│── values.yaml        # (설정 값) 사용자 정의 변수 (필수❗)
│── Chart.yaml         # (메타데이터) Chart의 기본 정보 (필수❗)
│── .helmignore        # (무시 파일) Chart 패키징 시 제외할 파일 목록
```

# Helm v2와 Helm v3의 차이
Helm v2
- **Tiller**라는 서버 컴포넌트가 k8s 클러스터에 설치되어 Release를 관리함
- 같은 namespace에서 같은 이름의 Release를 사용하여 충돌이 발생
Helm v3
- Tiller를 완전히 제거 -> CLI에서 직접 k8s API와 통신하여 설치 및 업데이트를 진행
- k8s와의 호환성이 더 향상됨

```
❓Tiller가 없어진 이유
Tiller는 보안 취약점과 복잡성을 유발하기 때문에 v3에서 없애버림
```

∴ Helm v3는 보안 문제나 k8s 호환성, 이름 충돌 등의 문제가 해결되었기 때문에 Helm v3를 사용하는게 좋음.

