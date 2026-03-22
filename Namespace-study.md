# 🐳 Kubernetes Study - namespace

이 문서는 namespace 실습 과정을 정리한 노트입니다.  

---
# 1. How many Namespaces exist on the system?



```bash
kubectl get namespace | wc -l 
```
👉 10 (헤더 제외 )

---
# 2. How many pods exist in the research namespace?

```bash
kubectl get pods -n research
```

👉 2

---
# 3. Create a POD in the finance namespace.


Use the spec given below:

Name: redis
Image name: redis

```bash
kubectl create deployment redis --image=redis --namespace=finance --dry-run=client -oyaml >> redis.yaml
```



---
# 4. Which namespace has the blue pod in it?

```bash
   kubectl get pods --all-namespaces | grep blue
```
 
👉 marketing

---
# 5. Access the Blue web application using the link above your terminal!!

From the UI you can ping other services.

👉  
```bash
Blue - Marketing Application
```
---
# 6. What is the shortest DNS name the Blue application should use to access the database redis-db-service in its own namespace - marketing?

You can try it in the web application UI. Use port 6379.

---

 
```bash
NAMESPACE       NAME                                      READY   STATUS             RESTARTS        AGE
default         redis-7c84479b6c-gp4bf                    1/1     Running            0               2m50s
dev             redis-db                                  1/1     Running            0               9m41s
finance         payroll                                   1/1     Running            0               9m41s
kube-system     coredns-7896679cc-fbkl4                   1/1     Running            0               17m
kube-system     helm-install-traefik-crd-kn6fh            0/1     Completed          0               17m
kube-system     helm-install-traefik-nx99g                0/1     Completed          1               17m
kube-system     local-path-provisioner-578895bd58-5xnsp   1/1     Running            0               17m
kube-system     metrics-server-7b9c9c4b9c-q4qpv           1/1     Running            0               17m
kube-system     svclb-traefik-42e9b1d0-n6gjv              2/2     Running            0               17m
kube-system     traefik-6f986b958c-9rnj4                  1/1     Running            0               17m
manufacturing   red-app                                   1/1     Running            0               9m42s
marketing       blue                                      1/1     Running            0               9m41s
marketing       redis-db                                  1/1     Running            0               9m41s
research        dna-1                                     0/1     CrashLoopBackOff   6 (3m32s ago)   9m42s
research        dna-2                                     0/1     CrashLoopBackOff   6 (3m46s ago)   9m42s
```
👉 같은 namespace기 때문에 redis-db-service 


---
# 7. What DNS name should the Blue application use to access the database redis-db-service in the dev namespace?

You can try it in the web application UI. Use port 6379.
In the current(default) namespace

---
  
```bash
controlplane ~ ✖ kubectl get pods --all-namespaces 
NAMESPACE       NAME                                      READY   STATUS             RESTARTS        AGE
default         redis-7c84479b6c-gp4bf                    1/1     Running            0               7m47s
dev             redis-db                                  1/1     Running            0               14m
finance         payroll                                   1/1     Running            0               14m
kube-system     coredns-7896679cc-fbkl4                   1/1     Running            0               22m
kube-system     helm-install-traefik-crd-kn6fh            0/1     Completed          0               22m
kube-system     helm-install-traefik-nx99g                0/1     Completed          1               22m
kube-system     local-path-provisioner-578895bd58-5xnsp   1/1     Running            0               22m
kube-system     metrics-server-7b9c9c4b9c-q4qpv           1/1     Running            0               22m
kube-system     svclb-traefik-42e9b1d0-n6gjv              2/2     Running            0               22m
kube-system     traefik-6f986b958c-9rnj4                  1/1     Running            0               22m
manufacturing   red-app                                   1/1     Running            0               14m
marketing       blue                                      1/1     Running            0               14m
marketing       redis-db                                  1/1     Running            0               14m
research        dna-1                                     0/1     CrashLoopBackOff   7 (3m20s ago)   14m
research        dna-2                                     0/1     CrashLoopBackOff   7 (3m40s ago)   14m
```
👉 다른 namespace기 때문에 redis-db-service.dev.svc.cluster.local

혹은 redis-db-service.dev

---
