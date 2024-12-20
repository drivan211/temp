Давайте выполним вашу задачу по созданию Docker стека с использованием образа containous/whoami в Play with Docker (PWD). Мы также рассмотрим, как управлять репликами сервисов, ресурсами нод и секретами.

### Шаг 1: Создание Docker Стека

- **Создайте Docker Compose файл.**

   Создайте файл docker-compose.yml со следующим содержимым:

   ```
version: '3.8'

   services:
     whoami:
       image: containous/whoami
       deploy:
         replicas: 3
         placement:
           constraints:
             - node.role == manager
       ports:
         - "80"

     whoami2:
       image: containous/whoami
       deploy:
         replicas: 3
         placement:
           constraints:
             - node.role == manager
       ports:
         - "81"
```

### Шаг 2: Развертывание стека

- **Разверните стек** с помощью команды:

   ```
docker stack deploy -c docker-compose.yml my_stack
```

### Шаг 3: Определение количества оперативной памяти

- **Проверьте количество оперативной памяти на мастер ноде и воркер нодах**:

   Для этого выполните команды:

   ```
docker node ls  # Выведет список нод и их роли
   docker info  # Выводит информацию о состоянии системы, включая доступную память
```

   Вы можете также узнать подробности о каждой ноде:

   ```
docker node inspect <NODE-ID>
```

### Шаг 4: Размещение нового сервиса

- **Создайте новый сервис с использованием параметра placement для размещения на ноде с меньшим потреблением ресурсов**.

   Если у вас есть одна воркер нода, и вы хотите разместить второй сервис только на ней, добавьте следующий сервис в ваш docker-compose.yml:

   ```
services:
     whoami2:
       image: containous/whoami
       deploy:
         replicas: 1
         placement:
           constraints:
             - node.opa == worker  # Убедитесь, что вы заменили "worker" на нужное имя вашей воркер ноды
```

### Шаг 5: Установите зарезервированные и максимальные ресурсы

- **Установите зарезервированные и максимальные ресурсы для сервиса**. В docker-compose.yml добавьте настройки для ресурсов:

   ```
services:
     whoami:
       image: containous/whoami
       deploy:
         replicas: 3
         resources:
           limits:
             cpus: '0.5'
             memory: 512M
           reservations:
             cpus: '0.25'
             memory: 256M
```
- **Обновите стек с этими изменениями**:

   ```
docker stack deploy -c docker-compose.yml my_stack
```

### Шаг 6: Инспекция сервиса

- **Выведите результаты проинспектировав сервис** после развертывания:

   ```
docker service ps my_stack_whoami
```

### Шаг 7: Создание и использование секрета

- **Создайте секрет и добавьте его к контейнеру с базой данных**:

   Например, создайте секрет с помощью следующей команды:

   ```
echo "my_secret_password" | docker secret create db_password -
```

   Затем добавьте этот секрет к сервису в docker-compose.yml:

   ```
services:
     your_db_service:
       image: your_database_image
       secrets:
         - db_password

   secrets:
     db_password:
       external: true
```
- **Обновите стек с секретом**:

   ```
docker stack deploy -c docker-compose.yml my_stack
```

### Заключение

Теперь у вас есть стек, состоящий из нескольких сервисов с заданным количеством реплик, размещенных на мастер ноде. Вы также установили ограничения на использование ресурсов и добавили секрет для вашего контейнера с базой данных. Не забудьте проверить, что все сервисы работают должным образом, запустив команды инспекции.
