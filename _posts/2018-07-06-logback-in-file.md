---
layout: post
title: "Logback: Настройка вывода в файл"
permalink: logback-in-file
tags: java logback spring spring-boot spring-cloud
---
Для возможности вывода логов в файл создаем файловый **appender** в файле **logback-prod.xml**:

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">  
	    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
		    <fileNamePattern>${logDir}/${springAppName}_%d{yyyy-MM-dd}_%i.log</fileNamePattern>  
		    <maxHistory>${maxHistory}</maxHistory>  
		    <totalSizeCap>${totalSizeCap}</totalSizeCap>  
		      
		    <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">  
			    <maxFileSize>${maxFileSize}</maxFileSize>  
		    </timeBasedFileNamingAndTriggeringPolicy>  
	    </rollingPolicy>  
	      
	    <encoder>  
		    <charset>UTF-8</charset>  
		    <pattern>${filePattern}</pattern>  
	    </encoder>  
    </appender>

В настройках этого **appender** участвуют **springProperty**, которые указываются в **репозитории конфигурационных файлов**:

	<springProperty name="springAppName" source="spring.application.name"/>  
	<springProperty name="logDir" source="logging.logDir"/>  
	<springProperty name="totalSizeCap" source="logging.totalSizeCap"/>  
	<springProperty name="maxHistory" source="logging.maxHistory"/>  
	<springProperty name="maxFileSize" source="logging.maxFileSize"/>  
	<springProperty name="filePattern" source="logging.filePattern"/>

application.yml:

	#######################################################  
	# Конфигурирование логирования  
	######################################################  
	logging:  
	config: classpath:logback-prod.xml  
	# Каталог для хранения логов  
	logDir: D:/LOGS  
	# История в днях, максимальный объем логов распространяется на количество дней  
	maxHistory: 30  
	# Максимальный размер логов  
	totalSizeCap: 40KB  
	# Максимальный размер одного файла  
	maxFileSize: 15KB  
	# Шаблон выводимой информации  
	filePattern: "%d %-4relative [%thread] %-5level %logger{35} - %msg%n"

Настраиваемые для логирования свойства:

-   **logging.config** – имя и расположение конфигурационного файла logback.    
-   **logging.logDir** – абсолютный путь к каталогу, где размещаются конфигурационные файлы.    
-   **logging.maxHistory** – количество дней, в которых хранится история, при условии, что не достигнуты другие предельные величины для логов (общий размер файлов). Максимальный объем логов распространяется на заданное количество дней.    
-   **logging.totalSizeCap** – максимальный размер всех лог-файлов для сервиса.    
-   **logging.maxFileSize** – максимальный размер одного лог-файла.    
-   **logging.filePattern** - шаблон выводимой информации в логе.
    
Очищение старых логов происходит в момент, когда создается новый лог-файл, так у предыдущего превышен размер, заданный в параметре **maxFileSize**. В этот момент происходит проверка, что общий объем логов за число дней в **maxHistory** не превышает объем, указанный в **totalSizeCap**. Если условие проверки выполняется, очищаются лишние логи в пределах дней **maxHistory** до объема **totalSizeCap**. Поэтому, если мы хотим подчищать хвосты от логов, которые остались в периоды простоя сервсиов необходимо расширить количество дней в **maxFileSize**.

По предварительным настроенным параметрам хранение логов выглядит так:

![logs]({{ site.url }}/assets/2018-07-06-logback-in-file/logs.jpg)

[Пример проекта.](https://github.com/OlgaFedorova/spring-boot-logback)
