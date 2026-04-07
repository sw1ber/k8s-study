1️⃣ WSL2 설치 (리눅스 환경 만들기)
1. PowerShell 관리자 실행
wsl.exe --install -d Ubuntu-22.04

WSL2 활성화
Ubuntu 자동 설치됨

✅ 설치 확인
wsl -l -v

✅ Ubuntu 최초 실행
wsl  
id pw 생성(swiber, 1234)

2️⃣ Docker Desktop 설치

Docker Desktop 실행 → Settings → WSL Integration

✔ 체크해야 할 것:

✅ “Enable integration with my default WSL distro”
✅ Ubuntu 체크

3️⃣ k3d 설치 (WSL 안에서)

✅ 설치 명령어

WSL(Ubuntu) 안에서 실행:
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

✅ 설치 확인
k3d version

-------------------------------------------------------------------------------------

1️⃣ k3d 클러스터 생성

WSL(Ubuntu)에서 실행:

k3d cluster create cicd-cluster \
  -p "8081:80@loadbalancer"

✅ 확인
kubectl get nodes

👉 결과:

k3d-cicd-cluster-server-0   Ready
 ArgoCD 설치 (핵심🔥)

2️⃣ 👉 Argo CD

설치
kubectl create namespace argocd

kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

3️⃣ ArgoCD 접속 설정
NodePort로 열기
kubectl patch svc argocd-server -n argocd \
-p '{"spec": {"type": "NodePort"}}'
포트 확인
kubectl get svc -n argocd

👉 port-forward 사용(백그라운드)
kubectl port-forward svc/argocd-server -n argocd 8080:443 &

👉 그 다음 접속:

https://localhost:8080

NodePort가 Docker 내부 네트워크에만 열리기 때문

초기 비밀번호 확인
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d

id admin 
pw 4RrQawRRFbp2BuG1

-------------------------------------------------------------------------------------

1️⃣ Jenkins 설치

Helm 설치 (필수)

👉 Helm

WSL(Ubuntu)에서 실행 👇

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

✅ Helm으로 설치 
kubectl create namespace jenkins

helm repo add jenkins https://charts.jenkins.io
helm repo update

helm install jenkins jenkins/jenkins -n jenkins

접속(포트포워딩)

kubectl port-forward svc/jenkins -n jenkins 18080:8080 &

✅ admin password
kubectl get secret jenkins -n jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 -d

id admin 
pw ExK2p6jgif2u9Zt6pF7lYz

-------------------------------------------------------------------------------------

2️⃣ Git Repo 준비 

👉 repo 2개 구조 추천

1. app-repo (코드)
2. infra-repo (k8s yaml)
📦 예시 구조
app-repo
app/
 ├── app.py
 └── Dockerfile
infra-repo
k8s/
 └── deployment.yaml
📄 deployment.yaml 예시
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
        image: dockerhub아이디/myapp:latest
        ports:
        - containerPort: 5000
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


