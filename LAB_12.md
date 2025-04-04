# Лабораторная работа №12
### Тема : Docker compose
### Цель : Масштабируемость и отказоустойчивость + docker compose
---
### Порядок работы :

### Установка docker-compose

```sh
# Если не установлена утилита curl
sudo apt install curl
```

```sh
sudo curl -SL https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```

```sh
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

```sh
chmod a+x /usr/local/bin/docker-compose
```

### Подготовка проекта

скачать проект <a href="https://github.com/iu5git/DevOps"> DevOps </a> на локальную машину: Code ->Download Zip  
<img src="LAB\src\img\lb12\download.png" width=800px>  
Разархивировать проект и зайти в папку с проектом:

```sh
cd DevOps-master/Лабы/Лаб3
```

Структура директории

<img src="LAB\src\img\lb12\path.png" width=250px>  

### Задачи лабораторной работы

### 1. Изучить работу docker-compose

Изучить структуру проекта: test-compose

```sh
cd test-compose/
ls
cat docker-compose.yml
```

docker-compose.yml - файл конфиогурайии запуска любого проекта docker-compose

```sh
version: "3.2"

services:
  whale: #название сервисов. Сервис=контейнер docker
    image: blaines/whalesay #образ, на основании которогостроится контейнер
    command: ["cowsay", "hello!"]  #команда, которая передается в контейнер
```

Стартовать контейнер

```docker
docker compose up
```

<img src="LAB\src\img\lb12\test-compose.png" width=800px>

> ### Задача: Заменить надпись на свое имя, запустить контейнер
> ### Для замены надписи необходимо открыть файл docker-compose.yml в папке test-compose в редакторе nano
> ### Зафиксировать результат для отчета

### 2. Изучить методы балансировки нагрузки между узлами кластера

Изучить структуру проектов: Haproxy balancer

```sh
cd ../haproxy-static-balancer
```

### docker-compose.yml

```yml
version: '3'

services:
  worker1:
    build: # собираем образ воркера наосновании контейнера с nginx.Контейнер отдает нам статическую страницу с значением WORKER_ID 
      context: ../worker
      dockerfile: Dockerfile
      args:
        - WORKER_ID=1  # Аргумент, который мы передаем в контейнер
    expose:
        - 80 # Порт, по которому запущен nginx внутри контейнера
  worker2:
    build:
      context: ../worker
      dockerfile: Dockerfile
      args:
        - WORKER_ID=2
    expose:
        - 80 # Порт, по которому запущен nginx внутри контейнера
  haproxy:
    image: haproxy
    volumes:
        - ./haproxy:/usr/local/etc/haproxy # путь к файлу конфигурации haproxy
    links: # Связываем воркеры и балансировщик. ВАЖНО- имена контейнеров воркеров нам пригодятся длянастройки балансировки
        - worker1
        - worker2
    ports:
        - "8080:80"  # Преобразование внешнего порта, входная точка балансировки в контейнер балансировки по порту 8080 
    expose:
        - 80   # haproxy Внешние порты для контейнеров, по кторым будет идти балансировка
```

### haproxy.cfg

```sh
defaults
    log global # Пишем в лог все события
    mode http # Какой тип проксирования трафика - http, tcp
    timeout connect 5000ms # настройки времени соединений
    timeout client 50000ms
    timeout server 50000ms
    stats uri /status # проверка статуса соединенияпо этому адреcу
frontend balancer
    bind 0.0.0.0:80 # балансировщик будет принимать соединения с локальной машины по этому порту от клиента
    default_backend web_backends
backend web_backends # Настраиваем куда балансировщик будет перенаправлять входящие запросы- это два наших воркер контейнера по порту 80
    balance roundrobin # Метод балансировки, см. материалы лекции
    server web1 worker1:80
    server web2 worker2:80
```

### ../worker/Dockerfile

```sh
FROM nginx:alpine # базовый образ веб сервера nginx, построенный на базе дистрибутива Linux Alpine

ARG WORKER_ID # Аргумент, передаваемый в докер файл снаружи при построении

RUN echo "WORKER $WORKER_ID" > /usr/share/nginx/html/index.html # Записываем его в статическую страницу для веб-вервера

EXPOSE 80
```

Запустить контейнеры:

```sh
# Убедитесь что вы в нужной папке
pwd
# DevOps-master/Лабы/Лаб3/haproxy-static-balancer
```

```docker
docker compose up
```

Зайти в окно браузера и набрать: localhost:8080, обновите страницу несколько раз и вы увидите, как меняется цифра (localhost меняем на bridge-ip вашего сервера)
> Если не меняется - откройте вторую страницу в другом браузере

### 3. Многокомпонентый проект docker-compose

Изучить структуру проекта сервис-база данных приложения развернутого в кластере из двух контейнеров

```sh
cd ../python-ui-database
```

```sh
.
├── app # - простое веб-приложение на python, работающее с базой
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
├── db - скрипт создания базы данных
│   └── init.sql
└── docker-compose.yml
└── .env  - файл, содержащий парметры конфигурациисреды запуска 
```

### docker-compose.yml

```yml
version: "3"
services:
  app:
    build: ./app  # контекст для сборки, по-умолчанию ищет Dockerfile в этой директории
    links: # связь контейнеров друг с другом, один из способов организации сетевого взаимодействия
      - db
    ports:  # проброс портов <Host>:<Container>
      - "8090:5000"
    environment: # передаем параметры связи сбазой контейнеру приложения
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_HOST: ${MYSQL_HOST}
      MYSQL_PORT: ${MYSQL_PORT}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      PORT: 5000 # порт приложения
  db:
    image: mysql:5.7
    ports:
      - "32000:3306" # 32000 - порт для подключения к базе внешним клиентом
    volumes: 
      - ./db:/docker-entrypoint-initdb.d/:ro # привязка каталога на диске со скриптами для баы данных хост машины к контейнеру
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD} # тут есть небольшой нюанс в dockre-compose, по-умолчанию параметры из .env не пробрасываются внутрь контейнера, необходиомо их явно задавать, даже если имена совпадают. Это сделано для контролявходных параметров контейнера

  phpmyadmin: # админ клиент для базы mysql
    image: phpmyadmin/phpmyadmin:latest
    restart: always
    links:
      - db
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      PMA_USER: ${MYSQL_USER}
      PMA_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    ports:
      - "8080:80" # админ клиент выставляется по этому порту
```

### .env

```sh
# значения передаются в docker-compose, но не в контейнеры
MYSQL_USER=root
MYSQL_ROOT_PASSWORD=root
MYSQL_HOST=db
MYSQL_PORT=3306
```

> Задача1: Текущее собержимое базы: Зайти в окно браузера и набрать: localhost:8090, вы увидите текущее содержимое базы, сделать снимок экана для отчета

> Задача2: Добавить запись в базу используя встроенный контейнер phpmyadmin
- В дереве слева выбрать knights->favorite_colors
- Вставить запись в таблицу, сделать снимок экана для отчета
- Зайти в окно браузера и набрать: localhost:8090, вы увидите добавленную запись, сделать снимок экана для отчета
