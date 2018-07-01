---
layout: post
title: "Удаление индексов"
permalink: elasticsearch-curator-delete-indexes
tags: docker elasticsearch elasticsearch-curator
---
Чтобы избежать переполнения дискового пространства **elasticsearch** необходимо периодически очищать индексы.
**Требования:**
-   храним индексы за 1 день
-   удаляем индексы fluentd    
-   удаляем все служебные индексы, такие как: .monitoring-logstash-*, .monitoring-kibana-*, .monitoring-es-*
-   исключаем удаление индекса .kibana, т.к. в нем хранятся основные настройки kibana.

Файл **actions.yml** для elasticsearch-curator:

    ---
    actions:  
      1:  
        action: delete_indices  
        options:  
          ignore_empty_list: true  
          continue_if_exception: false  
          disable_action: false  
        filters:  
          - filtertype: age  
            source: creation_date  
            direction: older  
            unit: days  
            unit_count: 1  
          - filtertype: pattern  
            kind: regex  
            value: .kibana  
            exclude: True


