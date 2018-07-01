---
layout: post
title: "Docker-контейнер Elasticsearch-curator"
permalink: elasticsearch-curator-container
tags: docker elasticsearch elasticsearch-curator
---
Образ **Elasticsearch-curator** создан на основе образа из dockerfile:

    FROM alpine:latest
    RUN apk --update add python py-pip && \  
    pip install elasticsearch-curator && \  
    pip install requests-aws4auth && \  
    rm -rf /var/cache/apk/*

В существующий образ внесены изменения согласно прилагаемому **docker-файлy**:

    FROM elasticsearch-curator:latest  
    ADD ./entrypoint.sh /entrypoint.sh  
    WORKDIR /usr/share/curator  
    RUN chmod +x /entrypoint.sh    
    ENV CRON 00 5 * * *  
    ENV CONFIG_FILE /usr/share/curator/config/curator.yml  
    ENV COMMAND /usr/share/curator/config/actions.yml    
    ENTRYPOINT ["/entrypoint.sh"]

Где:
-   **entrypoint.sh** – это скрипт запуска cron-а. 
-   **curator.yml** – конфигурационный файл. 
-   **actions.yml** – файл с описанными действиями. 
Файлы **curator.yml, actions.yml** указываются монтированием каталога при старте контейнера.

**Стартуем командой:**

    docker run --rm -d -v /u01/docker/elasticsearch-curator-by-cron/config:/usr/share/curator/config --net=host --name=elasticsearch-curator-by-cron elasticsearch-curator-by-cron:0.0.1
- [elasticsearch-curator](https://github.com/OlgaFedorova/dockers/tree/master/elasticsearch-curator)
- [elasticsearch-curator-by-cron](https://github.com/OlgaFedorova/dockers/tree/master/elasticsearch-curator-by-cron)

