---
layout: post
title: "Настройка SSL в Windows"
permalink: set-ssl-in-windows
tags: ibm-mq ssl windows
---
- Добавить в переменные среды:

    AMQ_SSL_V3_ENABLE=1
    AMQ_SSL_WEAK_CIPHER_ENABLE=RC4_MD5_US

[ssl]({{ site.url }}/assets/2018-07-06-set-ssl-in-windows/ssl_5.png)
- Перезапустить MQ
- Запустить KeyManager (утилита по адресу **App_IBM_MQ_Install** \bin\strmqikm.exe, где **App_IBM_MQ_Install** – установочная директория сервера IBM MQ)
- Создать хранилище. Указать путь, где будет располагаться хранилище. Рекомендуется сохранять файлы хранилища в папке ssl менеджера очередей, к примеру: C:\Program Files (x86)\IBM\WebSphere MQ\Qmgrs\ONE!QM\ssl.
При указании пароля на хранилище обязательно выбрать опцию **«Сохранить пароль в файле»**.

[ssl]({{ site.url }}/assets/2018-07-06-set-ssl-in-windows/ssl_6.png)

[ssl]({{ site.url }}/assets/2018-07-06-set-ssl-in-windows/ssl_7.png)

- Создать запрос на выпуск сертификата с параметрам

[ssl]({{ site.url }}/assets/2018-07-06-set-ssl-in-windows/ssl_8.png)

- Свой сертификат загружаем в Personal Certificate путем Recieve
- Cертификат удостоверяющего центра загружаем в Signer Certificate
- Указываем путь к созданному хранилищу для менеджера очередей без расширения .kdb

[ssl]({{ site.url }}/assets/2018-07-06-set-ssl-in-windows/ssl_9.png)

- Для канала настраиваем раздел MCA: указываем пользователя, у которого есть доступ к каналу и очередям менеджера очередей, при этом он должен быть заведен на уровне ОС

[ssl]({{ site.url }}/assets/2018-07-06-set-ssl-in-windows/ssl_10.png)

- Для канала настраиваем раздел SSL:

**SSL Cipher Spec**: параметр определяющий какой тип шифрования используется. Параметр должен быть одинаковым на обоих сторонах, тех кто запрашивает соединение и на уровне MQ, куда приходит запрос.

[ssl]({{ site.url }}/assets/2018-07-06-set-ssl-in-windows/ssl_11.png)

- Перезапустить MQ
