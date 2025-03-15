---
sticker: emoji//270d-fe0f
aliases:
  - Jenkins 개념\
---
Reference: [Jenkins 사용방법 유튜브 강의](https://www.youtube.com/watch?v=6YZvp2GwT0A)
# 동작 방식
```
🚀 Jenkins 동작 방식
개발자가 git commit을 날림 
-> Master 서버에서 커밋을 인식 -> 파이프라인 생성 -> Label에 맞는 Agent 중 하나에 배포 
-> Agent에서 파이프라인 실행
```

# 리소스
**Master Server** : Jenkins가 돌아가는 서버
- Pipeline을 컨트롤 함
- agent 빌드를 스케쥴 함
**Agent**: 빌드 작업을 실행하는 노드(서버) 또는 컨테이너
- 작업공간에서 빌드를 수행함
- Permanent Agents: 서버에서 고정되어 돌아가는 agent
- Cloud Agents: 비륻가 필요할 때만 동적으로 생성 및 할당되는 agent(빌드가 끝나면 자동 삭제)
	- 클라우드 환경에서 동작하여 Docker, K8S와 같은 컨테이너 기반의 빌드 환경

# 빌드 구성 방법
1. **FressStyle Projects**
- Jenkins UI에서 클릭을 통해, 스크립트 없이 사용할 수 있는 빌드 방식
2. **Declarative Piplines**
-  `Jenkinsfile`을 사용하여 코드 기반으로 CI/CD 파이프라인을 작성하는 빌드 방식
- Groovy 기반의 선언형(`Declarative`) 문법 사용

