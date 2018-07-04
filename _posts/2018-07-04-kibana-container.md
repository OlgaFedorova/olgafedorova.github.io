---
layout: post
title: "Контейнер Kibana"
permalink: kibana-container
tags: docker kibana
---
**Образ Kibana** создан на основе официального образа **docker.elastic.co/kibana/kiban:5.6.9**.

В существующий образ внесены изменения согласно прилагаемому docker-файлy:

    FROM docker.elastic.co/kibana/kibana:5.6.9  
    ADD ./config/kibana.yml /usr/share/kibana/config/kibana.yml    
    EXPOSE 5601

Где **kibana.yml** – это конфигурационный файл. Пример, [конфигурационного файла.](https://github.com/OlgaFedorova/dockers/blob/master/kibana/config/kibana.yml)

Стартуем командой:

    docker run --rm -d --net=host --name=kibana kibana:5.6.9
