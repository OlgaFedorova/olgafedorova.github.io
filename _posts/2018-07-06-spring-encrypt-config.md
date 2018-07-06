---
layout: post
title: "Spring: механизм шифрования для конфигурационных данных"
permalink: spring-encrypt-config
tags: java spring spring-cloud spring-boot
---

## Unlimited strength Java Cryptography Extension (JCE)
Для поддержки шифрования/дешифрования необходимо установить [unlimited strength Java Cryptography Extension (JCE)](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html). Это zip-файл, который содержит два jar-ника:
-   local_policy.jar    
-   US_export_policy.jar

Данные библиотеки необходимо разместить в **$JAVA_HOME/ jre/lib/security**.

## Ассиметричное шифрование
Генерируем хранилище публичного и приватного ключа, которое в дальнейшем будем использовать для шифрования/дешифрования.

    keytool -genkeypair -alias devkey -keyalg RSA -dname "CN=Dev environment,OU=00CA,O=Home organization,C=RU" -keypass keypass -keystore C:\work\dev-properties-server.jks -storepass storepass

В файл **bootstrap.yml** **config-server** и всех сервисов, которые используют зашифрованную информацию добавляем следующие свойства:

    encrypt:  
    key-store:  
    location: ${ENCRYPT_KEY_STORE_LOCATION}  
    password: ${ENCRYPT_KEY_STORE_PASSWORD}  
    alias: ${ENCRYPT_KEY_STORE_ALIAS}  
    secret: ${ENCRYPT_KEY_STORE_SECRET}

Значение параметров **ENCRYPT_KEY_STORE_LOCATION, ENCRYPT_KEY_STORE_PASSWORD, ENCRYPT_KEY_STORE_ALIAS, ENCRYPT_KEY_STORE_SECRET** передаем как параметры **jvm** при запуске **сервиса и config-server**:

    java -jar -Dspring.profiles.active=dev -DENCRYPT_KEY_STORE_LOCATION= file:/C:/work/dev-properties-server.jks -DENCRYPT_KEY_STORE_PASSWORD=storepass -DENCRYPT_KEY_STORE_ALIAS= devkey -DENCRYPT_KEY_STORE_SECRET=keypass service.jar

## Расшифровка конфигов на стороне клиента
Чтобы конфигурационные параметры расшифровывались на стороне клиента необходимо добавить в файл **bootstrap.yml** значение параметров **config-server**:

    spring.cloud.config.server.encrypt.enabled=false

## Шифрование/Дешифрование
У config-server есть следующие **еndpoint-ы**:
1.  Http-method: POST-запрос  
    **Path: /encrypt**  
    Request Body: то, что нужно зашифровать  
    Respone: зашифрованная информация 
2.  Http-method: POST-запрос  
    **Path: /decrypt**  
    Request Body: то, что нужно расшифровать  
    Respone: расшифрованная информация
    

## Конфигурационные файлы
Для того, чтобы внести зашифрованную информацию в конфиги необходимо:
1.  Воспользоваться сервисом **encrypt** config-сервера.    
2.  Полученную информацию внести в конфиги так:   

	    '{cipher}…….здесь..зашифрованное…сообщение…'

Пример:

    password: '{cipher}AQBktEJNrfYF7ToKUwRJaw2/fCYhKSLH0oHUACt9Ma5rhxPhMG1dWGs7SfsbcOjxu3sDld379iJZBBGZ2YLDBBdl/3P8ceIQXUa8vdPQmnC8qXLB4GZ+RjNa0AjIHj2v1OU+6NUOpMEC26wI7MqTHm3kc2lfAD6Tcii6OtBaKTeoek/LfU6vloAdnE5ZEdY3C+XZz6PMsywZ+IpJgp+NDhqKua3b6PpYEVP1TaszvPDQaf84cjTWwkGAr9r8NNrexKOAAzxAU5nbJt4w2MfCpQCQ9fZOSLYgwh01+V3X7hjjoxchZf1FaybXWCj5j0azJ8xE3qXcF6oBQXcGMupkXUs01AqAor5vr9kANzICYy3J7Iulp/LpR0ppA9XXzvM+MKI='

## Для локальной отладки сервисов в Intelij IDEA
В VM options добавляем:

-DENCRYPT_KEY_STORE_LOCATION=**file:/u01/docker/encrypt-properties/dev-properties-server.jks** -DENCRYPT_KEY_STORE_PASSWORD=keypass -DENCRYPT_KEY_STORE_ALIAS=devkey -DENCRYPT_KEY_STORE_SECRET=storepass

То, что выделено жирным замените на свой путь к файлу **jks.**
