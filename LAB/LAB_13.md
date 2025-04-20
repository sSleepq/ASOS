# Лабораторная работа №13
### Тема : MongoDB
---
### Порядок работы :

1) Первым шагом является установка необходимых пакетов, необходимых во время установки. Для этого выполните следующую команду.

```sh
sudo apt install -y software-properties-common gnupg apt-transport-https ca-certificates
```

2) Установка MongoDB

```sh
sudo apt install -y mongodb
```

3) Импортируем открытый ключ для MongoDB в вашу систему, используя команду wget следующим образом:

```sh
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
```

4) Добавьте репозиторий APT MongoDB в /etc/apt/sources.list.d каталог.

```sh
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
```

Команда добавляет mongodb-org-5.0.list файл в /etc/apt/sources.list.d/ каталог. Этот файл содержит следующую строку:

```sh
deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse
```

5) После добавления репозитория перезагрузите локальный индекс пакета.

```sh
sudo apt update
```

6) Установите mongodb-org мета-пакет, предоставляющий MongoDB.

```sh
sudo apt install -y mongodb-org
```

7) По завершении установки вы можете проверить установленную версию MongoDB, как показано на рисунке.

```sh
mongod --version
```

8) По умолчанию служба MongoDB отключена при установке. Вы можете убедиться в этом, выполнив команду

```sh
sudo systemctl status mongod
```

> Запустите службу и проверить статус.

9) Показанная команда подключается к базе данных и отображает текущую версию MongoDB, URL сервера и порт, который она прослушивает.

```sh
mongo --eval 'db.runCommand({ connectionStatus: 1 })'
```

10) После проверки того, что служба работает должным образом, теперь вы можете включить запуск MongoDB при загрузке, как показано на рисунке.

```sh
sudo systemctl enable mongod
```

11) Чтобы получить доступ к MongoDB, выполните следующую команду:

```sh
mongosh
```