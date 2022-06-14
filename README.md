# Домашнее задание к занятию "14.3 Карты конфигураций"

<details>

  <summary>Описание задачи</summary> 
## Задача 1: Работа с картами конфигураций через утилиту kubectl в установленном minikube

Выполните приведённые команды в консоли. Получите вывод команд. Сохраните
задачу 1 как справочный материал.

### Как создать карту конфигураций?

```
kubectl create configmap nginx-config --from-file=nginx.conf
kubectl create configmap domain --from-literal=name=netology.ru
```

### Как просмотреть список карт конфигураций?

```
kubectl get configmaps
kubectl get configmap
```

### Как просмотреть карту конфигурации?

```
kubectl get configmap nginx-config
kubectl describe configmap domain
```

### Как получить информацию в формате YAML и/или JSON?

```
kubectl get configmap nginx-config -o yaml
kubectl get configmap domain -o json
```

### Как выгрузить карту конфигурации и сохранить его в файл?

```
kubectl get configmaps -o json > configmaps.json
kubectl get configmap nginx-config -o yaml > nginx-config.yml
```

### Как удалить карту конфигурации?

```
kubectl delete configmap nginx-config
```

### Как загрузить карту конфигурации из файла?

```
kubectl apply -f nginx-config.yml
```
</details>



### Решение
  
1. Как создать карту конфигураций?
```
$ kubectl create configmap nginx-config --from-file=nginx.conf -n clokub-14-03 
configmap/nginx-config created

$ kubectl create configmap domain --from-literal=name=netology.ru -n clokub-14-03 
configmap/domain created
```

2. Как просмотреть список карт конфигураций?

```shell
$ kubectl get configmaps -n clokub-14-03
NAME               DATA   AGE
domain             1      57s
kube-root-ca.crt   1      22m
nginx-config       1      106s

$ kubectl get configmap -n clokub-14-03
NAME               DATA   AGE
domain             1      82s
kube-root-ca.crt   1      22m
nginx-config       1      2m11s
```

3. Как просмотреть карту конфигурации?

```
$ kubectl get configmap nginx-config -n clokub-14-03
NAME           DATA   AGE
nginx-config   1      2m52s

$ kubectl describe configmap domain -n clokub-14-03
Name:         domain
Namespace:    clokub-14-03
Labels:       <none>
Annotations:  <none>

Data
====
name:
----
netology.ru

BinaryData
====

Events:  <none>
```

4. Как получить информацию в формате YAML и/или JSON?

```
$ kubectl get configmap nginx-config -o yaml -n clokub-14-03
apiVersion: v1
data:
  nginx.conf: |
    server {
        listen 80;
        server_name  netology.ru www.netology.ru;
        access_log  /var/log/nginx/domains/netology.ru-access.log  main;
        error_log   /var/log/nginx/domains/netology.ru-error.log info;
        location / {
            include proxy_params;
            proxy_pass http://10.10.10.10:8080/;
        }
    }
kind: ConfigMap
metadata:
  creationTimestamp: "2022-06-14T10:21:32Z"
  name: nginx-config
  namespace: clokub-14-03
  resourceVersion: "617836"
  uid: e211abfe-c2ff-4266-a1c9-d71c778ce7ce

$ kubectl get configmap domain -o json -n clokub-14-03
{
    "apiVersion": "v1",
    "data": {
        "name": "netology.ru"
    },
    "kind": "ConfigMap",
    "metadata": {
        "creationTimestamp": "2022-06-14T10:22:21Z",
        "name": "domain",
        "namespace": "clokub-14-03",
        "resourceVersion": "618098",
        "uid": "1606afec-e565-4c63-8b2a-6a5652659cfb"
    }
}
```

5. Как выгрузить карту конфигурации и сохранить его в файл?

```
$ kubectl get configmaps -o json > configmaps.json -n clokub-14-03
$ kubectl get configmap nginx-config -o yaml > nginx-config.yml -n clokub-14-03
```
Создались соотвествующие файлы.

6. Как удалить карту конфигурации?

```
$ kubectl delete configmap nginx-config -n clokub-14-03
configmap "nginx-config" deleted
```

7. Как загрузить карту конфигурации из файла?

```
$ kubectl apply -f nginx-config.yml -n clokub-14-03
configmap/nginx-config created
```

---


## Задача 2 (*): Работа с картами конфигураций внутри модуля

<details>

  <summary>Описание задачи</summary> 
Выбрать любимый образ контейнера, подключить карты конфигураций и проверить
их доступность как в виде переменных окружения, так и в виде примонтированного
тома
</details>



### Решение

1. Создаем манифест для ConfigMap с двумя ключами cm-demo-file и cm-demo-env.
```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config-map
data:
  cm-demo-file: |
    Hello! I'm demo file config!
  cm-demo-env: |
    Hello! I'm demo env!
```

2. Создаем манифест для deployment. Монтируем ключи в env (значение cm-demo-env) и volume (весь ConfigMap demo-config-map).
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: config-map-demo
  labels:
    role: config-map-demo
spec:
  replicas: 1
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      role: config-map-demo
  template:
    metadata:
      labels:
        role: config-map-demo
    spec:
      containers:
        - image: rdegtyarev/fedora-pip:0.2.0
          imagePullPolicy: Always
          name: fedora-app
          resources:
            limits:
              cpu: 200m
              memory: 256Mi
            requests:
              cpu: 100m
              memory: 128Mi
          env:
            - name: CM_ENV_DEMO
              valueFrom:
                configMapKeyRef:
                  name: demo-config-map
                  key: cm-demo-env
          command: ["/bin/sleep", "365d"]
          volumeMounts:
            - mountPath: /config/
              name: demo-config
      volumes:
        - name: demo-config
          configMap:
            name: demo-config-map
```

Применяем манифесты, загружаемся в запущенный контейнер fedora-app и проверяем результат: 
- проверяем что в директории /config/ присутствуют файлы конфигурации.
- проверяем содержимое переменной CM_ENV_DEMO

```
$ exec kubectl exec -i -t -n clokub-14-03 config-map-demo-767db659f4-s9lm2 -c fedora-app -- sh
[root@config-map-demo-767db659f4-s9lm2 config]# ls /config/
cm-demo-env  cm-demo-file

[root@config-map-demo-767db659f4-s9lm2 config]# cat /config/cm-demo-file 
Hello! I'm demo file config!

[root@config-map-demo-767db659f4-s9lm2 config]# echo $CM_ENV_DEMO 
Hello! I'm demo env!

```

---
