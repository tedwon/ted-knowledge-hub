좋아요, Konflux를 Tekton 기반으로 사용하는 방법을 실제 예제 중심으로 같이 살펴볼게요. 많이 정보가 공개되어 있진 않아서 “사용자 측면(onboarding + 간단한 component 만들기)” 관점으로 가능한 절차 중심으로 설명할게요.

---

## **Konflux 란?**

  

먼저 Konflux가 뭔지 간단 정리하면:

- 오픈소스, 쿠버네티스 네이티브 CI/CD 소프트웨어 팩토리(platform)예요. Tekton 위에 여러 보안(supply chain security), 컴플라이언스, 서브스크립션(Red Hat subscription) 통합 등을 추가한 구조. 
    
- Git 이벤트(GitHub / GitLab)로 파이프라인을 트리거 가능하고, Tekton PipelineRuns, Tasks, Chains, Results 등을 이용해 빌드/테스트/릴리즈 과정을 구성함. 
    
- 프로젝트(application), component, snapshot, release, integration-test-scenario, release-plan 등의 커스텀 리소스(CR)를 통해 워크플로우를 선언적(declarative)으로 정의함. 
    

---

## **Konflux 사용하는 실제 예제 따라해보기**

  

지역 환경(macOS + kind 또는 미니쿠베 같은 쿠버네티스 클러스터)에서는 Konflux 전체 운영 환경을 똑같이 구성하기 어렵지만, 사용자(onboarding) + 간단한 component 빌드 흐름을 체험해볼 수 있는 가상의 워크플로우를 만들어 볼게요.

  

아래는 “내 코드 저장소 → Konflux component 등록 → GitHub PR로 인해 파이프라인 트리거됨 → 빌드 → SBOM / provenance 생성 → 결과 확인” 이런 흐름이에요.

---

## **가정 및 사전 조건**

  

이 예제를 따라하려면 다음이 필요해요:

1. 쿠버네티스 클러스터가 있고 Konflux 인스턴스(예: 클러스터 + Konflux 관련 CRDs, 컨트롤러)가 설치되어 있어야 함.
    
    → 만약 내부에 Konflux 설치 권한 없으면 Red Hat 내부 또는 허가된 Konflux 환경이 필요함.
    
2. GitHub 저장소: 코드 + Tekton 파이프라인 정의 등이 있어야 함. (예: .tekton/ 디렉터리)
    
3. OCI 레지스트리 접근 가능 (이미지/artifact push 가능)
    
4. 필요한 권한: Konflux tenant namespace 접근 + Component/Application 생성 권한 등.
    
5. 필요한 secret들을 준비 (예: Red Hat activation key, entitlement, etc.) – 만약 Red Hat 구독 내용이 필요한 경우. 
    

---

## **단계별 예제**

  

아래 워크플로우를 따라가면 Konflux에서 나만의 간단한 component를 등록하고, GitHub PR로 빌드 트리거 하는 과정을 체험할 수 있어요.

---

### **1단계: GitHub 저장소 준비**

  

예: my-konflux-demo 라는 저장소를 GitHub에 만듦.

  

구조 예:

```
my-konflux-demo/
├── .tekton/
│    ├── pipelines/
│    │    └── build-pipeline.yaml
│    └── tasks/
│         └── build-task/
│              └── 0.1/
│                   └── build-task.yaml
└── src/
     └── main.go    <-- 간단한 Go 프로그램 예제
```

- build-task.yaml 은 소스를 받아서 빌드하고 이미지를 만들고 레지스트리에 push 하는 작업을 정의.
    
- build-pipeline.yaml 은 해당 task를 호출하는 pipeline 정의.
    

  

예제 build-task.yaml (Tekton Task 버전):

```
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: build-task
spec:
  params:
    - name: repo-url
      type: string
    - name: revision
      type: string
  workspaces:
    - name: source
  steps:
    - name: clone
      image: alpine/git
      script: |
        #!/bin/sh
        git clone --branch $(params.revision) $(params.repo-url) /workspace/source
    - name: build
      image: golang:1.21
      workingDir: /workspace/source
      script: |
        #!/bin/sh
        go build -o app main.go
    - name: docker-build-push
      image: quay.io/buildah/stable:latest
      script: |
        #!/bin/sh
        buildah bud -f $(workspaces.source)/Dockerfile -t myregistry/myimage:$(params.revision) $(workspaces.source)
        buildah push myregistry/myimage:$(params.revision)
```

