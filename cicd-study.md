<h1> CICD - Study </h1>

1️⃣ WSL2 설치 (리눅스 환경 만들기) <br>

 ✅ PowerShell 관리자 실행
 ```bash
   wsl.exe --install -d Ubuntu-22.04
 ```


WSL2 활성화
Ubuntu 자동 설치됨

✅ 설치 확인
```bash
   wsl -l -v
```


✅ Ubuntu 최초 실행
wsl  
id pw 생성(swiber, 1234)

------------------------------------------------------------------------

2️⃣ Docker Desktop 설치

Docker Desktop 실행 → Settings → WSL Integration

✔ 체크해야 할 것:

✅ “Enable integration with my default WSL distro”
✅ Ubuntu 체크

------------------------------------------------------------------------

3️⃣ k3d 설치 (WSL 안에서)

✅ 설치 명령어

WSL(Ubuntu) 안에서 실행:
```bash
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

✅ 설치 확인
k3d version

------------------------------------------------------------------------

1️⃣ k3d 클러스터 생성

WSL(Ubuntu)에서 실행:
```bash
k3d cluster create cicd-cluster -p "8081:80@loadbalancer"
```


✅ 확인
```bash
kubectl get nodes
```


👉 결과:

k3d-cicd-cluster-server-0   Ready
 ArgoCD 설치 (핵심🔥)

 -----------------------------------------------------------------------

2️⃣ 👉 Argo CD

✅설치
```bash
kubectl create namespace argocd
```

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```


 -----------------------------------------------------------------------

3️⃣ ArgoCD 접속 설정 <br>

✅ NodePort로 열기
```bash
kubectl patch svc argocd-server -n argocd \
-p '{"spec": {"type": "NodePort"}}'
```

✅ 포트 확인

```bash
kubectl get svc -n argocd
```


👉 port-forward 사용(백그라운드)
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
```


👉 그 다음 접속:

https://localhost:8080

NodePort가 Docker 내부 네트워크에만 열리기 때문

초기 비밀번호 확인

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```


id admin <br>
pw 4RrQawRRFbp2BuG1

 -----------------------------------------------------------------------

1️⃣ Jenkins 설치

Helm 설치 (필수)

👉 Helm

WSL(Ubuntu)에서 실행 👇

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

✅ Helm으로 설치 

```bash
kubectl create namespace jenkins
```
```bash
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm install jenkins jenkins/jenkins -n jenkins
```
✅ 접속설정(포트포워딩)

```bash
kubectl port-forward svc/jenkins -n jenkins 18080:8080 &
```


✅ admin password

```bash
kubectl get secret jenkins -n jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 -d
```


id admin <br>
pw ExK2p6jgif2u9Zt6pF7lYz

 -----------------------------------------------------------------------

2️⃣ Git Repo 준비 

👉 repo 2개 구조 

1. app-repo (코드)
2. infra-repo (k8s yaml)
📦 예시 구조
app-repo
<br>
app/ <br>
 ├── app.py <br>
 └── Dockerfile
infra-repo
<br>
k8s/ <br>
 └── deployment.yaml<br>
📄 deployment.yaml 예시 
```bash
apiVersion: apps/v1  
kind: Deployment 
metadata: 
  name: myapp   
spec:
  replicas: 1  
  selector: 
    matchLabels:  
      app: myapp 
  template: 
    metadata: 
      labels: 
        app: myapp  
    spec: 
      containers: 
      - name: myapp 
        image: nginx 
        ports: 
        - containerPort: 5000
```
 
 -----------------------------------------------------------------------    
3️⃣ ArgoCD에 Git 연결

👉 Argo CD

✅ ArgoCD UI 접속 → 로그인

👉 Applications → NEW APP

설정
Application Name: infra-cicd
Project Name: default
Sync Policy: Manual
Repository URL: https://github.com/sw1ber/infra-repo.git
Revision: main 또는 HEAD
Path: k8s

Destination:

Cluster URL: https://kubernetes.default.svc
Namespace: default

👉 Create 누르면 자동 배포됨


4️⃣ Jenkins 자동 배포 <br>
app-repo 코드 수정 <br>
→ Jenkins 빌드 <br>
→ Docker Hub에 이미지 push <br>
→ Jenkins가 infra-repo deployment.yaml의 image 태그 수정 <br>
→ Git push <br>
→ ArgoCD 자동 Sync <br>
→ k3d에 새 버전 배포

✅ repo 구조를 두 개의 저장소로 나누기 <br>
app-repo   : 파이썬/프론트/백엔드 소스 + Dockerfile + Jenkinsfile <br>
infra-repo : k8s/deployment.yaml, service.yaml 

<h3>1. ArgoCD 자동 배포 켜기(지금 까지는 수동 Sync)</h3>

