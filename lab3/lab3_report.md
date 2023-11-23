University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4113c  
Author: Iumadilova Angela\
Lab: Lab3 
Date of create: 21.11.2023  
Date of finished: 23.11.2023
___
# Отчёт о лабораторной работе №3

## Сертификаты и "секреты" в Minikube, безопасное хранение данных.

### Цель работы

Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.
### Ход работы

#### Создание манифеса
Создадим манифест для объявления configMap, в котором объявим две переменных `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME`, также для replicaSet и укажем заданные по заданию параметры, и манифест для сервиса. Число реплик - две, образ контейнера - ifilyaninitmo/itdt-contained-frontend:master, порт контейнера - 3000. С помощью секции `env` передадим в контейнер переменные окружения, которые подставляются из созданного нами ранее configMap:
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmaplab3
data:
  REACT_APP_USERNAME: Angela
  REACT_APP_COMPANY_NAME: ITMO

---

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: lab3replica
  labels:
    app: lab3
spec:
  replicas: 2
  selector: 
    matchLabels:
      app: lab3
  template:
    metadata:
      labels:
        app: lab3
    spec:
      containers:
      - name: lab3
        image: ifilyaninitmo/itdt-contained-frontend:master
        env:
        - name: REACT_APP_USERNAME
          valueFrom:
            configMapKeyRef:
              name: configmaplab3
              key: REACT_APP_USERNAME
        - name: REACT_APP_COMPANY_NAME
          valueFrom:
            configMapKeyRef:
              name: configmaplab3
              key: REACT_APP_COMPANY_NAME
        ports:
        - containerPort:  3000

---

apiVersion: v1
kind: Service
metadata:
  name: lab3replica
spec:
  type: NodePort
  selector:
    app: lab3
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
```
Применим манифест:
```bash
$ kubectl apply -f ./lab3/lab3.yaml
configmap/configmaplab3 created
replicaset.apps/lab3replica created
service/lab3replica created
```

Проверим работу реплики:
```bash
$ kubectl port-forward services/lab3replica 3000:3000
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```
Перейдём в браузер:

#### Создание Ingress
Установим плагин Ingress в minikube:
```bash
$ minikube addons enable ingress

💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
💡  After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
    ▪ Using image registry.k8s.io/ingress-nginx/controller:v1.8.1
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```
Сгенерируем TLS сертификат и импортируем его в minikube, добавив также в Ingress:
```bash
$ openssl req -new -newkey rsa:4096 -x509 -sha256 -days 5 -nodes -out lab3.crt -keyout

$ kubectl create secret tls secret-tls --key="lab3.key" --cert="lab3.crt"
secret/secret-tls created
```
Теперь можно создать манифест для Ingress, указав в нём FQDN и TLS-сертификат. Также указываем сервис, который мы создали для доступа в реплику:
```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lab3-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - lab3.local
    secretName: tls-secret
  rules:
  - host: lab3.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: lab3replica
            port:
              number: 3000
```
После сохранения манифеста создадим Ingress:
```bash
$ kubectl apply -f ./lab3/ingress.yaml
ingress.networking.k8s.io/lab3-ingress created
```
#### Проверка сертификата

Прописываем в hosts (C:\Windows\System32\Drivers\etc\hosts) FQDN и IP адрес : 127.0.0.1 lab3.local

Теперь, вбивая в браузере наше FQDN, мы попадаем в веб-приложение. Подключение при этом производится по протоколу HTTPS:
![image1](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/9514b570-f376-4514-850e-af9f8f9e5e9f)


Убедимся, что выдаётся наш сертификат:
![image2](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/c4b47b55-ff56-46a2-ab63-2a12c8633a97)

Таким образом, нам удалось настроить Ingress на работу с созданным нами сервисом, используя FQDN и TLS-сертификат. 

### Схема организации контейнеров и сервисов


