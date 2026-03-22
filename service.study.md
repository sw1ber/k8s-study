# 🐳 Kubernetes Study - service

이 문서는 service 실습 과정을 정리한 노트입니다.  

---
# 1. How many Services exist on the system?

In the current(default) namespace
👉
```bash
kubectl get services
```
---
# 2. That is a default service created by Kubernetes at launch.


# 3. What is the type of the default kubernetes service?

```bash
kubectl get services
```

👉 ClusterIP

---
# 4. What is the targetPort configured on the kubernetes service?

```bash
   kubectl describe service
```
 
👉 6443

---
# 5. How many labels are configured on the kubernetes service?

👉  
```bash
kubectl describe service
```
---
# 6. How many Endpoints are attached on the kubernetes service?
---

👉  
```bash
kubectl describe service
```


---
# 7. How many Deployments exist on the system now?
In the current(default) namespace

---
  
```bash
kubectl get deployments
```
👉 1

---
# 8. What is the image used to create the pods in the deployment?
---

 
```bash
kubectl describe deployments
```
👉 kodekloud/simple-webapp:red

---

# 9. Can you currently access the Web App UI through the tab?

Try to access the Web Application UI using the tab simple-webapp-ui above the terminal.
---

👉  
```bash
502 Bad Gateway
nginx/1.27.4
```


---
# 10. Create a new service to access the web application using the service-definition-1.yaml file.



Name: webapp-service
Type: NodePort
targetPort: 8080
port: 8080
nodePort: 30080
selector:
  name: simple-webapp


There is an issue with the file, so try to fix it.

---

👉  
```bash
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: default
spec:
  ports: 
  - nodePort: 30080
    port: 8080
    targetPort: 8080
  selector:
    name: simple-webapp
  type: NodePort
```


---
# 11. Access the web application using the tab simple-webapp-ui above the terminal window.



---

👉  
```bash
Hello from simple-webapp-deployment-5cf5f46dc7-l6sbc!
```
---