```bash
kubectl get application infra-cicd -n argocd -o yaml
```
✅ spec.syncPolicy 없을시 추가 
```bash
kubectl edit application infra-cicd -n argocd
```
📄 수정 예시:
```bash
spec:
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

<h3>2. Docker Hub 계정과 Jenkins Credential 준비 <br> </h3>

Jenkins는 민감정보를 Credentials로 관리하는 게 기본이고, Pipeline에서는 withCredentials로 바인딩해 쓸 수 있다.

Jenkins UI에서 아래 두 개를 먼저 만든다.

첫 번째는 Docker Hub 계정.

Kind: Username with password <br>
ID: dockerhub-creds

두 번째는 GitHub push용 계정.

GitHub Personal Access Token을 쓰는 걸 추천 <br>
Kind는 보통 Username with password 또는 Secret text 
ID 예시: github-creds <br>
그리고 Jenkins가 app-repo와 infra-repo 둘 다 접근 가능해야 한다.

🚀 1️⃣ GitHub Personal Access Token 만들기

👉 GitHub

👉 생성 방법
```bash
GitHub 로그인 
오른쪽 위 프로필 클릭 
Settings 
Developer settings 
Personal access tokens 
Tokens (classic) 클릭 
Generate new token (classic)
```

👉 권한 설정 (중요🔥)

✔ 체크:
``` bash
repo
```
👉 이것만 있으면 충분

👉 생성하면
```bash
ghp_xxxxxxxx
```

👉 이거 한 번만 보여줌 (복사 필수)

🚀 2️⃣ Jenkins에 등록
👉 경로

Jenkins UI에서:
```bash
Manage Jenkins
 → Credentials
 → System
 → Global credentials
 ```
 👉 Add Credentials 클릭
 
 👉 입력값

 Kind 
 ```bash
 Username with password
 ```
 Username
 ```bash
 GitHub 아이디
 ```
 Password
 ```bash
 방금 만든 Personal Access Token(ghp_xxxxxx)
 ```
 ID (중요)
 ```bash
 github-creds
 ```
  Description (선택)
 ```bash
 github token
 ```
 👉 Save

 🚀 3️⃣ 정상 확인

👉 Jenkins에서 테스트:
1. 새 Job 생성

Jenkins 메인 화면에서:
```bash
New Item
이름: github-creds-test
Pipeline 선택
OK
```
2. Pipeline script로 직접 테스트

맨 아래 Pipeline 부분에서:
```bash
Definition: Pipeline script
```

여기에 아래 코드 넣기.
```bash
pipeline {
  agent any

  stages {
    stage('Test GitHub Credentials - Clone') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'github-creds',
          usernameVariable: 'GIT_USER',
          passwordVariable: 'GIT_PASS'
        )]) {
          sh '''
            set -e
            rm -rf infra-repo-test
            git clone https://${GIT_USER}:${GIT_PASS}@github.com/sw1ber/infra-repo.git infra-repo-test
            ls -al infra-repo-test
          '''
        }
      }
    }
  }
}
```
👉 되면 성공


🚀 4️⃣ push 테스트 

1. 기존 Job 수정

기존 github-creds-test Job 수정
```bash
pipeline {
  agent any

  stages {
    stage('Test GitHub Credentials - Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'github-creds',
          usernameVariable: 'GIT_USER',
          passwordVariable: 'GIT_PASS'
        )]) {
          sh '''
            set -e
            rm -rf infra-repo-test
            git clone https://${GIT_USER}:${GIT_PASS}@github.com/sw1ber/infra-repo.git infra-repo-test
            cd infra-repo-test

            echo "jenkins test $(date)" > jenkins-test.txt

            git config user.name "jenkins"
            git config user.email "jenkins@local"

            git add jenkins-test.txt
            git commit -m "test push from jenkins" || true
            git push origin main
          '''
        }
      }
    }
  }
}
```
3. 실행
```bash
Save
Build Now
```

4. 확인

<h4>Jenkins Console Output</h4>

``` bash 
git push origin main
Finished: SUCCESS
```
<h4>GitHub repo</h4>

infra-repo 들어가서
```bash
jenkins-test.txt
```
파일이 생겼는지 확인

-------------------------------------------------------------------

<h3>Jenkins가 실제로 이미지를 빌드할 수 있게 만들기</h3>

<h4>전체 흐름</h4>

```bash
app-repo 코드 수정
→ Jenkins 빌드
→ Docker Hub push
→ infra-repo deployment.yaml 수정
→ GitHub push
→ ArgoCD 자동 Sync
→ 새 버전 배포
```

<h4>1. app-repo 만들기</h4>

이제 infra-repo 말고 앱 소스용 repo 하나 필요

```bash
app-repo/
├─ app.py
├─ requirements.txt
├─ Dockerfile
└─ Jenkinsfile
```
<h4>2. 앱 파일 만들기</h4>

app.py
```bash
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from Jenkins auto deploy!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
requirements.txt
```bash
flask==3.0.3
```

Dockerfile
```bash
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

<h4>3. infra-repo YAML 바꾸기</h4>

infra-repo/k8s/deployment.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: sw1ber/myapp:initial
          ports:
            - containerPort: 5000
```
infra-repo/k8s/service.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 5000
  type: ClusterIP
