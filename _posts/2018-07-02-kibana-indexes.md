---
layout: post
title: "Индексы в Kibana"
permalink: kibana-indexes
tags: kibana
---
Для мониторинга соответствующего индекса в **Kibana** необходимо его настроить.
Заходим в пункт меню **Management => Create Index Pattern**:
![kibana-index]({{ site.url }}/assets/kibana-index_1.png)

В полe **«Index pattern»** указываем шаблон имени для отслеживаемого индекса. Если заданный индекс существует, то заполнится поле **«Time Filter field name».**
![kibana-index]({{ site.url }}/assets/kibana-index_2.png)

Для заданного имени будут автоматически сгенерированы поля.
![kibana-index]({{ site.url }}/assets/kibana-index_3.png)

Для мониторинга информации по заданному индексу необходимо перейти в пункт меню Discover, и в выпадающем списке выбрать необходимый индекс
![kibana-index]({{ site.url }}/assets/kibana-index_4.png)

![kibana-index]({{ site.url }}/assets/kibana-index_5.png)

![kibana-index]({{ site.url }}/assets/kibana-index_6.png)

Каждая запись представлена в виде таблицы с настроенными в индексе полями:
![kibana-index]({{ site.url }}/assets/kibana-index_7.png)
