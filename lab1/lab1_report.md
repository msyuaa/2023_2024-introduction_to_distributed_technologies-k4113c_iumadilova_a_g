University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4113c  
Author: Iumadilova Angela\
Lab: Lab1  
Date of create: 04.11.2023  
Date of finished: 08.11.2023


## Установка Docker и Minikube, мой первый манифест
### Цель работы
Ознакомиться с инструментами Minikube и Docker, развернуть под.
### Ход работы
#### 1. Подготовка
Для начала выполнения работы был установлен Docker и Minikube.
Далее развернут minikube cluster командой: 
```
minikube start
```
![image](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/16b229d3-f1a8-4087-b911-97bb07d7df89)


#### 3. Создание манифеста
Для развертывания "пода" создан файл `lab1.yaml`, описывающий конфигурацию.
```
apiVersion: v1
kind: Pod
metadata:
  name: vault
  labels:
    app: vault
spec:
  containers:
  - name: vault
    image: vault:1.13.3
    ports:
    - containerPort:  8200
```
Описание полей:
   * `metadata` - содержит метаданные ресурса, такие как имя и метки (labels), которые помогают в организации и идентификации ресурса.
   * `spec` - содержит конфигурацию ресурса, включая параметры, которые определяют его поведение и характеристики.
     *  `containers` - содержит описывание каждого контейнера в данном поде.
  
Контейнер созданн на основе [образа](https://hub.docker.com/_/vault/) image версии 1.13.3.

#### 4. Создание "пода" в кластере
Используем команду:
```
kubectl apply -f lab1.yaml
```
#### 5. Cоздание сервиса для доступа к контейнеру
Для доступа к контейнеру создаем сервис с помощью команды:
```
minikube kubectl -- expose pod vault --type=NodePort --port=8200
```
 Cервис обеспечивает доступ к поду снаружи кластера при подключении к порту 8200. Флаг `--type=NodePort` указывает, что используется тип сервиса `NodePort` с выбранным портом `--port=8200`. Это значит, что при подключении к ноду по порту `8200` нод будет перенаправлять трафик на ClusterIP созданного сервиса (который, в свою очередь, перенаправляет трафик на под).
ClusterIP - IP адрес, присвоенный сервису (доступен только внутри кластера).
 
#### 6. Прокидываение порта
Для того, чтобы зайти в контейнер, прокинут порт локального компьютера в контейнер с помощью команды:
```
minikube kubectl -- port-forward service/vault 8200:8200
```
После этого переходим по ссылке [http://localhost:8200](http://localhost:8200), где открывается интерфейс.

![vault](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/7093465b-d14a-4ec0-8842-267e222c302a)


#### 7. Получение токена для входа в Vault
Чтобы найти токен, открываем логи с помощью комнады:
```
kubectl logs vault
```
kubectl logs используется для получения логов из контейнера в указанном поде. 
Результат выполнения команды:

![Token](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/ab8a6a99-3a66-4667-848c-562ab1c15300)


Токен для входа располагается в поле `Root Token`. С помощью него можно авторизоваться в Vault, что показано на скрине.

![Vault_page](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/72d71fb4-97f8-488e-9df1-2abe5efe17a0)


#### Ответы на вопросы
1. Что сейчас произошло и что сделали команды, указанные ранее?\
Развернуто приложение Vault в кластере Kubernetes. Создан манифест, согласно которому был создан под с контейнером. В контейнере запущено приложение Vault. Для работы контейнеру был указан порт 8200, а впоследствии был создан дополнительный уровень абстрации - сервис для доступа к контейнеру. Сервис открывает доступ на порт контейнера. Последней командой был настроен проброс порта 8200 на localhost в порт 8200 на контейнере.
2. Где взять токен для входа в Vault?
В логах "пода", используя команду:
```
kubectl logs vault
```

#### Схема организации контейнеров и сервисов

![scheme](https://github.com/msyuaa/2023_2024-introduction_to_distributed_technologies-k4113c_iumadilova_a_g/assets/97636484/dab7abb3-26a7-4fdd-9adb-7143f7ba99ff)
