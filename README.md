# Домашнее задание к занятию «Helm»

## Описание домашнего задания

### Цель работы
Научиться работать с пакетным менеджером Helm для Kubernetes, а именно:
- Создавать и настраивать чарты
- Управлять релизами
- Работать с разными окружениями

### Условия выполнения
- Установленное k8s-решение (MicroK8S)
- Установленный локальный kubectl
- Установленный локальный Helm
- Редактор YAML-файлов

## Задание 1. Подготовить Helm-чарт для приложения

1) Необходимо упаковать приложение в чарт для деплоя в разные окружения.
2) Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
3) В переменных чарта измените образ приложения для изменения версии.

## Задание 2. Запустить две версии в разных неймспейсах

1) Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
2) Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
3) Продемонстрируйте результат.

## Выполнение работы

### 1. Подготовка рабочего окружения

#### 1.1. Установка и настройка MicroK8S
```bash
# Установка MicroK8S
sudo apt update
sudo apt install snapd
sudo snap install microk8s --classic

# Настройка доступов
sudo usermod -a -G microk8s $USER
sudo chown -R $USER ~/.kube
newgrp microk8s

# Проверка статуса
microk8s status
```

#### 1.2. Настройка kubectl
```bash
mkdir -p $HOME/.kube
sudo microk8s config > $HOME/.kube/config
kubectl get nodes
```

#### 1.3. Установка Helm
```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

![image](https://github.com/temagraf/10-HelmK8s/blob/main/1-1%20ecnfyjdrf%20HELM%20.png)


### 2. Создание Helm-чарта

#### 2.1. Создание базовой структуры чарта
```bash
helm create app-chart
```

#### 2.2. Структура созданного чарта
```
app-chart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   └── service.yaml
└── values.yaml
```

![image](https://github.com/temagraf/10-HelmK8s/blob/main/2-2%20структура%20чарта%20.png)

#### 2.3. Конфигурация Chart.yaml
```yaml
apiVersion: v2
name: app-chart
description: A Helm chart for test application
type: application
version: 0.1.0
appVersion: "1.16.0"
```

#### 2.4. Конфигурация values.yaml
```yaml
replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.16.0"

nameOverride: ""
fullnameOverride: ""

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

### 3. Развертывание приложений

#### 3.1. Создание namespace
```bash
kubectl create namespace app1
kubectl create namespace app2
```

![image](https://github.com/temagraf/10-HelmK8s/blob/main/3-1%20Создание%20namespace.png)

#### 3.2. Установка релизов
```bash
helm install app1-v1 app-chart --namespace app1 --set image.tag=1.16.0
helm install app1-v2 app-chart --namespace app1 --set image.tag=1.17.0
helm install app2-v1 app-chart --namespace app2 --set image.tag=1.18.0
```

![image](https://github.com/temagraf/10-HelmK8s/blob/main/3-2%20установка%20релизов.png)



### 4. Проверка работоспособности

#### 4.1. Проверка релизов Helm
```bash
$ helm list --all-namespaces
NAME                           NAMESPACE  REVISION  UPDATED                                  STATUS    CHART                                   APP VERSION
app1-v1                        app1       1        2025-02-01 01:31:24.242019876 +0300 MSK deployed  app-chart-0.1.0                       1.16.0     
app1-v2                        app1       1        2025-02-01 01:31:31.305521843 +0300 MSK deployed  app-chart-0.1.0                       1.16.0     
app2-v1                        app2       1        2025-02-01 01:31:39.990735761 +0300 MSK deployed  app-chart-0.1.0                       1.16.0     
```

![image](https://github.com/temagraf/10-HelmK8s/blob/main/4-1%20проверка%20релизов%20Helm%20.png)


#### 4.2. Проверка подов
```bash
$ kubectl get pods -n app1
NAME                                  READY   STATUS    RESTARTS   AGE
app1-v1-app-chart-69f55ddbb5-852mf   1/1     Running   0          109s
app1-v2-app-chart-9cfdd9955-jcd6n    1/1     Running   0          102s

$ kubectl get pods -n app2
NAME                                  READY   STATUS    RESTARTS   AGE
app2-v1-app-chart-84b55d68db-5lrhw   1/1     Running   0          102s
```

![image](https://github.com/temagraf/10-HelmK8s/blob/main/4-2%20проветка%20подов.png)


#### 4.3. Проверка сервисов
```bash
$ kubectl get services -n app1
NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
app1-v1-app-chart   ClusterIP   10.152.183.100   <none>        80/TCP    3m26s
app1-v2-app-chart   ClusterIP   10.152.183.58    <none>        80/TCP    3m19s

$ kubectl get services -n app2
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
app2-v1-app-chart   ClusterIP   10.152.183.55   <none>        80/TCP    3m18s
```

![image](https://github.com/temagraf/10-HelmK8s/blob/main/4-3%20проверка%20снрвисов.png)

## В результате мы видим, что у нас: 

 Установлен и настроен Helm  
 Создан работающий Helm-чарт  
 Развернуто три версии приложения:
  - app1-v1 (nginx:1.16.0)
  - app1-v2 (nginx:1.17.0)
  - app2-v1 (nginx:1.18.0)  
 Все поды успешно запущены и работают  
 Созданы и настроены соответствующие сервисы
