Давайте поэтапно рассмотрим, как выполнить указанные вами задачи, начиная с резервного копирования статического контента, установки сервера времени, создания Dockerfile для серверов на Ubuntu/Alpine/CentOS, а также резервного и восстановления баз данных PostgreSQL или MongoDB с использованием томов Docker.

### Шаг 1: Резервное копирование volume статического контента сайта с использованием Docker Compose

Создайте docker-compose.yml для статического контента сайта и выполнения резервного копирования:

```
version: '3.8'

services:
  web:
    image: nginx
    volumes:
      - nginx:/usr/share/nginx/html

  backup:
    image: alpine
    volumes:
      - nginx:/backup
      - backup_nginx:/data
    entrypoint: ['sh', '-c', 'tar -czvf /backup/backup.tar.gz /backup']

volumes:
  nginx:
  backup_nginx:
```

Запустите команды для создания резервной копии:

```
docker-compose up backup
```

Эта команда создаст backup.tar.gz в Docker volume nginx.

### Шаг 2: Установка NTP сервера в контейнерах на базе Ubuntu, CentOS и Alpine

Создайте Dockerfile для сервера времени на базе Ubuntu, CentOS и Alpine:

```
# Ubuntu NTP server
FROM ubuntu:latest
RUN apt-get update && apt-get install -y ntp
COPY ntp.conf /etc/ntp.conf
CMD ["ntpd", "-g", "-u", "ntp:ntp"]
```

Для CentOS:
```
# CentOS NTP server
FROM centos:latest
RUN yum install -y ntp
COPY ntp.conf /etc/ntp.conf
CMD ["ntpd", "-g", "-u", "ntp:ntp"]
```

Для Alpine:
```
# Alpine NTP server
FROM alpine:latest
RUN apk add --no-cache openntpd
COPY ntpd.conf /etc/ntpd.conf
CMD ["ntpd", "-f", "/etc/ntpd.conf"]
```

### Шаг 3: Сборка образов

Соберите образы для каждого контейнера:

```
docker build -t ntp-ubuntu -f Dockerfile.ubuntu .
docker build -t ntp-centos -f Dockerfile.centos .
docker build -t ntp-alpine -f Dockerfile.alpine .
```

### Шаг 4: Резервное копирование и восстановление базы данных PostgreSQL или MongoDB

Для резервного копирования MongoDB:
- Запустите контейнер MongoDB:

```
docker run --name mongo-container -d mongo
```
- Внутри контейнера выполните команду резервного копирования:

```
docker exec mongo-container sh -c 'exec mongodump --archive=/data/db_backup.archive'
```
- Чтобы восстановить базу данных из резервной копии, выполните команду:

```
docker run --rm --link mongo-container:mongo -v <path_to_backup>:/data mongo sh -c 'exec mongorestore --archive=/data/db_backup.archive'
```

### Шаг 5: Используйте Docker volume для резервного копирования

Создайте резервную копию с помощью Docker volume:

```
docker run -v mongo-volume:/backup -v backup_mongo:/data alpine tar -czvf /backup/db_backup.tar.gz /backup
```

Для восстановления используйте:

```
docker run --rm -v backup_mongo:/backup -v new_volume:/data alpine tar -xzvf /backup/db_backup.tar.gz -C /data
```

### Шаг 6: Дополнительные инструменты для резервного копирования

- **Duplicity** - это инструмент для резервного копирования, который использует шифрование и инкрементальное резервное копирование. Его можно использовать в контейнерах или на хосте.

- **Velero** - это инструмент для резервного копирования и восстановления Kubernetes ресурсов и объектов хранилища. Устанавливается в кластер Kubernetes и используем для управления резервными копиями.

- **Restic** - это быстрый, эффективный и безопасный инструмент для резервного копирования, который поддерживает различные хранилища, включая S3, Azure, Google Cloud и другие.

### Заключение

Каждый из шагов может быть настроен под ваши конкретные нужды и окружение. Убедитесь, что ваши конфигурации и команда работают под вашим окружением и адаптированы для конкретных требований вашего приложения или инфраструктуры.
