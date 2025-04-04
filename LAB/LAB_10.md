# Лабораторная работа №10
### Тема : Docker — контейнеризатор приложений (Продвинутый)
### Цель : Расширить познания контейнеризации Docker
---
### Порядок работы :

### 1. Запускаем контейнер - docker run, docker logs:  

Запустим контейнер с СУБД командой docker run:

```sh
docker run mysql
```

<img src="src\img\lb10\1.png" width=600px>

Исправляем ошибки, добавим аргумент -e для передачи переменной окружения, зададим пароль:

```sh
docker run -e MYSQL_ROOT_PASSWORD=password mysql
```
> База запустилась, но как будто мы просто запустили ее командой mysql из консоли.

Теперь попробуем запустить в фоновом режиме с аргументом -d, кроме того, дадим ему вменяймое имя --name $NAME:

```sh
docker run -d -e MYSQL_ROOT_PASSWORD=password --name db1 mysql
```

<img src="src\img\lb10\2.png" width=600px>

> Получилось, в ответ Docker выдал нам хэш-ID контейнера

Посмотрим логи контейнера по его ID командой `docker logs`:

```sh
docker logs $ID
# Это можно сделать и по имени
docker logs db1
```

### 2. Списки контейнеров - docker ps

Посмотрим списки контейнеров с помощью команды `docker ps`:

```sh
docker ps
docker ps -a # включая завершенные
```

### 3. Подключаемся к работающему контейнеру - docker exec

Подключимся (запустим еще один процесс внутри контейнера) с помощью команды docker exec:

```sh
# i - interactive - держать STDIN открытым
# t - tty - создать псевдо-tty
docker exec -it db1 /bin/bash
```

<img src="src\img\lb10\3_0.png" width=600px>

<img src="src\img\lb10\3_1.png" width=600px>

Выполним некоторые команды в контейнере

```sh
touch file.txt
echo "Hello" >> file2.txt
exit
```

### 4. Список изменений - docker diff

Просмотрим список изменений в слое на ФС с помощью команды `docker diff`:

```sh
docker diff db1
```

<img src="src\img\lb10\4.png" width=600px>

Наблюдаем наличие новых файлов file.txt и file2.txt.

### 5. Не теряем данные - docker volume

Завершим контейнер нежно с помощью `stop`, сразу завершим с помощью `kill` и удалим остатки (из спика завершенных, включая логи контейнера) c помощью `rm`:

```sh
docker stop db1
docker kill db1
docker rm db1
```

Запустим контейнер обратно:

```sh
docker run -d -e MYSQL_ROOT_PASSWORD=password --name db1 mysql
```

Выведем содержимое файла:

```sh
docker exec -it db1 cat file2.txt
```

О ужас! Его нет. Как и всего содержимого базы. Все данные удалились при завершении контейнера.

Чтобы этого избежать запустим контейнер с томом:

```sh
docker run --rm -d \
	-v mysql:/var/lib/mysql \
	-v mysql_config:/etc/mysql \
	--name db1 \
	-e MYSQL_ROOT_PASSWORD=password \
	mysql
```

`Если не получилось выполнить команду просто скопировав ее, копируйте нижнюю`

```sh
docker run --rm -d -v mysql:/var/lib/mysql -v mysql_config:/etc/mysql --name db1 -e MYSQL_ROOT_PASSWORD=password mysql
```

Внесем изменения:

```sh
docker exec -it db1 mysql -ppassword
```

```sh
create database testdb;
create database blog;
show schemas;
```
> После необходимо выйти из интерактивного режима `Ctrl + D`

Завершим контейнер:

```sh
docker stop db1
```

А теперь перезапустим и убедимся, что базы testdb и blog не исчезли:

```sh
docker run --rm -d \
	-v mysql:/var/lib/mysql \
	-v mysql_config:/etc/mysql \
	--name db1 \
	-e MYSQL_ROOT_PASSWORD=password \
	mysql
```

```sh
docker exec -it db1 mysql -ppassword -e "show schemas;"
```

### 6. Контейнер Adminer

Запустим образ Adminer (скачается автоматически). Для того, чтобы попасть из виртуальки внутрь контейнера на порт 8080 укажем ключик `-p HostPort:ContainerPort`:

```sh
docker run -d -p 8080:8080 --name adminer adminer
```

Подключимся в браузере на хосте к http://127.0.0.1:8080/
> В вашем случае на Windows `ip-адрес-сервера:8080`

<img src="src\img\lb10\5.png" width=600px>

Adminer (бывший phpMinAdmin) — это легковесный инструмент администрирования MySQL, PostgreSQL, SQLite, MS SQL и Oracle. Проект родился как «облегчённый» вариант phpMyAdmin. Распространяется в форме одиночного PHP-файла размером около 380 KB, который является результатом компиляции исходных php- и js-файлов с помощью специального PHP-скрипта. Т.о. контейнер с ним содержит php-сервер и один php-скрипт.

> Однако как бы мы не пытались подключиться к базе - ничего не выйдет. Котейнеры не связаны по сети

### 7. Сети - docker network

Для начала попробуем связать контейнеры простым способом, завершим предыдущий контейнер Adminer и запустим новый, с параметром `--link Container:AliasName`

```sh
docker rm -f adminer
docker run -d -p 8080:8080 --link db1:mysql --name adminer adminer
```

Теперь подключимся к базе по ее Alias из контейнера adminer:

<img src="src\img\lb10\6.png" width=600px>

<img src="src\img\lb10\7.png" width=600px>

Неплохо, но такой способ считается устаревшим. К тому же, это может быть не всегда удобно. Теперь создадим новую сеть:

```sh
docker network create cluster
```

Проверим как создалась сеть с параметрами по умолчанию:

<img src="src\img\lb10\8.png" width=600px>

Проверим в какой сети работают сейчас Adminer и MySQL:

```sh
docker inspect db1
docker inspect adminer | egrep "IPAddress|Gateway|IPPrefixLen"
```

<img src="src\img\lb10\9.png" width=600px>

Теперь пересоздадим контейнеры СУБД и Adminer в этой сети:

```sh

docker rm -f db1
docker rm -f adminer

# Добавим к предыдущей команде запуска: --net NetName
docker run --rm -d \
	-v mysql:/var/lib/mysql \
	-v mysql_config:/etc/mysql \
	--name db1 \
	-e MYSQL_ROOT_PASSWORD=password \
	--net cluster \
	mysql

# А теперь аналогично для Adminer
docker run -d -p 8080:8080 --net cluster --name adminer adminer

```

Пробуем подключиться к базе по IP-адресу:

<img src="src\img\lb10\10.png" width=600px>

Для диагностики сетей есть полезный образ, подробное применение рассматривается по ссылке. Попробуем запустить его в интерактивном режиме и проверить сеть контейнеров с помощью `nmap`:

```sh
docker run -it --net cluster nicolaka/netshoot
```

```sh
nmap db1
```

```sh
nmap adminer
```