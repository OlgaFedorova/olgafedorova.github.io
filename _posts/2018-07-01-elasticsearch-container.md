---
layout: post
title: "Контейнер Elasticsearch"
permalink: elasticsearch-container
tags: docker elasticsearch
---


_**Образ Elasticsearch**_ создан на основе официального образа **docker.elastic.co/elasticsearch/elasticsearch:5.6.9.**
В существующий образ внесены изменения согласно прилагаемому docker-файлy:  

    FROM docker.elastic.co/elasticsearch/elasticsearch:5.6.9  
    USER root  
    COPY ./limits.conf /etc/security/limits.conf  
    RUN chown -R elasticsearch:elasticsearch /usr/share/elasticsearch/data  
    RUN chown -R elasticsearch:elasticsearch /usr/share/elasticsearch/logs
    USER elasticsearch

Где,
-   **elasticsearch.yml** – это конфигурационный файл.    
-   **/usr/share/elasticsearch/data** – путь куда пишутся индексы    
-   **/usr/share/elasticsearch/logs** – путь куда пишутся логи
-   **limits.conf** - cистемный файл **/etc/security/limits.conf**, в котором указаны настройки для пользователя **elasticsearch**: максимальное количество открытых файловых дескрипторов (**nofile**) и максимальное количество запущенных тредов (**nproc**).

elasticsearch - nofile 65536  
elasticsearch - nproc 2048

Данные каталоги монтируются из внешних каталогов хоста при старте контейнера:

**Проблемы:**

Для того чтобы контейнер запустился необходимо :
-   дать права на монтируемые каталоги    

    chmod 0777 /u01/docker/elasticsearch/node1/logs  
    chmod 0777 /u01/docker/elasticsearch/node1/data  
    chmod 0777 /u01/docker/elasticsearch/node1/config/  
    chmod 0777 /u01/docker/elasticsearch/node1/config/elasticsearch.yml  
    chmod 0777 /u01/docker/elasticsearch/node2/logs  
    chmod 0777 /u01/docker/elasticsearch/node2/data  
    chmod 0777 /u01/docker/elasticsearch/node2/config/  
    chmod 0777 /u01/docker/elasticsearch/node2/config/elasticsearch.yml
    
-   Установить значение max virtual memory areas vm.max_map_count:

sysctl -w vm.max_map_count=262144

**Кластер**
Elasticsearch развернут в двух нодах.
Пример конфигурационного файла [elasticsearch.yml](https://github.com/OlgaFedorova/dockers/blob/master/elasticsearch/node1/config/elasticsearch.yml) для node1.
Пример конфигурационного файла [elasticsearch.yml](https://github.com/OlgaFedorova/dockers/blob/master/elasticsearch/node2/config/elasticsearch.yml) для node2.

**Старт node1:**

    docker run --rm -d --net=host --env "ES_JAVA_OPTS=""-Xms2g -Xmx2g""" --ulimit memlock=-1:-1 -v /u01/docker/elasticsearch/node1/logs:/usr/share/elasticsearch/logs -v /u01/docker/elasticsearch/node1/data:/usr/share/elasticsearch/data -v /u01/docker/elasticsearch/node1/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml --name=elasticsearch1 elasticsearch:5.6.9

**Старт node2:**

    docker run --rm -d --net=host --env "ES_JAVA_OPTS=""-Xms2g -Xmx2g""" --ulimit memlock=-1:-1 -v /u01/docker/elasticsearch/node2/logs:/usr/share/elasticsearch/logs -v /u01/docker/elasticsearch/node2/data:/usr/share/elasticsearch/data -v /u01/docker/elasticsearch/node2/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml --name=elasticsearch2 elasticsearch:5.6.9

