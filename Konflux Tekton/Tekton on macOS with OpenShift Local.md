
알겠습니다. 방금 공유해주신 실제 터미널 출력 내용을 반영하여, 이전 총정리 가이드를 더욱 현실적이고 구체적인 내용으로 업데이트해 드리겠습니다.

---

### **Tekton on macOS with OpenShift Local: 최종 정리 (v2)**

이 문서는 Red Hat의 CI/CD 솔루션인 **OpenShift Pipelines (Tekton)**를 로컬 macOS 환경에서 학습하고 실행하기 위한 모든 과정을 요약합니다.

---

### **1. 최종 개발 환경 구성 🖥️**

- **핵심 도구:**
    
    - **컨테이너 환경:** **Podman Desktop**
        
    - **로컬 Kubernetes 클러스터:** **OpenShift Local**
        
- **설치 방법:** **Podman Desktop UI** 내의 확장(Extensions) 메뉴에서 OpenShift Local을 찾아 설치하는 것이 가장 확실한 방법입니다.
    
- **필수 CLI 도구 (Homebrew로 설치):**
    
    - **OpenShift CLI (`oc`):** `brew install openshift-cli`
        
    - **Tekton CLI (`tkn`):** `brew install tektoncd-cli`
        

---

### **2. 클러스터 시작 및 로그인 🚀**

1. 클러스터 시작 명령어

터미널에서 아래 명령어를 실행하여 OpenShift Local 클러스터를 시작합니다.

Bash

```
# 초기 설정 (최초 한 번)
crc setup

# 클러스터 시작
crc start --driver podman
```

2. 실행 결과 확인

crc start 명령을 실행하면, 여러 단계의 초기화 과정이 로그로 출력됩니다. 모든 과정이 성공적으로 끝나면, 아래와 같이 클러스터 접속 정보가 나타납니다.

Bash

```
~ » crc start
INFO Using bundle path /Users/jwon/.crc/cache/crc_vfkit_4.19.8_arm64.crcbundle
... (많은 초기화 로그) ...
INFO Operators are stable (3/3)...
INFO Adding crc-admin and crc-developer contexts to kubeconfig...
Started the OpenShift cluster.

The server is accessible via web console at:
  https://console-openshift-console.apps-crc.testing

Log in as administrator:
  Username: kubeadmin
  Password: vRevF-xKTzK-wUgWk-MXN6n

Log in as user:
  Username: developer
  Password: developer

Use the 'oc' command line interface:
  $ eval $(crc oc-env)
  $ oc login -u developer https://api.crc.testing:6443
```

3. 관리자 계정으로 CLI 로그인

위 출력 결과에 나온 비밀번호를 사용하여 kubeadmin 관리자 계정으로 로그인합니다. 이것이 앞으로 모든 작업을 수행할 계정입니다.

Bash

```
oc login https://api.crc.testing:6443 -u kubeadmin -p vRevF-xKTzK-wUgWk-MXN6n
```

_(참고: 비밀번호는 `crc start`를 실행할 때마다 변경될 수 있습니다.)_

---

### **3. Tekton 핵심 개념 요약 🧱**

|개념|비유|설명|
|---|---|---|
|**Step**|**일꾼** 🏃|컨테이너 안에서 단일 명령을 수행하는 최소 작업 단위입니다.|
|**Task**|**작업 설계도** 📝|여러 `Step`들을 묶어놓은 **재사용 가능한 템플릿**입니다.|
|**TaskRun**|**작업 지시서** 📨|`Task` 설계도를 실제로 실행하라고 내리는 명령입니다.|
|**Pipeline**|**공장 조립라인 설계도** 🏭|여러 `Task`들을 **어떤 순서로 실행할지** 정의한 전체 워크플로우 템플릿입니다.|
|**PipelineRun**|**공장 가동 명령** 🏭▶️|`Pipeline` 설계도 전체를 처음부터 끝까지 실행하라고 내리는 최종 명령입니다.|

---

### **4. 최종 실습 예제 전체 코드 📋**

**1. OpenShift Pipelines Operator 설치:**

Bash

```
# 새 프로젝트 생성 및 이동
oc new-project tekton-pipelines-demo

# Operator 설치 (Tekton 기능 활성화)
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-pipelines-operator-rh
  namespace: openshift-operators
spec:
  channel: "latest"
  installPlanApproval: Automatic
  name: openshift-pipelines-operator-rh
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

**2. Task 및 Pipeline 생성:**

Bash

```
# 'hello' Task 생성
cat <<EOF | oc apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello
spec:
  steps:
    - name: echo-hello
      image: registry.access.redhat.com/ubi8/ubi-minimal
      script: 'echo "➡️ Step 1: Hello from the first task!"'
EOF

# 'goodbye' Task 생성
cat <<EOF | oc apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: goodbye
spec:
  steps:
    - name: echo-goodbye
      image: registry.access.redhat.com/ubi8/ubi-minimal
      script: 'echo "✅ Step 2: Goodbye! The pipeline has finished."'
EOF

# 두 Task를 연결하는 Pipeline 생성
cat <<EOF | oc apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: hello-goodbye-pipeline
spec:
  tasks:
    - name: task-1-hello
      taskRef:
        name: hello
    - name: task-2-goodbye
      taskRef:
        name: goodbye
      runAfter:
        - task-1-hello
EOF
```

**3. Pipeline 실행 및 로그 확인:**

Bash

```
# PipelineRun으로 파이프라인 실행
cat <<EOF | oc apply -f -
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: hello-goodbye-run-1
spec:
  pipelineRef:
    name: hello-goodbye-pipeline
EOF

# 실행 로그 확인
tkn pipelinerun logs --last -f
```

---

### **5. OpenShift 웹 콘솔 접속 🌐**

파이프라인 실행을 시각적으로 확인하려면 웹 콘솔에 접속합니다.

- **접속 정보 확인:** 터미널에 `crc console` 명령을 입력하면 URL과 `kubeadmin` 계정의 비밀번호를 다시 확인할 수 있습니다.
    
- **파이프라인 확인:**
    
    1. 콘솔 로그인 후, 좌측 상단에서 **Administrator** → **Developer** 모드로 전환합니다.
        
    2. 좌측 메뉴에서 **Pipelines** 탭을 클릭하면 실행된 파이프라인과 그 결과를 그래픽으로 확인할 수 있습니다.