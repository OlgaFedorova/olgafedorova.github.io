---
layout: post
title: "Jdbc input"
permalink: jdbc-input
tags: docker logstash
---
Для чтения данных из **БД** в **Logstash** необходимо настроить **pipeline** с **jdbc-input**.

    input {    
      jdbc {    
        jdbc_driver_library => "/usr/share/logstash/ojdbc6-11.2.0.4.jar"    
        jdbc_driver_class => "Java::oracle.jdbc.driver.OracleDriver"    
        jdbc_connection_string => "jdbc:oracle:thin:@192.168.0.101:1521:test02"   
        jdbc_user => "user"    
        jdbc_password => "password"    
        schedule => "*/5 * * * *"   
        statement_filepath => "/usr/share/logstash/queue-atm.sql"    
        type => "monitoring-queue-atm"    
        jdbc_paging_enabled => "true"    
        jdbc_page_size => "6"    
      }  
    }    
    output {    
      if [type] == "monitoring-queue-atm" {    
        elasticsearch {  
          hosts => [ "192.168.0.101:9200" ]    
          index => "monitoring-queue-atm-%{+YYYY.MM.dd}"    
        }    
      }    
    }

Где,
-   **jdbc_driver_library** - путь к библиотеке ojdbc. "/usr/share/logstash/ojdbc6-11.2.0.4.jar".    
-   **jdbc_driver_class** - класс-драйвер. Пример, "Java::oracle.jdbc.driver.OracleDriver".    
-   **jdbc_connection_string** – строка подключения. Пример, "jdbc:oracle:thin:@192.168.0.101:test02".    
-   **jdbc_user** – имя пользователя.    
-   **jdbc_password** – пароль пользователя.    
-   **schedule** – расписание, по которому получается запрос.    
-   **statement_filepath** - путь к файлу, в котором указан запрос.    
-   **type** – для фильтра получаемой информации.    
-   **jdbc_paging_enabled** - маркер, что запрос, возвращает более одной записи.    
-   **jdbc_page_size** - количество записей, возвращаемое запросом.    

Получаемые по запросу данные передаются в **Elasticsearch**, и сохраняются там под индексом с шаблоном "**monitoring-queue-atm-%{+YYYY.MM.dd}**".

Далее с этим индексом работаем в графическом интерфейсе **Kibana**. Более подробная информация [здесь]({{ site.baseurl }}{% link _posts/2018-07-02-kibana-indexes.md %}).


FROM docker.elastic.co/logstash/logstash:5.6.9    
RUN rm -f /usr/share/logstash/pipeline/logstash.conf    
ADD config/logstash.yml /usr/share/logstash/config/logstash.yml    
ADD ojdbc6-11.2.0.4.jar /usr/share/logstash/

  

**Старт docker-контейнера:**

docker run --rm -d --net=host -v /u01/docker/logstash/pipeline:/usr/share/logstash/pipeline -v /u01/docker/logstash/queue-atm.sql:/usr/share/logstash/queue-atm.sql -v /u01/docker/logstash/data:/usr/share/logstash/data -v /u01/docker/logstash/logs:/usr/share/logstash/logs -v /u01/docker/logs:/home/logs --name=logstash logstash:5.6.9

  
**Монтируемые каталоги:**
-   файл с sql-запросом:  
    **-v /u01/docker/logstash/queue-atm.sql:/usr/share/logstash/queue-atm.sql**
  -   Каталог с файлами, в которых настроены pipeline:  
    **-v /u01/docker/logstash/pipeline:/usr/share/logstash/pipeline**  
    По умолчанию **Logstash** в каталоге **/usr/share/logstash/pipeline** все файлы воспринимает, как pipeline.
