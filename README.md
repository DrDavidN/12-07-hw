# «Хранение в K8s. Часть 2» - Дрибноход Давид

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.

#### Ответ

Создаю новый namespace ```kubectl create namespace 12-07-hw```

Создаю deployment.yaml применяю его и проверяю статус

![image](https://github.com/DrDavidN/12-07-hw/assets/128225763/167f1596-0788-402a-90f1-1b8a932bcd61)

Статус пода в состоянии Pending. Под не запустился по причине отсутствия PVC с именем ```pvc-vol```.

2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.

#### Ответ

Создаю pv-vol.yaml и pvc-vol.yaml применяю их и проверяю их статус, повторно проверяю статус pod

![image](https://github.com/DrDavidN/12-07-hw/assets/128225763/eef3442f-8387-484a-8fa7-8442358d2995)

3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 

#### Ответ

```BASH
kubectl exec -n 12-07-hw pvc-deployment-85cff6c56b-drnkg -c network-multitool -it -- sh
```
![image](https://github.com/DrDavidN/12-07-hw/assets/128225763/1170ff54-3538-46dc-9d09-57973acfbd7f)


4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.

#### Ответ

![image](https://github.com/DrDavidN/12-07-hw/assets/128225763/61566e5a-c40a-4be4-9df2-ac513a3ca17a)
![image](https://github.com/DrDavidN/12-07-hw/assets/128225763/fe0f900e-919a-43a9-baa7-38fa844163e0)
![image](https://github.com/DrDavidN/12-07-hw/assets/128225763/4e839ea6-3908-4b97-8337-510fdc2abd2d)

Файл остался. При конфигурировании pv я использовал режим ReclaimPolicy: Retain, при котором после удаления PV ресурсы из внешних провайдеров автоматически не удаляются". Даже после удаления pv файлы также останутся.

6. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

#### Ответ

deployment.yaml

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pvc-deployment
  labels:
    app: main2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main2
  template:
    metadata:
      labels:
        app: main2
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ['sh', '-c', 'while true; do echo Success! >> /tmp/cache/success.txt; sleep 5; done']
        volumeMounts:
        - name: my-vol-pvc
          mountPath: /tmp/cache

      - name: network-multitool
        image: wbitt/network-multitool
        volumeMounts:
        - name: my-vol-pvc
          mountPath: /static
        env:
        - name: HTTP_PORT
          value: "80"
        - name: HTTPS_PORT
          value: "443"
        ports:
        - containerPort: 80
          name: http-port
        - containerPort: 443
          name: https-port
        resources:
          requests:
            cpu: "1m"
            memory: "20Mi"
          limits:
            cpu: "10m"
            memory: "20Mi"
      volumes:
      - name: my-vol-pvc
        persistentVolumeClaim:
          claimName: pvc-vol
```

pv-vol.yaml

```YAML
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  storageClassName: ""
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/pv
  persistentVolumeReclaimPolicy: Retain
```

pvc-vol.yaml

```YAML
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-vol
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------
