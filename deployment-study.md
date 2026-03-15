# 🐳 Kubernetes Study - deployment

이 문서는 deployment 실습 과정을 정리한 노트입니다.  

---
# 1. How many PODs exist on the system?

In the current(default) namespace.

👉
```bash
kubectl get pods -n default
```
---
# 2. How many ReplicaSets exist on the system?

In the current(default) namespace.

👉
```bash
kubectl get rs -n default
```
---
# 3. How many Deployments exist on the system?

In the current(default) namespace.

👉
```bash
kubectl get deployments -n default
```
---
# 4. How many Deployments exist on the system now?

We just created a Deployment! Check again!

👉
```bash
   kubectl get deployments -n default
```
 

---
# 5. How many ReplicaSets exist on the system now?

👉  
```bash
kubectl get rs
```
---
# 6. How many PODs exist on the system now?
---

👉  
```bash
kubectl get pods
```


---
# 7. Out of all the existing PODs, how many are ready?
---

👉  
```bash
kubectl get pods
```


---
# 8. What is the image used to create the pods in the new deployment?
---

👉  
```bash
kubectl describe deployments
```


---

# 9. Why do you think the deployment is not ready?
---

👉  
```bash
kubectl describe pod webapp
```


---
# 10. Create a new Deployment using the deployment-definition-1.yaml file located at /root/.


There is an issue with the file, so try to fix it.
---

👉  
```bash
---
apiVersion: apps/v1 
kind: Deployment ##📌 'Deployment'로 수정
metadata:
  name: deployment-1 
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - name: busybox-container
        image: busybox
        command:
        - sh
        - "-c"
        - echo Hello Kubernetes! && sleep 3600
```


---
# 11. Create a new Deployment with the below attributes using your own deployment definition file.


Name: httpd-frontend;
Replicas: 3;
Image: httpd:2.4-alpine
---

👉  
```bash
kubectl create deployment httpd-frontend --replicas=3 --image=httpd:2.4-alpine --dry-run=client -oyaml >> httpd.yaml
```
---
