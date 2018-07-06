---
layout: post
title: "Spring boot admin: изменение уровня логирования"
permalink: spring-boot-admin-logging
tags: java logback spring spring-boot spring-cloud
---
**Уровни логирования:**
-   **TRACE** на этом уровне выводятся все сообщения уровня **TRACE, DEBUG, INFO, WARN, ERROR.**    
-   **DEBUG** на этом уровне выводятся все сообщения уровня **DEBUG, INFO, WARN, ERROR.**    
-   **INFO** на этом уровне выводятся все сообщения уровня **INFO, WARN, ERROR.**    
-   **WARN** на этом уровне выводятся все сообщения уровня **WARN, ERROR.**    
-   **ERROR** на этом уровне выводятся все сообщения уровня **ERROR.**  

По умолчанию для приложения включен уровень **INFO.**

Для возможности динамического изменения уровня логирования (без пересборки приложения) в конфигурацию **logback** добавлена возможность поддержки **jmx-конфигурирования**:

    <configuration>  
    …  
    <jmxConfigurator/>  
    </configuration>

  
Настройка логирования через **Spring Boot Admin** осуществляется в строке соответствующего сервиса в пункте меню **Details => Logging.**

![logging]({{ site.url }}/assets/2018-07-06-spring-boot-admin-logging/1.png)

Переключаемся, нажимая левой кнопкой мыши, на необходимый уровень и настройки сразу же вступают в силу:

![logging]({{ site.url }}/assets/2018-07-06-spring-boot-admin-logging/2.png)
