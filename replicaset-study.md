# 🐳 Kubernetes Study - ReplicaSets

이 문서는 Kubernetes ReplicaSet 개념과 실습 과정을 정리한 노트입니다.  
(`kubectl` 명령어와 함께 실제 상황에서의 동작을 학습)

---

## 📌 1. How many ReplicaSets exist on the system?
👉 현재(default) 네임스페이스에서 ReplicaSet 개수 확인
```bash
kubectl get rs -n <네임스페이스>
kubectl describe rs <replicaset-name>
kubectl get rs

## 📌 2. How about now? How many ReplicaSets do you see?
👉 변경 후 다시 ReplicaSet 개수 확인
```bash
kubectl get rs

## 📌 3. How many PODs are DESIRED in the new-replica-set?
```bash
kubectl get rs

## 📌 4. What is the image used to create the pods in the new-replica-set?
```bash
kubectl describe rs new-replica-set

## 📌 5. How many PODs are READY in the new-replica-set?
```bash
kubectl get rs

## 📌 6. Why do you think the PODs are not ready?
👉 Pod 상태 확인
```bash
kubectl get pods

## 📌 7. Delete one of the four PODs directly
👉 Pod 직접 삭제
```bash
kubectl get pods

## 📌 8. Why are there still 4 PODs, even after deletion?
👉 ReplicaSet은 원하는 수(Desired)를 보장하기 위해 자동으로 새 Pod를 생성합니다.

## 📌 9. Create a ReplicaSet from replicaset-definition-1.yaml
```bash
kubectl apply -f /root/replicaset-definition-1.yaml
⚠️ 주의: ReplicaSet은 apps/v1 API 그룹에 속함.
(v1은 Pod, Service 같은 Core 리소스에서만 사용)

## 📌 10. Fix & create ReplicaSet from replicaset-definition-2.yaml

파일 경로: /root/replicaset-definition-2.yaml
수정 포인트: selector와 template 라벨 일치 필요

```yaml 
tier: frontend   # selector와 동일하게 수정

## 📌 11. Delete the two newly created ReplicaSets

kubectl delete rs replicaset-1
kubectl delete rs replicaset-2

## 📌 12. Fix the original ReplicaSet new-replica-set
👉 올바른 busybox 이미지를 사용하도록 수정

방법 1) ReplicaSet 삭제 후 수정된 yaml 재적용
kubectl get rs new-replica-set -o yaml
kubectl delete rs new-replica-set
kubectl apply -f /root/new-replica-set.yaml

방법 2) ReplicaSet 수정 후 기존 Pod 삭제 → 올바른 이미지로 자동 재생성

## 📌 13. Scale the ReplicaSet to 5 Pods
```bash
kubectl scale rs new-replica-set --replicas=5

## 📌 14. Scale down the ReplicaSet to 2 Pods
```bash
kubectl scale rs new-replica-set --replicas=2

