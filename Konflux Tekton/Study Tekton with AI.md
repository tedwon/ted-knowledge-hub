
좋은 질문이에요 👍

Tekton은 **Kubernetes 기반의 CI/CD(Continuous Integration/Continuous Delivery) 시스템**이에요.

쉽게 말하면, Kubernetes 위에서 돌아가는 **파이프라인 툴**이고, Jenkins 같은 것보다 더 **클라우드 네이티브**스럽게 동작해요.

---

## **Tekton 핵심 개념**

  

Tekton을 이해하려면 이 3가지를 알면 돼요:

1. **Task**
    
    - “할 일” 하나를 정의
        
    - 예: 코드 빌드, 테스트 실행, 이미지 푸시 등
        
    - Task는 하나 이상의 Step으로 구성돼요. Step은 실제 컨테이너 실행 단위예요.
        
    
2. **Pipeline**
    
    - 여러 Task를 순서대로 묶은 것
        
    - CI/CD 전체 흐름을 정의해요.
        
    
3. **PipelineRun / TaskRun**
    
    - Pipeline이나 Task를 실제로 실행시키는 인스턴스
        
    

---

## **실제 예제 따라하기**

  

아래 예제는 **간단한 hello-world 파이프라인**이에요.

이걸 따라하면 Tekton 감 잡을 수 있어요.

  

### **1. Task 만들기 (hello-world-task.yaml)**

```
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello-world-task
spec:
  steps:
    - name: say-hello
      image: alpine
      script: |
        #!/bin/sh
        echo "Hello from Tekton Task!"
```

👉 이 Task는 alpine 컨테이너에서 echo만 해요.

  

적용:

```
kubectl apply -f hello-world-task.yaml
```

---

### **2. Pipeline 만들기 (hello-world-pipeline.yaml)**

```
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: hello-world-pipeline
spec:
  tasks:
    - name: run-hello
      taskRef:
        name: hello-world-task
```

👉 이 Pipeline은 위에서 만든 Task를 실행만 해요.

  

적용:

```
kubectl apply -f hello-world-pipeline.yaml
```

---

### **3. PipelineRun 실행하기 (hello-world-pipelinerun.yaml)**

```
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: hello-world-run
spec:
  pipelineRef:
    name: hello-world-pipeline
```

적용:

```
kubectl apply -f hello-world-pipelinerun.yaml
```

---

### **4. 실행 결과 확인하기**

```
kubectl get pipelinerun
kubectl logs -l tekton.dev/task=hello-world-task --all-containers
```

출력:

```
Hello from Tekton Task!
```

---

✅ 이렇게 해서 Tekton의 기본 구조를 한 번 돌려본 거예요.

실제 회사에서는 여기서 Task에 **빌드 → 테스트 → Docker 이미지 빌드 → 레지스트리 푸시 → 배포** 같은 걸 붙여서 CI/CD 파이프라인을 만들어요.

---

Ted, 혹시 내가 지금 보여준 예제를 네가 직접 로컬 Kubernetes(예: minikube나 kind)에서 돌려볼 수 있도록 **설치 가이드**도 알려줄까? 아니면 개념만 잡고 싶어?

좋습니다 👍 그럼 지금부터 **macOS 로컬 환경에서 Tekton을 직접 실행해보는 설치 가이드**를 단계별로 알려드릴게요.

우리는 kind (Kubernetes in Docker)를 이용해서 간단히 클러스터 올린 뒤 Tekton을 설치하는 흐름으로 갈게요.

---

## **1. 사전 준비**

  

macOS에서 필요한 툴들을 먼저 설치해야 해요.

  

### **Homebrew 설치 (없다면)**

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### **Docker 설치**

```
brew install --cask docker
```

👉 설치 후 Docker Desktop 실행해두세요.

  

### **kind 설치**

```
brew install kind
```

### **kubectl 설치**

```
brew install kubectl
```

---

## **2. Kubernetes 클러스터 생성**

  

kind를 이용해서 로컬 클러스터 하나 만듭니다.

```
kind create cluster --name tekton-cluster
```

확인:

```
kubectl cluster-info --context kind-tekton-cluster
```

---

## **3. Tekton 설치**

  

Tekton의 기본 컴포넌트(Tekton Pipelines)를 설치해야 해요.

```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

👉 설치가 끝날 때까지 잠시 기다립니다.

  

상태 확인:

```
kubectl get pods -n tekton-pipelines
```

모든 Pod가 Running이어야 정상입니다.

---

## **4. (선택) Tekton CLI 설치**

  

명령어로 쉽게 다루려면 tkn CLI가 있으면 좋아요.

```
brew install tektoncd-cli
```

확인:

```
tkn version
```

---

## **5. Hello World 예제 실행**

  

이제 앞에서 보여준 **Task, Pipeline, PipelineRun** YAML을 적용하면 돼요.

  

예제 폴더 만들기:

```
mkdir tekton-hello && cd tekton-hello
```

파일 만들기 (nano 또는 VSCode 사용):

- hello-world-task.yaml
    
- hello-world-pipeline.yaml
    
- hello-world-pipelinerun.yaml
    

  

적용:

```
kubectl apply -f hello-world-task.yaml
kubectl apply -f hello-world-pipeline.yaml
kubectl apply -f hello-world-pipelinerun.yaml
```

실행 확인:

```
tkn pipelinerun list
tkn pipelinerun logs hello-world-run -f
```

👉 출력이 나오면 성공 🎉

```
Hello from Tekton Task!
```

---

## **6. 정리 (클러스터 삭제)**

  

나중에 다 쓰고 나면 리소스 아끼려면 클러스터 삭제하면 돼요.

```
kind delete cluster --name tekton-cluster
```

---

여기까지 따라 하면 macOS에서 바로 Tekton 파이프라인을 경험할 수 있어요.

  

Ted, 혹시 내가 이 설치 가이드를 **하나의 쉘 스크립트로 자동화**해서 한 번에 실행되게 만들어줄까?