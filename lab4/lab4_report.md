University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4113c  
Author: Iumadilova Angela\
Lab: Lab4
Date of create: 03.12.2023  
Date of finished: 04.12.2023

___
# Отчёт о лабораторной работе №4

## Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS"

### Цель работы

Познакомиться с CNI Calico и функцией IPAM Plugin, изучить особенности работы CNI и CoreDNS.

### Ход работы

1) Запуск Minikube с плагином CNI Calico и в режиме Multi-Node Clusters: 
```bash
$ minikube start --network-plugin=cni --cni=calico --nodes 2
W1204 00:51:18.537721    7768 main.go:291] Unable to resolve the current Docker CLI context "default": context 
"default": context not found: open C:\Users\Анжела\.docker\contexts\meta\37a8eec1ce19687d132fe29051dca629d164e2c4958ba141d5f4133a33f0688f\meta.json: The system cannot find the path specified.
😄  minikube v1.31.2 on Microsoft Windows 10 Home 10.0.19045.3693 Build 19045.3693
✨  Automatically selected the docker driver
❗  With --network-plugin=cni, you will need to provide your own CNI. See --cni flag as a user-friendly alternat
ive
📌  Using Docker Desktop driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring Calico (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner

👍  Starting worker node minikube-m02 in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.49.2
    ▪ no_proxy=192.168.49.2
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
    ▪ env NO_PROXY=192.168.49.2
🔎  Verifying Kubernetes components...
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
2) Проверка работы CNI плагина Calico и количество нод командами:
```bash
$ kubectl get nodes
NAME           STATUS   ROLES           AGE    VERSION
minikube       Ready    control-plane   2m7s   v1.27.4
minikube-m02   Ready    <none>          12s    v1.27.4

$ kubectl get pods -n kube-system -l k8s-app=calico-node
NAME                READY   STATUS     RESTARTS   AGE
calico-node-b6dxq   0/1     Init:0/3   0          33s
calico-node-tk44n   0/1     Init:2/3   0          119s
```

Для проверки IPAM (IP Address Management), назначим метки узлам. Присвоем метку географического расположения: location=us-east и location=us-west, для соответсвующих нод.
```bash
$ kubectl label nodes minikube rack=c1
node/minikube labeled
$ kubectl label nodes minikube-m02 rack=c2
node/minikube-m02 labeled
```

Убедимся, что Calico выдал диапазон по умолчанию:
```bash
$ kubectl calico get ippool -o wide --allow-version-mismatch
NAME                  CIDR            NAT    IPIPMODE   VXLANMODE   DISABLED   DISABLEBGPEXPORT   SELECTOR   
default-ipv4-ippool   10.244.0.0/16   true   Always     Never       false      false              all()
```
Удалим диапазон:

```bash
$ kubectl calico delete ippools default-ipv4-ippool --allow-version-mismatch
Successfully deleted 1 'IPPool' resource(s)
```

Создадим манифест для Calico, определяющий IP-пулы в соответствии с метками узлов: 
```yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: rack1-pool
spec:
  cidr: 10.0.0.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: rack == "c1"
---
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: rack2-pool
spec:
  cidr: 10.244.2.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: rack == "c2"
```

Создадим IP пулы и проверим диапазоны:
```bash
$ kubectl calico apply -f ippool.yaml --allow-version-mismatch
Successfully applied 2 'IPPool' resource(s)

$ kubectl calico get ippool -o wide --allow-version-mismatch
NAME         CIDR          NAT    IPIPMODE   VXLANMODE   DISABLED   DISABLEBGPEXPORT   SELECTOR       
rack1-pool   10.0.0.0/24   true   Always     Never       false      false              rack == "c1"
rack2-pool   10.244.2.0/24   true   Always     Never       false      false              rack == "c2"
```
Создадим манифест для Deployment с необходимыми переменными окружения (лаб3): 
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmaplab4
data:
  REACT_APP_USERNAME: Angela
  REACT_APP_COMPANY_NAME: ITMO

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: lab4
  labels:
    app: lab4
spec:
  replicas: 2
  selector: 
    matchLabels:
      app: lab4
  template:
    metadata:
      labels:
        app: lab4
    spec:
      containers:
      - name: lab4
        image: ifilyaninitmo/itdt-contained-frontend:master
        env:
        - name: REACT_APP_USERNAME
          valueFrom:
            configMapKeyRef:
              name: configmaplab4
              key: REACT_APP_USERNAME
        - name: REACT_APP_COMPANY_NAME
          valueFrom:
            configMapKeyRef:
              name: configmaplab4
              key: REACT_APP_COMPANY_NAME
        ports:
        - containerPort:  3000

---

apiVersion: v1
kind: Service
metadata:
  name: lab4deploy
spec:
  type: NodePort
  selector:
    app: lab4
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
```
Применим:
```bash
$ kubectl apply -f lab4.yaml
configmap/configmaplab4 created
deployment.apps/lab4 created
service/lab4deploy created
```

Пробрасываем порты сервиса на локальную машину: 
```bash
$ kubectl port-forward service/lab4deploy 3000:3000
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```

Получим IP контейнеров:
```bash
$ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
lab4-6d7647697c-ft8cq   1/1     Running   0          102s   10.244.2.0   minikube-m02   <none>           <none>
lab4-6d7647697c-jhwqf   1/1     Running   0          102s   10.0.0.65    minikube       <none>           <none>
```
Зайдем в один из контейнеров и пропингуем другой:
```bash
$ kubectl exec -it lab4-6d7647697c-ft8cq sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/frontend # ping 10.0.0.65
PING 10.0.0.65 (10.0.0.65): 56 data bytes
64 bytes from 10.0.0.65: seq=0 ttl=62 time=1.202 ms
64 bytes from 10.0.0.65: seq=1 ttl=62 time=0.164 ms
64 bytes from 10.0.0.65: seq=2 ttl=62 time=0.155 ms
^C
--- 10.0.0.65 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.155/0.507/1.202 ms
/frontend # 
```
___

### Схема


