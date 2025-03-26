# Домашнее задание к занятию «Хранение в K8s. Часть 1» - Скрыпников Илья

## Задание 1: Обмен файлами между контейнерами в Pod

### Манифест Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shared-volume-deployment
  labels:
    app: shared-volume
spec:
  replicas: 1
  selector:
    matchLabels:
      app: shared-volume
  template:
    metadata:
      labels:
        app: shared-volume
    spec:
      containers:
      - name: busybox
        image: busybox:latest
        command: ["/bin/sh", "-c"]
        args: ["while true; do echo $(date) >> /shared-data/log.txt; sleep 5; done"]
        volumeMounts:
        - name: shared-volume
          mountPath: /shared-data
      - name: multitool
        image: wbitt/network-multitool:latest
        command: ["/bin/sh", "-c"]
        args: ["tail -f /shared-data/log.txt"]
        volumeMounts:
        - name: shared-volume
          mountPath: /shared-data
      volumes:
      - name: shared-volume
        emptyDir: {}
```

### Демонстрация работы
1. Применяем манифест:
```bash
kubectl apply -f deployment.yaml
```

2. Проверяем логи контейнера multitool:
```bash
kubectl logs -f shared-volume-deployment-55b4b6df5-dxhfb -c multitool
```

Пример вывода:

![image](https://github.com/user-attachments/assets/468f841e-fd67-4d7b-90c1-fc37cf31b399)


## Задание 2: Чтение логов ноды через DaemonSet

### Манифест DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-log-reader
  labels:
    app: node-log-reader
spec:
  selector:
    matchLabels:
      app: node-log-reader
  template:
    metadata:
      labels:
        app: node-log-reader
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool:latest
        command: ["/bin/sh", "-c"]
        args: ["tail -f /host/var/log/syslog"]
        volumeMounts:
        - name: host-log
          mountPath: /host/var/log
          readOnly: true
      volumes:
      - name: host-log
        hostPath:
          path: /var/log
          type: File
```

### Демонстрация работы
1. Применяем манифест:
```bash
kubectl apply -f node-log-reader-daemonset.yaml
```

2. Проверяем логи пода:
```bash
kubectl logs -f node-log-reader-<pod-id>
```

Пример вывода (фрагмент логов ноды):
```
Mar 22 10:20:01 microk8s-node01 systemd[1]: Starting Regular background program processing daemon...
Mar 22 10:20:01 microk8s-node01 cron[1234]: (CRON) INFO (pidfile fd = 3)
Mar 22 10:20:01 microk8s-node01 cron[1234]: (CRON) INFO (Running @reboot jobs)
...
```
