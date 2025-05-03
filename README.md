# Лабораторная работа №9: Оптимизация образов контейнеров

### Выполнила: Годорожа Оксана, группа I2302

## Цель работы

Целью работы является знакомство с методами оптимизации образов.

## Задание

Сравнить различные методы оптимизации образов:

- Удаление неиспользуемых зависимостей и временных файлов
- Уменьшение количества слоев
- Минимальный базовый образ
- Перепаковка образа
- Использование всех методов

## Подготовка

Для выполнения данной работы необходимо иметь установленный на компьютере [Docker](https://www.docker.com/).

## Выполнение

Создаю репозиторий `containers09` и копирую его себе на компьютер. В папке `containers09` создаю папку `site` и помещаю в нее файлы сайта (html, css, js).

Для оптимизации использую образ определенный следующим `Dockerfile.raw`:

```Dockerfile
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y

# install nginx
RUN apt-get install -y nginx

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Создаю его в папке `containers09` и собераю образ с именем `mynginx:raw`:

```bash
docker image build -t mynginx:raw -f Dockerfile.raw .
```

### Удаление неиспользуемых зависимостей и временных файлов

Удаляю временные файлы и неиспользуемые зависимости в `Dockerfile.clean`:

```Dockerfile
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y

# install nginx
RUN apt-get install -y nginx

# remove apt cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Собираю образ с именем `mynginx:clean` и проверяю его размер:

```bash
docker image build -t mynginx:clean -f Dockerfile.clean .
docker image list
```

### Уменьшение количества слоев

Уменьшаю количество слоев в `Dockerfile.few`:

```Dockerfile
# create from ubuntu image
FROM ubuntu:latest

# update system
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y nginx && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Собираю образ с именем `mynginx:few` и проверяю его размер:

```bash
docker image build -t mynginx:few -f Dockerfile.few .
docker image list
```

### Минимальный базовый образ

Заменяюе базовый образ на `alpine` и пересобираю образ:

```Dockerfile
# create from alpine image
FROM alpine:latest

# update system
RUN apk update && apk upgrade

# install nginx
RUN apk add nginx

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```
![image](/images/1.png)

Собираю образ с именем `mynginx:alpine` и проверяю его размер:

```bash
docker image build -t mynginx:alpine -f Dockerfile.alpine .
docker image list
```

### Перепаковка образа

Перепаковываю образ `mynginx:raw` в `mynginx:repack`:

```bash
docker container create --name mynginx mynginx:raw
docker container export mynginx | docker image import - mynginx:repack
docker container rm mynginx
docker image list
```

![image](/images/2.png)

---

Я использовала метод без пайпа (|), чтобы иметь возможность контролировать каждый шаг процесса. Это помогает легче отлаживать и выявлять ошибки на каждом этапе, а также предоставляет большую гибкость в случае необходимости изменить параметры или обработать данные перед следующим действием.

(А так же я решала проблему примерно 2 часа, и помог только тот способ который имеется на скриншоте :) )

### Использование всех методов

Создаю образ `mynginx:min` с использованием всех методов:

```Dockerfile
# create from alpine image
FROM alpine:latest

# update system, install nginx and clean
RUN apk update && apk upgrade && \
    apk add nginx && \
    rm -rf /var/cache/apk/*

# copy site
COPY site /var/www/html

# expose port 80
EXPOSE 80

# run nginx
CMD ["nginx", "-g", "daemon off;"]
```

Собираю образ с именем `mynginx:minx` и проверяю его размер. Перепаковываю образ `mynginx:minx` в `mynginx:min`:

```bash
docker image build -t mynginx:minx -f Dockerfile.min .
docker container create --name mynginx mynginx:minx
docker container export mynginx | docker image import - myngin:min
docker container rm mynginx
docker image list
```

## Запуск и тестирование

Проверяю размеры образов.

```bash
docker image list
```

Таблица с размерами образов.

![image](/images/3.png)

## Ответы на вопросы:

1. Какой метод оптимизации образов вы считаете наиболее эффективным?
 ---
Самым эффективным методом является использование минимального базового образа (например, alpine) и объединение всех команд в одном слое. Это минимизирует размер образа и количество слоев.

2. Почему очистка кэша пакетов в отдельном слое не уменьшает размер образа?
 ---
 Очистка кэша в отдельном слое не уменьшает размер образа, потому что Docker сохраняет каждый слой, включая слой с кэшем, даже если он удаляется в следующем слое.

3. Что такое перепаковка образа?
---
Перепаковка образа — это создание нового образа из контейнера, который не сохраняет промежуточные слои и историю сборки, что уменьшает размер, но теряет информацию о процессе сборки.

## Выводы

В ходе выполнения лабораторной работы были изучены различные методы оптимизации Docker-образов. Наиболее эффективным способом оптимизации является использование минимального базового образа (например, Alpine), а также уменьшение количества слоев путем объединения команд. Очистка кэша пакетов в отдельном слое не приводит к значительному уменьшению размера образа, так как Docker сохраняет все слои. Перепаковка образа позволяет удалить историю сборки, но делает образ менее удобным для отладки. В итоге, комбинация всех методов позволяет достичь наименьшего размера образа при сохранении функциональности.

## Библиография

Docker Documentation - https://docs.docker.com/

Dockerfile Best Practices - https://docs.docker.com/develop/dev-best-practices/

Optimizing Docker Images - https://www.digitalocean.com/community/tutorials/how-to-optimize-docker-images