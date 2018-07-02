---
layout: post
title: "Контейнер Logstash"
permalink: logstash-container
tags: docker logstash
---
_**Образ Logstash**_ создан на основе официального образа **docker.elastic.co/elasticsearch/logstash:5.6.9.**

В существующий образ внесены изменения согласно прилагаемому **docker-файлy**:

    FROM docker.elastic.co/logstash/logstash:5.6.9     
    RUN rm -f /usr/share/logstash/pipeline/logstash.conf  
    ADD pipeline/logstash.conf /usr/share/logstash/pipeline/logstash.conf  
    ADD config/logstash.yml /usr/share/logstash/config/logstash.yml

Где,
**logstash.conf** – файл в котором настроен основной pipeline.

    input {  
      file {  
        path => [ "/home/logs/*.log" ]  
        type => "microservices"  
        codec => multiline {  
        pattern => "^(%{TIMESTAMP_ISO8601})"  
        negate => true  
        what => "previous"}
      }
    }  
    filter {  
      grok {  
        match => [ "message", "(?m)%{TIMESTAMP_ISO8601:timestamp} %{NUMBER:duration} %{DATA:application} \[%{DATA:thread}\] %{LOGLEVEL:logLevel} %{DATA:class} \- %{GREEDYDATA:message}" ] 
        overwrite => ["message"] 
      } 
    }  
    output { 
      if [type] == "microservices" {  
        elasticsearch {  
          hosts => [ "192.168.0.101:9200" ]  
          index => "microservices-%{+YYYY.MM.dd}"  
        }  
      }  
    }

Для возможности парсинга логов был изменен **pattern** для **logback**:

%d{yyyy-MM-dd HH:mm:ss.SSS} %-4relative ${spring.application.name} [%thread] %-5level %logger{35} - %msg%n

**logstash.yml** – это конфигурационный файл.
    
    http.host: "192.168.0.101"  
    path.config: /usr/share/logstash/pipeline  
    path.data: /usr/share/logstash/data  
    path.logs: /usr/share/logstash/logs  
    xpack.monitoring.elasticsearch.url: "http://192.168.0.101:9200"

Каталоги **/usr/share/logstash/data**, **/usr/share/logstash/logs** монтируются из внешних каталогов хоста при старте контейнера.

**Проблемы:**
Для того чтобы контейнер запустился необходимо:
-   дать права на монтируемые каталоги

    chmod 0777 /u01/docker/logstash/data
    chmod 0777 /u01/docker/logstash/logs

Старт командой:

    docker run --rm -d --net=host -v /u01/docker/logstash/data:/usr/share/logstash/data -v /u01/docker/logstash/logs:/usr/share/logstash/logs -v /u01/docker/logs:/home/logs --name=parus-logstash parus-logstash:5.6.9

- [logstash](https://github.com/OlgaFedorova/dockers/tree/master/logstash)