```

<h4>3. Docker Hub 생성(harbor 대체)</h4>

1. Docker Hub 로그인
2. Repositories 화면으로 이동
3. Create repository 클릭

```bash
Namespace: 네 Docker Hub 계정명
Repository Name: 예를 들면 myapp
Description: 선택
Visibility: Public 또는 Private
```
4. 네이밍

Docker Hub 아이디가 swiber
Repository Name을 myapp

그러면 최종 이미지 이름은 이렇게 된다.

```bash
swiber/myapp
```
Jenkinsfile

```bash
environment {
  IMAGE_NAME = "swiber/myapp"
}
```

deployment.yaml에서 이렇게 사용

```bash
image: swiber/myapp:initial
```
<h4>4. Jenkins에 Docker Hub credential 추가</h4>
Jenkins UI:

```bash
Manage Jenkins
Credentials
System
Global credentials
Add Credentials
```
입력:

```bash
Kind: Username with password
Username: 네 Docker Hub 아이디
Password: Docker Hub 비밀번호 또는 Access Token(dckr_pat_THtm-xxxxxxxx)
ID: dockerhub-creds
```

<h4>5. app-repo에 Jenkinsfile 넣기</h4>

docker build(wsl docker 소켓 연결x) 대신 Kaniko executor를 쓰는 Jenkinsfile로 구성

app-repo/Jenkinsfile

```bash
agent {
  kubernetes {
    yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: jnlp
      image: jenkins/inbound-agent:3355.v388858a_47b_33-17
      tty: true
    - name: kaniko
      image: gcr.io/kaniko-project/executor:debug
      command:
        - /busybox/cat
      tty: true
      volumeMounts:
        - name: docker-config
          mountPath: /kaniko/.docker
  volumes:
    - name: docker-config
      emptyDir: {}
"""
  }
}
"""
    }
  }

  environment {
    IMAGE_NAME = "sw1ber/myapp"
    IMAGE_TAG  = "${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout App') {
      steps {
        checkout scm
      }
    }

    stage('Prepare Docker Auth') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            mkdir -p /home/jenkins/agent/docker-config
            AUTH=$(printf "%s:%s" "$DOCKER_USER" "$DOCKER_PASS" | base64 | tr -d '\n')
            cat > /home/jenkins/agent/docker-config/config.json <<EOF
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "$AUTH"
    }
  }
}
EOF
          '''
        }
      }
    }

    stage('Build & Push Image with Kaniko') {
      steps {
        container('kaniko') {
          sh '''
            cp /home/jenkins/agent/docker-config/config.json /kaniko/.docker/config.json
            /kaniko/executor \
              --context "${WORKSPACE}" \
              --dockerfile "${WORKSPACE}/Dockerfile" \
              --destination "${IMAGE_NAME}:${IMAGE_TAG}"
          '''
        }
      }
    }

    stage('Update Infra Repo') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'github-creds',
          usernameVariable: 'GIT_USER',
          passwordVariable: 'GIT_PASS'
        )]) {
          sh '''
            set -e
            rm -rf infra-repo
            git clone https://${GIT_USER}:${GIT_PASS}@github.com/sw1ber/infra-repo.git
            cd infra-repo

            sed -i 's#image: .*#image: '"${IMAGE_NAME}:${IMAGE_TAG}"'#' k8s/deployment.yaml

            git config user.name "jenkins"
            git config user.email "jenkins@local"
            git add k8s/deployment.yaml
            git commit -m "update image to ${IMAGE_NAME}:${IMAGE_TAG}" || true
            git push origin main
          '''
        }
      }
    }
  }
}
```

<h4>7. Jenkins Job 만들기</h4>

Jenkins에서:
```bash
New Item
이름: myapp-cicd
Pipeline
```
설정:
```bash
Definition: Pipeline script from SCM
SCM: Git
Repository URL: https://github.com/sw1ber/app-repo.git
Branch: */main
Script Path: Jenkinsfile
```
저장 후 Build Now

<h4>8. 빌드 성공 확인</h4>

Jenkins 콘솔에서:
```bash
Checkout 성공
Kaniko build/push 성공
infra-repo push 성공
```
그다음 GitHub infra-repo에서:
```bash
k8s/deployment.yaml의 image 태그 변경 확인
```

그다음 ArgoCD에서:
```bash
잠깐 OutOfSync
자동 Sync
Healthy
```
👉 기존 nginx 제거 
```bash
kubectl delete deploy cicd-nginx
kubectl delete svc cicd-nginx-svc
```

마지막으로 앱 확인:
```bash
kubectl get pods
kubectl port-forward svc/myapp-svc 28080:80 &
```
브라우저:
```bash
http://localhost:28080
```

🚀 최종 구조 (한 줄 요약)

👉 개발자가 코드 수정 → Jenkins가 이미지 빌드 → Docker Hub 업로드 → infra-repo 수정 → ArgoCD가 감지해서 쿠버네티스 배포
