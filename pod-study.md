# 🐳 Kubernetes Study - Pod

이 문서는 pod 실습 과정을 정리한 노트입니다.  

---
# 1. How many pods exist on the system?
## In the current(default) namespace.

👉
```bash
kubectl get pods -n default
```
---
# 2. Create a new pod using the nginx image.

👉
```bash
kubectl run nginx --image=nginx
```
---
# 3. How many pods are created now?

👉
```bash
kubectl get pods -n default
```
---
# 4. Which image is specified for the pods whose names begin with the newpods- prefix?

👉
```bash
   kubectl describe pod newpods-59hcp
```
 

---
# 5. Which nodes are these pods placed on?

👉  
```bash
kubectl get pods -owide
```
---
# 6. We just created a new pod named webapp. How many containers are part of the webapp pod?
---

👉  
```bash
kubectl kubectl describe pod webapp
```


---
# 7. What images are used in the new webapp pod?
---

👉  
```bash
kubectl kubectl describe pod webapp
```


---
# 8. What is the state of the container agentx in the pod webapp?
---

👉  
```bash
kubectl describe pod webapp
```


---

# 9. Why do you think the container agentx in pod webapp is in error?
---

👉  
```bash
kubectl describe pod webapp
```


---
# 10. What does the READY column in the output of the kubectl get pods command indicate?
---

👉  
```bash
kubectl get pods
```
📌Ready containers in pod / Total containers in pod

---
# 11. Delete the webapp Pod.
---

👉  
```bash
kubectl delete pod webapp
```
---
# 12. Create a new pod with the name redis and the image redis123.
Utilize a pod-definition YAML file. Please note that the image name redis123 is intentionally incorrect. Do NOT correct the image at this stage; you will address this in the subsequent task.
---

## 📌 Step 1
👉 We use kubectl run command with --dry-run=client -o yaml option to create a manifest file :-
```bash
kubectl run redis --image=redis123 --dry-run=client -o yaml > redis-definition.yaml
```
## 📌 Step 2
👉 After that, using kubectl create -f command to create a resource from the manifest file :-
```bash
kubectl create -f redis-definition.yaml
```
## 📌 Step 3
👉 After that, using kubectl create -f command to create a resource from the manifest file :-
```bash
kubectl get pods
```
---
# 13. Now change the image on this pod to redis.
Once done, the pod should be in a running state.
---

## 📌 Step 1
👉 Use the kubectl edit command to update the image of the pod to redis.
```bash
kubectl edit pod redis
```
## 📌 Step 2
👉 If you used a pod definition file then update the image from redis123 to redis in the definition file via Vi or Nano editor and then run kubectl apply command to update the image :-
```bash
kubectl apply -f redis-definition.yaml
```
---
