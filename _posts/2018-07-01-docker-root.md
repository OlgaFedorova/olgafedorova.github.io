---
layout: post
title: "Изменить Docker Root Dir"
permalink: docker-root
tags: docker
---
Иногда требуется изменить **Docker Root Dir**, например когда недостаточно места на диске.

Необходимо в файл **/etc/sysconfig/docker** в **OPTIONS** добавить **“ -g /u01/.docker-tmp”**.

Пример:

    OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false -g /u01/.docker-tmp'
 
Далее перезапускаем **docker**:

    systemctl daemon-reload   
    systemctl restart docker
 
Проверяем изменения:

    docker info
