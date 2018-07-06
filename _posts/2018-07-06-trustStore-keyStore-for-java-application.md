---
layout: post
title: "trustStore и keyStore для java-приложения"
permalink: trustStore-keyStore-for-java-application
tags: ibm-mq ssl windows java jms
---
- Создаем **trustStore в формате jks**, указываем для него пароль. 

Переходим в раздел **«Личные сертификаты»**, нажимаем на кнопку **«Импортировать»**. Выбираем базу данных ключей для импорта.

![ssl]({{ site.url }}/assets/2018-07-06-trustStore-keyStore-for-java-application/ssl_1.png) 

![ssl]({{ site.url }}/assets/2018-07-06-trustStore-keyStore-for-java-application/ssl_2.png)

Переходим в раздел **«Личные сертификаты»**, нажимаем на кнопку **«Импортировать»**. Выбираем базу данных ключей для импорта.

![ssl]({{ site.url }}/assets/2018-07-06-trustStore-keyStore-for-java-application/ssl_3.png)

В открывшемся списке выбираем для импорта только **«Подписывающие сертификаты»** и нажимаем на кнопку **«ОК»**.

**«Личные сертификаты»** будем импортировать в следующем пункте в **keyStore**.

![ssl]({{ site.url }}/assets/2018-07-06-trustStore-keyStore-for-java-application/ssl_4.png)

- По указанному пути создан **trustStore.jks.**  
- Создаем **keyStore в формате jks**, указываем для него пароль (аналогично созданию trustStore)
    
Далее импортируем **«Личные сертификаты»** аналогично импорту «Подписывающих сертификатов» в trustStore.

По указанному пути создан **keyStore.jks.**
