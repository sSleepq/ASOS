# Лабораторная работа №9
### Тема : Docker — контейнеризатор приложений
### Цель : Познакомится с технологией контейнеризации Docker
---
### Порядок работы :

### 1. Знакомство с Docker:  

Для первоночального знаковства с Docker советую прочитать первую главу статьи:  
Изучаем Docker, часть 1: основы  
<a href="https://habr.com/ru/companies/ruvds/articles/438796/">клик</a>  
Освоив этот материал, вы разберётесь с основами Docker

### 2. Первая работа с Docker 

Чтобы начать работу с Docker нужно подготовить почву для разработки. Для этого зайдем на офицальный сайт с документацией докер и скачаем его:  
`Нажми на картинку`  
<a href="https://docs.docker.com/"><img src="src\img\lb9\1.png"></a>

`ДЛЯ ОСОБО ЛЕНИВЫХ :)`  

1. Добавим репозиторий  
`Команды выполнять построчно`
```sh
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```sh
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```sh
sudo apt-get update
```
2. Установка Docker пакетов
```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
3. Проверим работоспособность Docker
```sh
sudo docker run hello-world
```

### 3. Первая работа с Docker 

Чтобы познакомится с основными командами Docker вам понадобится сайт Play With Docker  
`Нажми на картинку`  
<a href="https://training.play-with-docker.com/"><img src="src\img\lb9\2.png" width=600px></a>

> В открывшемся окне мы видим несколько Этапов изучения Docker, нас интересует Stage 1 : The Basics

<img src="src\img\lb9\3.png" width=600px>

> Данный этап предлагает три базовых темы для знакомства с Docker

<img src="src\img\lb9\4.png" width=600px>

> При открывании какой либо из тем, у вас на экране отобразится страничка с лабораторной работой, в которой описываются все действия выполняемые пользователем  

<img src="src\img\lb9\5.png" width=600px>

`Выполните все темы в Этапе - Stage 1 | отчет для этих лабораторных не нужно`

### Задание для выполнения (Выполнить отчет)
`Выполнение начинать после прохождения этапа Stage 1`
1) Скачать образ контейнера Nginx
2) Найти и скачать (либо написать самому) две hmtl страницы
3) Написать Dockerfile'ы которые при развертывании будут заменять стандартный файл index.html на ваш
4) Пробросить порты с контейнеров на основную машину, для доступа к сервисам Nginx
5) Продемонстрировать работу веб страниц

Схема работы

<img src="src\img\lb9\task.png">