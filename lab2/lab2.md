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

- Запустим в `minikube` режим проброса портов и подключился к контейнерам через веб-браузер.

```bash
kubectl port-forward service/lab2deployment 3000:3000
```



- В веб браузере переменные `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` и `Container name` не изменяются.

- Ввиду идентичности контенеров(помним что они реплики) логи контейнеров так же идентичны.


- Схема организации контейеров и сервисов данной лабораторной
