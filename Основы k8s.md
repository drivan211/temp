Давайте шаг за шагом создадим манифесты для каждого из типов сервисов в Kubernetes: **ClusterIP**, **NodePort**, **LoadBalancer**, и **ExternalName**. Я предоставлю примеры YAML-файлов для каждого случая, а затем опишу, как их применить и проверить.

### 1. Создание ClusterIP сервиса

### Манифест для пода приложения

```
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  labels:
    app: my-app
spec:
  containers:
    - name: my-app-container
      image: nginx  # Здесь вы можете указать любое подходящее вам приложение
      ports:
        - containerPort: 8080
```

### Манифест для ClusterIP сервиса

```
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

### Применение манифестов

```
kubectl apply -f my-app-pod.yaml
kubectl apply -f my-app-service.yaml
```

### Проверка работы сервиса

Чтобы проверить работоспособность сервиса из другого пода, создайте временный под и выполните тестовую команду:

```
kubectl run -it --rm --restart=Never temp-pod --image=busybox -- sh
```

Теперь в командной строке временного пода выполните:

```
wget -qO- http://my-app-service
```

### 2. Создание NodePort сервиса

### Манифест для пода приложения (аналогичный)

```
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  labels:
    app: my-app
spec:
  containers:
    - name: my-app-container
      image: nginx  # Это должно быть то же приложение
      ports:
        - containerPort: 8080
```

### Манифест для NodePort сервиса

```
apiVersion: v1
kind: Service
metadata:
  name: my-app-nodeport-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30001  # Вы можете указать любой доступный порт, или оставить пустым для автоматического выбора
  type: NodePort
```

### Применение манифестов

```
kubectl apply -f my-app-pod.yaml
kubectl apply -f my-app-nodeport-service.yaml
```

### Проверка работы сервиса

Теперь вы можете получить доступ к вашему приложению через любой узел кластера по IP-адресу узла и указанному NodePort:

```
http://<NODE_IP>:30001
```

### 3. Создание LoadBalancer сервиса

### Манифест для пода приложения (аналогичный)

```
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  labels:
    app: my-app
spec:
  containers:
    - name: my-app-container
      image: nginx  # Опять же, это должно быть то же приложение
      ports:
        - containerPort: 8080
```

### Манифест для LoadBalancer сервиса

```
apiVersion: v1
kind: Service
metadata:
  name: my-app-loadbalancer-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

### Применение манифестов

```
kubectl apply -f my-app-pod.yaml
kubectl apply -f my-app-loadbalancer-service.yaml
```

### Проверка работы сервиса

Через несколько минут, после применения манифеста, вы должны увидеть внешний IP-адрес вашего LoadBalancer:

```
kubectl get services
```

Получите доступ к вашему приложению по этому IP-адресу.

### 4. Создание ExternalName сервиса

### Манифест для ExternalName сервиса

```
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: ExternalName
  externalName: my.external-service.com  # Замените на нужное вам DNS-имя
```

### Применение манифеста

```
kubectl apply -f external-service.yaml
```

### Проверка работы сервиса

Чтобы убедиться, что сервис работает, вы можете выполнить следующее:

```
kubectl run -it --rm --restart=Never temp-pod --image=busybox -- sh
```

Внутри временного пода выполните:

```
nslookup external-service
```

Или:

```
wget -qO- http://external-service
```

### Заключение

Теперь у вас есть манифесты и инструкции по применению различных типов сервисов в Kubernetes: **ClusterIP**, **NodePort**, **LoadBalancer** и **ExternalName**. Не забывайте изменять параметры, такие как имя образа, порты и внешний DNS, чтобы использовать их в соответствующих случаях.
