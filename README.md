# Лабораторная работа №9: Оптимизация образов контейнеров

**Питропов Александр,группа I2302**  
**Дата выполнения:26.04.2025** 

## Цель работы
Целью данной лабораторной работы является знакомство с методами оптимизации Docker-образов, а также практическое применение различных техник оптимизации: удаление неиспользуемых зависимостей и временных файлов, уменьшение количества слоев, использование минимального базового образа, перепаковка образа.

## Задание
1. Сравнить различные методы оптимизации образов:
   - Удаление неиспользуемых зависимостей и временных файлов
   - Уменьшение количества слоев
   - Минимальный базовый образ
   - Перепаковка образа
   - Использование всех методов
2. Провести сборку и оптимизацию образов.
3. Провести тестирование.
4. Создать отчет с ответами на вопросы и выводами.

## Подготовка
На компьютере должен быть установлен Docker.
Создаем репозиторий `containers09`, внутри него создаем папку `site`, в которую помещаем файлы сайта (`html`, `css`, `js`).

## Выполнение работы

### 1. Базовый образ без оптимизаций
Создаем файл `Dockerfile.raw`:

```Dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get upgrade -y

RUN apt-get install -y nginx

COPY site /var/www/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Собираем образ:
```bash
docker image build -t mynginx:raw -f Dockerfile.raw .
```

![image](https://i.imgur.com/xL1X0ML.jpeg)

### 2. Удаление неиспользуемых зависимостей и временных файлов
Создаем файл `Dockerfile.clean`:

```Dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get upgrade -y

RUN apt-get install -y nginx

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY site /var/www/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Собираем образ:
```bash
docker image build -t mynginx:clean -f Dockerfile.clean .
docker image list
```

![image](https://i.imgur.com/zFH84bo.jpeg)

### 3. Уменьшение количества слоев
Создаем файл `Dockerfile.few`:

```Dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y nginx && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY site /var/www/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Собираем образ:
```bash
docker image build -t mynginx:few -f Dockerfile.few .
docker image list
```

![image](https://i.imgur.com/RSH263D.jpeg)

### 4. Минимальный базовый образ
Создаем файл `Dockerfile.alpine`:

```Dockerfile
FROM alpine:latest

RUN apk update && apk upgrade

RUN apk add nginx

COPY site /var/www/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Собираем образ:
```bash
docker image build -t mynginx:alpine -f Dockerfile.alpine .
docker image list
```

![image](https://i.imgur.com/OBrqPUS.jpeg)

### 5. Перепаковка образа
Перепаковываем базовый образ:
```bash
docker container create --name mynginx mynginx:raw
docker container export mynginx | docker image import - mynginx:repack
docker container rm mynginx
docker image list
```

![image](https://i.imgur.com/RVWrWa6.jpeg)

![image](https://i.imgur.com/0OKZAL7.jpeg)

### 6. Использование всех методов
Создаем файл `Dockerfile.min`:

```Dockerfile
FROM alpine:latest

RUN apk update && apk upgrade && \
    apk add nginx && \
    rm -rf /var/cache/apk/*

COPY site /var/www/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Собираем и перепаковываем образ:
```bash
docker image build -t mynginx:minx -f Dockerfile.min .
docker container create --name mynginx mynginx:minx
docker container export mynginx | docker image import - mynginx:min
docker container rm mynginx
docker image list
```

![image](https://i.imgur.com/FcRW9Od.jpeg)

![image](https://i.imgur.com/s1FlQtM.jpeg)

## Размер образов

![image](https://i.imgur.com/EKvaAlD.jpeg)

## Ответы на вопросы

**1. Какой метод оптимизации образов вы считаете наиболее эффективным?**
Наиболее эффективным методом является использование минимального базового образа, например `alpine`, в сочетании с очисткой кэшированных данных. Этот подход позволяет значительно сократить размер образа, сохраняя при этом его функциональность.

**2. Почему очистка кэша пакетов в отдельном слое не уменьшает размер образа?**
Каждый слой в Docker представляет собой разницу по сравнению с предыдущим состоянием. Если очистка происходит в новом слое, предыдущий слой с закешированными данными остается в истории слоев образа. Таким образом, очистка должна производиться в рамках того же слоя, где происходит установка пакетов.

**3. Что такое перепаковка образа?**
Перепаковка образа — это процесс создания нового образа из контейнера, который уже запущен. Этот способ помогает избавиться от истории слоев и уменьшить итоговый размер образа, однако потеряна информация о первоначальных инструкциях Dockerfile.

## Вывод
В ходе выполнения лабораторной работы были рассмотрены основные методы оптимизации Docker-образов. Было установлено, что:
- Очистка временных файлов и кэшированных данных помогает уменьшить размер образа, но если выполняется в отдельном слое — эффекта не дает.
- Объединение команд в один слой позволяет дополнительно сократить размер.
- Использование минимального базового образа, например `alpine`, значительно снижает размер конечного контейнера.
- Перепаковка образов приводит к уменьшению размеров за счет потери истории слоев.
На практике наилучший результат достигается комплексным применением всех перечисленных методов. Правильная оптимизация Docker-образов позволяет не только уменьшить их размер, но и ускорить доставку и развертывание приложений.
