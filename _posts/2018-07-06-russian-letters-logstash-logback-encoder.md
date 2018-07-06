---
layout: post
title: "Logback: 'Русские буквы' logstash-logback-encoder"
permalink: russian-letters-logstash-logback-encoder
tags: java logback spring spring-boot spring-cloud
---

**Проблема:** русские буквы выводятся в таком виде ": 

\u0438\u0434\u0435\u043D\u0442\u0438\u0444\u0438\u043A\u0430\u0442\u043E\u0440\u043E\u043C \u0440\u0435\u0441\u0443\u0440\u0441\u0430 - '/in' \u0443\u0441\u043F\u0435\u0448\u043D\u043E \u0437\u0430\u0432\u0435\u0440\u0448\u0438\u043B\u0441\u044F".

**Решение:**

По умолчанию доступна кодировка UTF-8, но jackson по умолчанию исключает все не ascii-символы. Поэтому необходимо переопределить поведение, создав класс, который имплементирует JsonFactoryDecorator и отключает эту опцию.

    import com.fasterxml.jackson.core.JsonGenerator;  
    import com.fasterxml.jackson.databind.MappingJsonFactory;  
    import net.logstash.logback.decorate.JsonFactoryDecorator;  
      
    public class NoEscapingJsonFactoryDecorator implements JsonFactoryDecorator {  
      
	    @Override  
	    public MappingJsonFactory decorate(MappingJsonFactory factory) {  
		    return (MappingJsonFactory) factory.disable(JsonGenerator.Feature.ESCAPE_NON_ASCII);  
	    }  
    }

Затем, конфигурируем encoder, используя созданный класс.

    <?xml version="1.0" encoding="UTF-8"?>  
	    <configuration>  
		    <include resource="org/springframework/boot/logging/logback/default.xml"/>  
		    <jmxConfigurator/>  
		    <springProperty name="springAppName" source="spring.application.name"/>  
		    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">  
			    <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">  
				    <jsonFactoryDecorator class="ru.sbt.parus.logstash.logback.decorate.NoEscapingJsonFactoryDecorator"/>  
			    ....  
			    </encoder>  
		    </appender>  
		      
		    <root level="info">  
			    <appender-ref ref="STDOUT"/>  
		    </root>  
    </configuration>
