<h1> Opensearch - Study </h1>

1️⃣ Helm 설치 확인 <br>

 ✅ WSL:
 ```bash
   helm version
 ```
2️⃣ namespace 생성 <br>

 ```bash
   kubectl create namespace logging
 ```
3️⃣ Helm repo 추가

 ```bash
   helm repo add opensearch https://opensearch-project.github.io/helm-charts/
   helm repo update
 ```
4️⃣ OpenSearch 설치

 ```bash
 helm install opensearch opensearch/opensearch -n logging --set image.tag=2.19.1 --set replicas=1 --set persistence.enabled=false --set opensearchJavaOpts="-Xms512m -Xmx512m" --set extraEnvs[0].name=OPENSEARCH_INITIAL_ADMIN_PASSWORD --set-string extraEnvs[0].value='Ehduq5114*'                                                       
 ```

 # PVC 없이 테스트용
 # JVM 512m(메모리 절약)
 # 단일 노드
 # Admin password Ehduq5114* 설정


5️⃣ 상태 확인

 ```bash
  kubectl get pods -n logging
  kubectl get svc -n logging
 ```
6️⃣ 포트포워딩

```bash
kubectl port-forward svc/opensearch-cluster-master -n logging 9200:9200 &
curl -k -u admin:Ehduq5114* https://localhost:9200
```
7️⃣ Dashboards 설치

```bash
helm install opensearch-dashboards opensearch/opensearch-dashboards -n logging --set image.tag=2.19.1
```

8️⃣ Dashboards 포트포워딩

kubectl port-forward svc/opensearch-dashboards -n logging 5601:5601 &
http://localhost:5601

---------------------------------------------------------------------------

1️⃣ Fluentd Bit 설치 준비

```bash
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update
```
2️⃣ Fluentd values.yaml 생성

 ```bash
mkdir fluentd
cd fluentd
vi values.yaml
```

```bash
daemonSetVolumes:
  - name: varlog
    hostPath:
      path: /var/log

daemonSetVolumeMounts:
  - name: varlog
    mountPath: /var/log

config:
  service: |
    [SERVICE]
        Flush        1
        Daemon       Off
        Log_Level    info
        HTTP_Server   On
        HTTP_Listen   0.0.0.0
        HTTP_PORT     2020

  inputs: |
    [INPUT]
        Name              tail
        Path              /var/log/containers/*.log
        Parser            docker
        Tag               kube.*
        Refresh_Interval  5

  filters: |
    [FILTER]
        Name                kubernetes
        Match               kube.*
        Merge_Log           On
        Keep_Log            Off

  outputs: |
    [OUTPUT]
        Name              es
        Match             *
        Host              opensearch-cluster-master.logging.svc.cluster.local
        Port              9200
        HTTP_User         admin
        HTTP_Passwd       Ehduq5114*
        tls               On
        tls.verify        Off
        Index             fluentbit
        Suppress_Type_Name On
 ```
opensearch-cluster-master
→ Kubernetes Service 이름
logging
→ namespace
svc.cluster.local
→ Kubernetes 내부 DNS 도메인

logging 네임스페이스의 opensearch-cluster-master 서비스로 접속

Kubernetes 내부 Pod끼리 통신할 때 사용하는 주소.

3️⃣ Fluentd-bit 설치

 ```bash
 helm install fluent-bit fluent/fluent-bit -n logging -f values.yaml
 ```
4️⃣ Fluentd-bit 상태 확인

 ```bash
kubectl get pods -n logging
```
5️⃣ Fluentd-bit 로그 확인 

 ```bash
kubectl logs -n logging -l app.kubernetes.io/name=fluent-bit
 ```
6️⃣ OpenSearch 인덱스 확인

```bash
https://localhost:9200  
```
7️⃣ Dashboards에서 로그 보기

화면 왼쪽:

OpenSearch Dashboards
 └ Discover

Create index pattern

입력:

fluentbit*

@timestamp

Discover에서 로그 확인

8️⃣ 실제 로그 테스트

app.py 수정 

```bash 
from flask import Flask
import time

app = Flask(__name__)

@app.route("/")
def home():
    print("REQUEST RECEIVED")
    return "Hello from OpenSearch logging!"

@app.route("/error")
def error():
    print("ERROR TEST")
    return "error route"

if __name__ == "__main__":
    print("APP START")
    app.run(host="0.0.0.0", port=5000)
```    
Discover 검색창에서

REQUEST RECEIVED

