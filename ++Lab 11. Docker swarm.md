Для выполнения заданий в Play with Docker (PWD) с использованием Docker Swarm для управления кластером, давайте поэтапно пройдем через все шаги.

### Шаг 1: Установка кластера Docker Swarm

- **Создание кластера с одной управляющей нодой и двумя рабочими**:

- Перейдите на сайт Play with Docker.

- Нажмите на кнопку **"Add New Session"**.

- На левой панели выберите + ADD NEW INSTANCE.

- Создайте 3 ноды, у вас должна быть одна управляющая нода и две рабочие ноды. Вы можете реализовать это с помощью следующих команд:

   ```
docker swarm init  # на управляющей ноде
```

   После выполнения этой команды, вы получите команду, которую можно использовать для добавления рабочих нод, например:
   
   ```
docker swarm join --token <TOKEN> <MANAGER-IP>:<PORT>
```
- Подключите обе рабочие ноды, используя полученную команду.

- **Проверка состояния кластера**:

   ```
docker node ls
```

### Шаг 2: Удаление и повторное добавление рабочей ноды

- **Удалите одну из рабочих нод** (на управляющей ноде):

   ```
docker node update --availability drain <WORKER-NODE-ID>
   docker node rm <WORKER-NODE-ID>
```
- **Добавьте рабочую ноду обратно**:

   Снова используйте команду, полученную при инициализации, чтобы присоединить рабочую ноду.

### Шаг 3: Создание Docker образа, показывающего метаданные узла

- **Создайте Dockerfile**. Например, используем Python для создания простого приложения, которое будет выводить метаданные узла, включая IP адрес:

    ```
# Dockerfile
    FROM python:3.9-slim

    WORKDIR /app

    COPY app.py .

    EXPOSE 5000

    RUN pip install flask

    CMD ["python", "app.py"]
```
- **Создайте файл app.py**:

    ```
# app.py
    from flask import Flask, request
    import socket
    import os

    app = Flask(__name__)

    @app.route('/')
    def metadata():
        hostname = socket.gethostname()
        ip_address = request.host
        return f'Metadata of the Node:\nHostname: {hostname}\nIP Address: {ip_address}'

    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=5000)
```
- **Постройте образ и загрузите его в Docker Hub** (в Play with Docker вы можете использовать docker build):

    ```
docker build -t mynodeinfo:latest .
```

### Шаг 4: Развертывание сервиса с 3 репликами

```
docker service create --name nodeinfo-service --replicas 3 -p 5000:5000 mynodeinfo:latest
```

Проверьте, на какие IP-адреса вы попадаете:

```
docker service ps nodeinfo-service
```

### Шаг 5: Обновление образа с информацией о разработчике

- **Обновите app.py**, добавив информацию о разработчике (например, имя и контакт):

    ```
# app.py
    # ...
    @app.route('/')
    def metadata():
        hostname = socket.gethostname()
        ip_address = request.host
        developer_info = "Developed by: Your Name, Contact: you@example.com"
        return f'Metadata of the Node:\nHostname: {hostname}\nIP Address: {ip_address}\n{developer_info}'
```
- **Пересоберите образ**:

    ```
docker build -t mynodeinfo:latest .
```
- **Обновите сервис**:

    ```
docker service update --image mynodeinfo:latest nodeinfo-service
```

### Заключение

Теперь у вас есть развернутый Docker Swarm с активными узлами, сервис с несколькими репликами и произведенные обновления приложения, включая информацию о разработчике. Вы можете обращаться к конечной точке по IP-адресу управляющей ноды на порту 5000, чтобы увидеть результаты.
