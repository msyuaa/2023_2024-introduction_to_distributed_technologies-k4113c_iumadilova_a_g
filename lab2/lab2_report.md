University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4113c  
Author: Iumadilova Angela\
Lab: Lab2 
Date of create: 20.11.2023  
Date of finished: 21.11.2023


## Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."

### Описание

В данной лабораторной работе вы познакомитесь с развертыванием полноценного веб сервиса с несколькими репликами.

### Цель работы

Ознакомиться с типами "контроллеров" развертывания контейнеров, ознакомится с сетевыми сервисами и развернуть свое веб приложение.

### Ход работы

- Создаем `deployment` с 2 репликами контейнера и передать переменные в эти реплики: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` с помощью манифеста `lab2.yaml`. В этом же файле пропишем данные для сервиса `lab2deployment`, тип - NodePort.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lab2deployment
  labels:
    app: lab2
spec:
  replicas: 2
  selector: 
    matchLabels:
      app: lab2
  template:
    metadata:
      labels:
        app: lab2
    spec:
      containers:
      - name: lab2
        image: ifilyaninitmo/itdt-contained-frontend:master
        env:
        - name: REACT_APP_USERNAME
          value: Angela
        - name: REACT_APP_COMPANY_NAME
          value: ITMO
        ports:
        - containerPort:  3000

---

apiVersion: v1
kind: Service
metadata:
  name: lab2deployment
spec:
  type: NodePort
  selector:
    app: lab2
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
```

```bash
kubectl apply -f lab2.yaml
```
![Apply yaml](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/666840f4-ee07-4c01-b000-f2981afa8319)

- Количество подов:
![Pods](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/1e28cbe6-7d62-4b71-b2d6-7c5c1c79d997)


- Запустим в `minikube` режим проброса портов и подключился к контейнерам через веб-браузер.

```bash
kubectl port-forward service/lab2deployment 3000:3000
```
- Перейдем на данный адрес в веб-браузер и увидим:
![React](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/1efd5eda-0376-4a6a-968d-7f02353e7979)


- В веб браузере переменные `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` и `Container name` не изменяются.
- Проверим, что переменные передались внутрь контейнера:
![ENV](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/83235bf7-476c-47dc-bac5-461844e0cacf)


- Ввиду идентичности контенеров(помним что они реплики) логи контейнеров так же идентичны.
![logs](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/62cc4621-e254-40d1-a93b-49d31c58a276)


- Схема организации контейеров и сервисов данной лабораторной
![scheme](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/59fb932c-fec0-4f0b-973b-dbb032e3ee8e)