예제 build-pipeline.yaml:

```
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-pipeline
spec:
  params:
    - name: repo-url
      type: string
    - name: revision
      type: string
  workspaces:
    - name: source
  tasks:
    - name: build
      taskRef:
        name: build-task
      params:
        - name: repo-url
          value: $(params.repo-url)
        - name: revision
          value: $(params.revision)
      workspaces:
        - name: source
          workspace: source
```

---

### **2단계: Konflux에서 Application / Component 등록**

  

Konflux UI 또는 kubectl을 통해 다음과 같은 CR을 만듭니다.

  

예: my-demo-app 이라는 **Application**, 그 아래 build-task-component 같은 **Component**.

  

예시 CR (kubectl로 적용 가능한 YAML):

```
apiVersion: apps.konflux-ci.dev/v1alpha1  # 실제 API 그룹/버전은 Konflux 문서 확인
kind: Application
metadata:
  name: my-demo-app
  namespace: <내 tenant 네임스페이스>
spec:
  # 필요한 필드, 예: description, owner 등
  description: "Demo app for Konflux build"
---
apiVersion: apps.konflux-ci.dev/v1alpha1
kind: Component
metadata:
  name: build-task-component-v01
  namespace: <내 tenant 네임스페이스>
spec:
  application: my-demo-app
  source:
    type: git
    url: https://github.com/<your-org>/my-konflux-demo.git
    revision: main
    path: .tekton/tasks/build-task/0.1
  pipeline:
    type: default  # 혹은 사용자 정의 pipeline type
```

(필드명은 실제 Konflux 문서 보고 맞춰야 함) 

  

이렇게 하면 Konflux 쪽에서 기본 pipeline run이 Source repo 리포 안의 .tekton/tasks/... 경로를 보고, 해당 Task를 OCI artifact (bundle)으로 빌드하고, provenance/SBOM 등의 메타데이터를 생성함. 

---

### **3단계: GitHub 앱 연동**

  

GitHub 저장소에 PR 또는 push 이벤트가 생기면 Konflux가 자동으로 파이프라인을 실행하도록 GitHub App을 설치하고 설정함. 

- Konflux GitHub App을 repo에 설치. 
    
- .tekton/ 폴더 안의 PipelineRun 정의 또는 Konflux가 만든 default 파이프라인을 이용해, PR 또는 push 이벤트 시 trigger 되도록 설정됨. 
    

---

### **4단계: 빌드 및 결과 확인**

- PR을 열거나 커밋 + push 하면 Konflux가 해당 component에 대한 Tekton PipelineRun을 트리거함.
    
- 작업이 실행되면서 다음과 같은 아티팩트/메타정보들이 생성됨:
    
    - 생성된 컨테이너 이미지 또는 OCI artifact 
        
    - SBOM, provenance attestation (예: in-toto, Tekton Chains) 
        
    - 로그, 테스트 결과 등 Konflux UI 혹은 kubectl을 통해 접근 가능.
        
    

---

## **간단한 가상의 예제: “hello” 컴포넌트**

  

좀 더 실습 형식으로 간단한 예제로 만들어볼게요.

---

### **예제 목표**

- hello-app 이라는 component 만들기
    
- 코드: echo "Hello, Konflux!" 간단
    
- 이미지는 OCI artifact로 빌드
    
- GitHub PR/push로 트리거
    
- Build 결과 + provenance 보기
    

---

### **예제 코드 및 repo 구조**

```
hello-konflux/
├── .tekton/
│    ├── tasks/
│    │    └── hello-task/
│    │         └── 0.1/
│    │              └── hello-task.yaml
│    └── pipelines/
│         └── hello-pipeline.yaml
└── src/
     └── hello.txt   # 단순 텍스트를 echo로 출력
```

- hello-task.yaml:
    

```
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello-task
spec:
  steps:
    - name: say-hello
      image: alpine
      script: |
        #!/bin/sh
        echo "Hello, Konflux! Step from $(context.pipelineRun.name)"
```

- hello-pipeline.yaml:
    

