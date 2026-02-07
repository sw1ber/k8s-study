# 🐳 Kubernetes Study - Install VPA

이 문서는 Install VPA 실습 과정을 정리한 노트입니다.  

---
# 1. Introduction to Kubernetes Vertical Pod Autoscaler (VPA)
---

## 📌 Step 1: Install VPA Custom Resource Definitions (CRDs)
👉 These CRDs allow Kubernetes to recognize the custom resources that VPA uses to function properly. To install them, run this command:
```bash
kubectl apply -f /root/vpa-crds.yml
```
## 📌 Step 2: Install VPA Role-Based Access Control (RBAC)
👉 RBAC ensures that VPA has the appropriate permissions to operate within your Kubernetes cluster. To install the RBAC settings, run:
```bash
kubectl apply -f /root/vpa-rbac.yml
kubectl get crd
```
---
# 2. Clone the VPA Repository and Set Up the Vertical Pod Autoscaler
---
## 📌 Step 1: Clone the repository:
👉 First, navigate to the /root directory and clone the repository:
```
git clone https://github.com/kubernetes/autoscaler.git
```
## 📌 Step 2: Navigate to the Vertical Pod Autoscaler directory:
👉 After cloning, move into the vertical-pod-autoscaler directory:
```bash
   cd autoscaler/vertical-pod-autoscaler
```

## 📌 Step 3: Run the setup script:
👉 Execute the provided script to deploy the Vertical Pod Autoscaler:
```bash
      ./hack/vpa-up.sh
```
---
# 3. Which of the following are the VPA CRDs that get installed as part of the Vertical Pod Autoscaler setup?
---
 1 . verticalpodresources.autoscaling.k8s.io & vpa-memory.cpu.k8s.io

 2 . verticalpodautoscaler.autoscaling.k8s.io & horizontalpodautoscalers.autoscaling.k8s.io

 3 . verticalpodautoscalers.autoscaling.k8s.io &verticalpodautoscalercheckpoints.autoscaling.k8s.io

 4 . vpa-deployments.autoscaling.k8s.io & vpa-pods.autoscaling.k8s.io

👉 A. 3 . verticalpodautoscalers.autoscaling.k8s.io &verticalpodautoscalercheckpoints.autoscaling.k8s.io

---
# 4. How many VPA deployments typically run in the kube-system namespace after installation?
---
 1 . 9

 2 . 3

 3 . 6

 4 . 4

```bash
   kubectl get deployments -n kube-system | grep vpa
```
👉 A. 2 . 3

---
# 5. You are given a Kubernetes deployment file named flask-app.yml located in the /root directory. Your task is to:
Deploy the flask-app.yml file to the Kubernetes cluster
After deployment, check the logs of the Vertical Pod Autoscaler(VPA) updater to ensure it is functioning correctly
Note: The pod may take some time to reach the running state.
---
 1 . Have you done the deployment with the flask-app name?

 2 . Is there a pod with the name flask-app-XXXX?

 3 . Is the pod named flask-app-xxxx is running?

👉 A. 
```bash
   cd
   kubectl apply -f flask-app.yaml
   kubectl describe pod flask-app-67b666c5fc-tv77m
```
---
# 6. You have recently deployed a Flask application to your Kubernetes cluster. However, the Vertical Pod Autoscaler (VPA) vpa-updater-XXXX pod indicates that there may be an issue with the newly deployed flask-app pods.
---
Inspect the logs of the vpa-updater-XXXX pod and observe the following message:

Check the logs of the vpa-updater-XXXX pod to identify any potential issues with the flask-app deployment.
When checking the logs, you see the following error message:

pods_eviction_restriction.go:226] **too few replicas** for **ReplicaSet** default/**flask-app-b6c9c4f78**

Problem Analysis:

Flask application is running with only 1 replica pod.
The Vertical Pod Autoscaler (VPA) needs to evict (remove) the existing pod to create a new one with updated resource settings.
Kubernetes has a safety feature that prevents removing the last pod of a deployment to avoid service downtime.
When you have only 1 replica and VPA tries to evict it, Kubernetes blocks this action with the error message: "too few replicas".
VPA wants to optimize your pod's resources but cannot because Kubernetes is protecting your service availability.
As a result, VPA cannot apply its resource recommendations, and application cannot benefit from automatic resource optimization.

---
👉 Approach to Resolve the Issue:
1. Increase the replica count:
```bash
kubectl scale deployment flask-app --replicas=2
```
2. Verify the Deployment:
```
kubectl get deployment flask-app -owide
```
3. Check the Pod Status:
```
kubectl get pods -l app=flask-app
```
4. Verify VPA operation:
```
kubectl describe vpa flask-app
```
---