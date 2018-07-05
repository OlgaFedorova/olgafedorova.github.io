---
layout: post
title: "Конфликт соединений через Bridge docker0"
permalink: conflict-connections-bridge-docker0
tags: docker linux
---
***Проблема:***
После обновления системы не работает wi-fi, единственное доступное соединение Bridge docker0.
![ifconfig]({{ site.url }}/assets/2018-07-05-conflict-connections-bridge-docker0/1.png)

***Решение:***
- Делаем копию файла /etc/default/grub:

    sudo cp /etc/default/grub /etc/default/grub.bak

- В файле /etc/default/grub меняем:

    GRUB_DEFAULT=0

на

    GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 4.13.0-45-generic"

вместо Ubuntu, with Linux 4.13.0-45-generic – указываем свою последнюю стабильную версию, посмотреть можно в каталоге /usr/src

- Затем выполняем команду:

    sudo update-grub

- Перезапускаем компьютер.
- После перезапуска мы откатились на предыдущую версию ядра и у нас снова доступно интернет-соединение.
- Теперь выполняем обновление системы:

    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get -f install

- После этого возвращаем предыдущую версию файла /etc/default/grub:

    sudo cp /etc/default/grub /etc/default/grub.prev.bak
    sudo cp /etc/default/grub.bak /etc/default/grub
    sudo update-grub

- Перезапускаем компьютер и снова выполняем обновление:

    sudo apt-get update
    sudo apt-get upgrade