```
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: hello-pipeline
spec:
  tasks:
    - name: hello
      taskRef:
        name: hello-task
```

---

### **예제 절차**

1. GitHub 저장소 생성: 코드 + 위 .tekton/ 디렉터리 올리기.
    
2. Konflux에서 component 등록:
    

```
apiVersion: apps.konflux-ci.dev/v1alpha1
kind: Component
metadata:
  name: hello-component-v01
  namespace: <내 tenant 네임스페이스>
spec:
  application: hello-app
  source:
    type: git
    url: https://github.com/<your-org>/hello-konflux.git
    revision: main
    path: .tekton/tasks/hello-task/0.1
  pipeline:
    type: default
```

3. GitHub App 설치 & 권한 확보.
    
4. Push 하거나 PR 생성 → Konflux가 자동으로 실행.
    
5. 빌드 로그/결과 보기: UI + kubectl, artifact 및 provenance 보기.
    

---

## **제한사항 / 알아두면 좋은 점**

- 당신이 Konflux 인스턴스를 직접 설치하지 않았다면, 팀 또는 조직에서 Konflux 액세스가 필요함.
    
- 사용자 customization 가능성은 있지만 사전에 정의된 pipeline 타입(default vs custom) 혹은 Konflux 빌드 정책(보안 / 컴플라이언스 관련) 영향을 받음. 
    
- Red Hat subscription 또는 activation key 설정이 필요할 수 있음, 특히 빌드가 Red Hat 패키지 또는 RPMs 등을 참조해야 할 경우. 
    

---

필요하면, 너의 환경(macOS + kind)에서 Konflux를 **로컬로 설치해보고** 이 예제를 돌리는 방법도 같이 준비해줄까? 그렇게 하면 실제로 손에 익을 거야.


좋은 질문이에요 👍

Konflux에서 **Application**은 제일 상위에 있는 개념이에요.

---

## **쉽게 말하면**

- **Application = 프로젝트 전체를 대표하는 컨테이너**
    
- 그 안에 여러 **Component**(마이크로서비스, 라이브러리, 빌드 가능한 유닛)를 담을 수 있어요.
    
- CI/CD 파이프라인은 보통 **Application → Component → PipelineRun** 구조로 내려가면서 실행돼요.
    

---

## **공식 문서에서의 정의**

  

Konflux 문서에 따르면 ([konflux-ci.dev](https://konflux-ci.dev/docs/getting-started/)):

- Application은 “하나의 소프트웨어 제품이나 서비스”를 나타내는 오브젝트예요.
    
- Application 안에는 여러 Component를 넣을 수 있고, 각각의 Component는 Git repo, 빌드 방식(Task/Pipeline) 등을 정의해요.
    
- Application 레벨에서는 **릴리즈 계획**, **스냅샷(snapshot)**, **통합 테스트** 같은 더 넓은 수준의 작업을 관리할 수 있어요.
    

---

## **예시로 이해하기**

  

예를 들어 **“온라인 쇼핑몰 서비스”**를 만든다고 해볼게요.

- Application: shopping-app
    
    - Component: frontend (React)
        
    - Component: backend (Quarkus, Spring Boot 같은 API 서버)
        
    - Component: payment-service (별도 Go 서비스)
        
    - Component: recommendation-service (Python ML 모델)
        
    

  

👉 이렇게 한 Application에 여러 Component가 속해서 함께 개발되고, Konflux 파이프라인이 각 Component를 빌드/테스트/보안검증하고, 최종적으로는 Application 단위로 **스냅샷 → 릴리즈**까지 이어지는 거예요.

---

## **YAML 예제 (간단)**

```
apiVersion: appstudio.redhat.com/v1alpha1
kind: Application
metadata:
  name: shopping-app
  namespace: team-namespace
spec:
  description: "Online shopping service built with multiple components"
```

👉 이 Application 안에 나중에 Component 리소스를 추가하게 돼요.

---

정리하면:

**Application = Konflux에서 전체 소프트웨어 제품을 묶는 최상위 리소스**,

**Component = 그 Application을 구성하는 개별 서비스/모듈**이라고 보면 돼요.

---

Ted, 혹시 내가 Application → Component → Snapshot → ReleasePlan 흐름을 **그림으로 시각화**해서 보여줄까?