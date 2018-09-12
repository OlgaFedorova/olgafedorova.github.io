---
layout: post
title: "Intellij idea: запуск maven-проекта на основе Spring boot с исключением resources"
permalink: intellij-idea-with-maven-and-spring-boot-exclude-resources
tags: java spring spring-boot maven intellij-idea
---
**_Проблема:_** есть **maven-проект**, в **pom-файле** которого указано:

    <build>
        <resources>
            <resource>
                <directory>${basedir}/src/main/resources</directory>
                <excludes>
                    <exclude>*.*</exclude>
                </excludes>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        <plugins>
    <build>
Т.е. используем плагин **spring-boot-maven-plugin**, и исключаем из проекта папку **${basedir}/src/main/resources**.
Если у нас есть файл **application.properties** в котором указаны свойства для инициализации компонент, приложение из idea не запуститься. Мы получим ошибки о том, что нам не удалось инициализировать некоторые из бинов.
**Решение:**
В свойствах меню **Run => Edit configuration** выберем нашу конфигурацию запуска и в **VM options** укажем полный путь к каталогу где хранится файл **application.properties**:
	
	-Dspring.config.location=path

![idea]({{ site.url }}/assets/2018-09-12-intellij-idea-with-maven-and-spring-boot-exclude-resources/1.png)
