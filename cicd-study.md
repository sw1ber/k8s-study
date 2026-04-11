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

<h3>3. Jenkins가 실제로 이미지를 빌드할 수 있게 만들기</h3>

가장 중요하다. 지금 Jenkins는 Kubernetes 안의 Pod로 떠 있고, 그냥 기본 설치만으로는 바로 docker build가 안 될 수 있다. Jenkins Pipeline은 Docker 기반 실행 환경을 잘 지원하지만, 실제 빌드를 하려면 Jenkins Pod에서 Docker CLI/엔진 접근이 되거나, 별도의 빌드 도구가 필요하다.

실습용으로 제일 쉬운 방법은 Jenkins Pod에 Docker socket을 연결해서 docker build를 쓰는 방식이다. 운영 환경에서는 권장되지 않지만, 지금 같은 로컬 학습용에서는 제일 빠르다.

그래서 먼저 확인:
```bash
kubectl exec -n jenkins deploy/jenkins -- docker version
```
여기서 docker 명령이 없거나 접속 실패하면 Jenkins chart values를 손봐야 한다. <br>
실습용으로는 크게 두 가지 중 하나다.

+Jenkins 컨테이너에 docker CLI 추가 <br>
+/var/run/docker.sock 마운트

<h3> 4. app-repo 만들기 </h3>

예시로 파이썬 Flask 앱 하나 만들기

[app.py]
```bash
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from Jenkins + ArgoCD!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

[requirements.txt]
```bash
flask==3.0.3
```

[Dockerfile]
```bash
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```
<h3> 5. infra-repo deployment.yaml 바꾸기 </h3>

infra-repo/k8s/deployment.yaml 수정 

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
service.yaml도 포트를 5000으로 맞춘다.
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
<h3> 6. Jenkinsfile 만들기 </h3>

app-repo/Jenkinsfile

```bash
pipeline {
  agent any

  environment {
    IMAGE_NAME = "sw1ber/myapp"
    IMAGE_TAG  = "${env.BUILD_NUMBER}"
    APP_REPO   = "https://github.com/sw1ber/app-repo.git"
    INFRA_REPO = "https://github.com/sw1ber/infra-repo.git"
  }

  stages {
    stage('Checkout App Repo') {
      steps {
        checkout scm
      }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
      }
    }

    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${IMAGE_NAME}:${IMAGE_TAG}
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
            rm -rf infra-repo
            git clone https://${GIT_USER}:${GIT_PASS}@github.com/sw1ber/infra-repo.git
            cd infra-repo

            sed -i 's#image: .*#image: '"${IMAGE_NAME}:${IMAGE_TAG}"'#' k8s/deployment.yaml

            git config user.name "jenkins"
            git config user.email "jenkins@local"
            git add k8s/deployment.yaml
            git commit -m "Update image to ${IMAGE_NAME}:${IMAGE_TAG}" || true
            git push origin main
          '''
        }
      }
    }
  }
}
```

위 Jenkinsfile이 하는 일은 이거다.

앱 소스 checkout 
Docker 이미지 빌드<br>
Docker Hub push<br>
infra-repo clone<br>
deployment.yaml의 image: 줄 교체<br>
GitHub push

그리고 나면 ArgoCD가 그 변경을 자동 감지해서 배포한다. 이게 바로 Argo CD 자동 동기화가 권장하는 CI/CD 방식이다.

<h3> 7. Jenkins Job 만들기 </h3>
Jenkins에서: <br>
New Item <br>
Pipeline <br>
이름: myapp-cicd

설정에서: <br>
Pipeline script from SCM <br>
Git repo URL: https://github.com/sw1ber/app-repo.git <br>
Branch: main <br>
Script Path: Jenkinsfile <br>
저장 후 Build Now.

<h3> 8. 성공하면 어디서 확인? </h3>
Jenkins 콘솔 로그에서:

docker build 성공 <br>
docker push 성공 <br>
infra-repo push 성공 <br>

그다음 GitHub의 infra-repo/k8s/deployment.yaml에서 image 태그가 바뀐 걸 확인.

그 다음 ArgoCD 앱 infra-cicd에서:
잠깐 OutOfSync <br>
자동으로 Sync <br>
Healthy 

이 흐름이 보여야 정상이다.

<h3> 9. 배포된 앱 확인 </h3>

서비스 포트포워딩:
```bash
kubectl port-forward svc/myapp-svc 8082:80
```
브라우저:
```bash
http://localhost:8082
```