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
  name: test-daemonset
  namespace: volume1
  labels:
    app: multitool
spec:
  selector:
    matchLabels:
      name: test-daemonset
  template:
    metadata:
      labels:
        name: test-daemonset
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        command: ["/bin/sh", "-c", "tail -f /nodes-logs/syslog"]  # Автоматический просмотр логов
        volumeMounts:
        - name: host-logs
          mountPath: /nodes-logs
          readOnly: true
      volumes:
      - name: host-logs
        hostPath:
          path: /var/log
          type: Directory
```

### Демонстрация работы
1. Применяем манифест:
```bash
kubectl apply -f daemonset.yaml
```

2. Проверяем логи пода:
![image](https://github.com/user-attachments/assets/d20c366d-6f44-4c03-8979-e1657668f247)

