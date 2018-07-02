---
layout: post
title: "Аналитика таймеров"
permalink: analytics-of-timers
tags: timers quartz
---
## Почему Quartz, чем он лучше остальных
Среди существующих планировщиков задач (IBM WebSphere Scheduler, EJB Timer Service, Quartz Scheduler, Spring Task Scheduler) предпочтение отдается Quartz Scheduler, так как он обладает рядом полезных свойств:
-   Хранение состояния задачи в БД (Персистентность)    
-   Распределение нагрузки между серверами (Кластеризация)    
-   Возможность гибкого задания расписания (cron, интервал)    
-   Возможность просмотра статуса выполнения задач    
-   Возможность запуска задач в контексте транзакции    
-   Простота и легкость в использовании

**Ссылки:**
 - [java-quartz-scheduler-ejb-3-0-timer-service-and-java-timer-task-when-to-use-each](https://mhashem.wordpress.com/2010/03/29/java-quartz-scheduler-ejb-3-0-timer-service-and-java-timer-task-when-to-use-each/)
- [ejb-timer-performance](https://stackoverflow.com/questions/6843942/ejb-timer-performance)
- [job-scheduling-ejb-3-1-timers-or-quartz](https://stackoverflow.com/questions/8681557/job-scheduling-ejb-3-1-timers-or-quartz)
- [programmatic-timers-and-automatic-timers-difference](https://stackoverflow.com/questions/17421799/programmatic-timers-and-automatic-timers-difference)
- [ejb3-vs-spring-framework.jsp](http://www.visionsdeveloper.com/blog/page/ejb3-vs-spring-framework.jsp)
- [j-quartz](https://www.ibm.com/developerworks/ru/library/j-quartz/index.html)
- [java-ee-schedulers.html](https://www.javacodegeeks.com/2016/10/java-ee-schedulers.html)
## Архитектура Quartz Framework
Quartz Framework обладает простой для понимания архитектурой и оперирует тремя основными понятиями (классами):
-   **Шедулер** (планировщик) - управляет таймерами запуска задач. Связывает классы “Job” и “Trigger” вместе и выполняет их.    
-   **Джоб** - конкретная задача, запускаемая по таймеру.    
-   **Триггер** - условие выполнения задачи - задает временные рамки запуска задач и их выполнения. В Quattz два основных тригера:
-   **SimpleTrigger** – позволяет устанавливать время страта, время окончания и интервал повторения.    
-   **CronTrigger** – позволяет использовать Unix cron-выражения для указания даты и времени запуска вашей джобы.    

**Ссылки:**
- [Работа с шедулером в Java (Spring)](https://habrahabr.ru/post/193140/)
- [quartz-scheduler-tutorial](https://examples.javacodegeeks.com/enterprise-java/quartz/quartz-scheduler-tutorial/)
- [quartz-scheduler-tutorial](https://www.mkyong.com/tutorials/quartz-scheduler-tutorial/)
- [quartz-2-scheduler-tutorial](https://www.mkyong.com/java/quartz-2-scheduler-tutorial/)
- [java-quartz-configuration-example](https://examples.javacodegeeks.com/enterprise-java/quartz/java-quartz-configuration-example/)
- [quartz-2.x-configuration](http://www.quartz-scheduler.org/documentation/quartz-2.x/configuration/index)
## Персистентность
**Персистентность таймеров** – важное необходимое свойство, включающее в себя следующее:
-   Сервер приложений сохраняет события планирования, когда приложение не работает, чтобы не потерять их.
-   Планировщик может быть настроен на сохранение пропущенных событий и их выполнение, когда приложение снова будет работать.
-   Сервер приложений использует внутреннюю базу данных для хранения пропущенных событий.
 
Примечание: сервер приложений будет генерировать все пропущенные события при рестарте приложения.

Очередь событий настраивается по частоте и задержке.

Есть возможность не сохранять события, которые были потеряны, если приложение не запущено – в таком случае таймеры будут являться **неперсистентными**.

В неперсистентном случае, жизненный цикл планировщика совпадает с приложением: создается при старте приложения и уничтожается по завершению приложения. Напротив персистентный планировщик выживает при рестарте приложения. Он просто спит, когда приложение не работает.

Примеры поведения персистентного планировщика:
-   Если триггер должен сработать только несколько раз. То после рестарта приложения Scheduler должен знать об этом и не активировать его.    
-   Если у нас есть триггер, который должен срабатывать каждый час, но спустя, например, 59 минут наше приложение упало. То когда его перезапустили, этот триггер должен сработать через минуту, а не через час.    

Что выбрать: персистентный или неперсистентный планировщик?
-   Если функциональность, выполняемая по расписанию, критична для бизнеса и мы не можем себе позволить потерять событие, то персистентный планировщик - решение проблемы.    
-   В остальных случаях, неперсистентный планировщик легче (не использует БД) и проще в управлении (меньше трудностей при обновлении приложений, потому что не создается очередь планируемых событий при рестарте приложения, всегда создается новый планировщик при рестарте приложения).
    
Quartz предлагает два различных средства, с помощью которых можно хранить связанные с заданиями и триггерами данные в памяти или в базе данных:
-   Первое средство, экземпляр класса RAMJobStore, является настройкой по умолчанию. Это самое простое в использовании хранилище заданий, дающее к тому же наибольшую производительность, поскольку все данные хранятся в памяти. Главным недостатком этого метода является недостаточная сохраняемость данных. Поскольку все данные сохраняются в RAM, вся информация будет утеряна после "падения" приложения или системы.    
-   Чтобы исправить эти недостатки, Quartz предлагает JDBCJobStore. Как следует из названия, этот способ сохранения заданий помещает данные в базу данных через JDBC. Ценой более надёжного хранения данных является более низкая производительность, а также большая сложность.
    
Дистрибутив quartz содержит скрипты для генерации таблиц для соответствующих баз в docs/dbTables.

**Ссылки:**
- [spring-quartz-with-a-database](http://codrspace.com/Khovansa/spring-quartz-with-a-database/)
- [spring-quartz-jdbc-example](https://github.com/LukeDowell/spring-quartz-jdbc-example)
- [configuring-quartz-with-jdbcjobstore-in](http://www.nurkiewicz.com/2012/04/configuring-quartz-with-jdbcjobstore-in.html)
## Кластеризация
**_Проблема:_**
В кластере запущено более чем один экземпляр нашего приложения (на каждый узел кластера) и все экземпляры имеют свои копии шедулеров. Но мы нуждаемся в одном шедулере, запущенном среди всех узлов кластера, в противном случае мы получаем множество копий одного и того же события.
***Решение проблемы:***
Каждый сервер приложений имеет свой вариант для решения проблемы "множества экземпляров шедулеров".

Но общее решение требует сохраняемости (персистентности) шедулера при использовании в кластере. Т.к. нам нужно какое-то непереходное глобальное хранилище, чтобы отслеживать, какие задания выполнялись, чтобы они выполнялись точно на одной машине. Реляционная база данных как разделяемая память хорошо работает в этом сценарии.

Кластеризация в quartz позволяет решить две основные задачи:
1.  Отказоустойчивость
2.  Балансировка нагрузки   

Кластеризация в quartz в настоящее время работает только с JDBC-Jobstore (JobStoreTX или JobStoreCMT), и основывается на том, что каждый узел в кластере работает с одной и той же базой.

**Балансировка нагрузки** происходит автоматически, на каждом узле кластера запускаются джобы так быстро как это возможно. Когда наступает время запуска триггера, первый узел, который овладевает им (размещает блокировку на нем), становится узлом, который запустит его. Только один узел запускает джобу для каждого старта. Т.е. если джоба имеет повторяющийся триггер, который говорит запускаться каждые 10 секунд, то только один узел стартует джобу в 12:00:00, затем в 12:00:10 и т.д. И необязательно, что это будет один и тот же узел каждый раз. Какой именно узел запускает джобу – определяется случайно. Механизм балансировки нагрузки является почти случайным для занятых шедулеров (у которых много триггеров), но поддерживает один и тот же узел для незанятых (несколько триггеров) планировщиков.

**Отказоустойчивость** случается, когда один узел падает в процессе выполнения одного или нескольких джоб. Когда узел падает, другой узел обнаруживает это состояние и определяет джобы в базе данных, которые были в процессе выполнения внутри упавшего узла. Каждая джоба, помеченная для восстановления (с значением свойства “requests recovery” в JobDetail), будет перевыполнена на оставшихся узлах. Джоба, непомеченная для восстановления, будет освобождена от выполнения в следующий раз при старте связанного триггера.

Кластеризация лучше всего подходит для масштабируемых долго-выполняемых и ресурсо-затратных джоб (распределяя загрузку работы среди множества узлов). Если вам необходимо горизонтально масштабировать поддержку тысячи часто запускаемых (1с) джоб, рассмотрите разделение джоб, используя множество различных планировщиков. Планировщик вынужден использовать кластерную блокировку. Паттерн который ухудшает производительность, когда вы добавляете больше узлов (свыше трех узлов – зависит от возможностей вашей базы данных).

Кластеризация включается установкой свойства **«org.quartz.jobStore.isClustered»** в значение **true**. Каждый экземпляр в кластере должен использовать одну и ту же копию файла **quartz.properties**. Исключением из этого является использование файлов, которые являются идентичными, со следующими допустимыми исключениями: различный размер пула потоков и различное значение для свойства **org.quartz.scheduler.instanceId**. Каждый узел в кластере должен иметь уникальный **instanceId**, который настраивается установкой значения «AUTO» в его свойство.

Никогда не запускайте кластеризацию на отдельных машинах, если их системы не синхронизируются регулярно по времени (часы должны быть в пределах секунды друг от друга).

Никогда не стартуйте некластеризованный экземпляр с тем же набором таблиц базы данных, что и любой другой экземпляр. Вы можете получить серьезное повреждение данных и неустойчивое поведение.

Пример properties-файла для кластеризованного планировщика:

    # Configure Main Scheduler Properties
    org.quartz.scheduler.instanceName = MyClusteredScheduler
    org.quartz.scheduler.instanceId = AUTO
    # Configure ThreadPool
    org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
    org.quartz.threadPool.threadCount = 25
    org.quartz.threadPool.threadPriority = 5
    # Configure JobStore
    org.quartz.jobStore.misfireThreshold = 60000
    org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
    org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.oracle.OracleDelegate
    org.quartz.jobStore.useProperties = false
    org.quartz.jobStore.dataSource = myDS
    org.quartz.jobStore.tablePrefix = QRTZ_
    org.quartz.jobStore.isClustered = true
    org.quartz.jobStore.clusterCheckinInterval = 20000
    # Configure Datasources
    org.quartz.dataSource.myDS.driver = oracle.jdbc.driver.OracleDriver
    org.quartz.dataSource.myDS.URL = jdbc:oracle:thin:@polarbear:1521:dev
    org.quartz.dataSource.myDS.user = quartz
    org.quartz.dataSource.myDS.password = quartz
    org.quartz.dataSource.myDS.maxConnections = 5
    org.quartz.dataSource.myDS.validationQuery=select 0 from dual
    
**Ссылки**
- [how-to-quartz-scheduler-with-clustering](http://blog.codeleak.pl/2014/05/how-to-quartz-scheduler-with-clustering.html)
- [configuring-quartz-scheduler-to-run-in-clustered-environment](https://myshittycode.com/2013/09/26/configuring-quartz-scheduler-to-run-in-clustered-environment/)
- [ConfigJDBCJobStoreClustering](http://www.quartz-scheduler.org/documentation/quartz-2.2.x/configuration/ConfigJDBCJobStoreClustering)
## Возможность управления таймерами
***Проблема:***
Как управлять шедулером, не останавливая приложение: убирать из списка выполнения задачи или же менять cron-таймер и т.д.

Можно выделить два типа изменений:
1. Изменения не вовлекают джобы или удаление триггеров. Например, изменение крон-выражения для триггера, изменение параметров джобы, или java-класса для джобы.
2. Изменения, вовлекающие джобу или удаление триггера. Переименование джобы или триггера попадает в эту категорию.

Spring-овый 'SchedulerFactoryBean' поддерживает изменения первого типа: его свойство overwriteExistingJobs определяет, следует ли обновлять определения заданий и триггеров, хранящихся в базе данных, при запуске веб-приложения. По умолчанию используется значение false.

Изменения второго типа:
Если данное свойство установить в true, то старые джобы и триггеры не будут удалены, пока они не упоминаются в новой версии вашего приложения.

Возможны два способа решения проблемы:
1. Запустите сценарий базы данных перед установкой своего веб-приложения, которое уничтожит данные из таблиц (либо просто заданий и триггеров, которые будут удалены, либо все данные). Скрипт должен быть применен, когда приложения остановлены.
2. Реализуйте подкласс Spring SchedulerFactoryBean, который даст возможность очистить таблицы Quartz от Java при запуске веб-приложения

В Quartz у объекта шедулера можно вызывать следующие методы:
-   pauseJob(String name, String Group) — остановить выполнение задачи шедулера в указанной группе джобов. Остановка происходит путём остановки соответствующего триггера (см. pauseTrigger) 
-   resumeJob(String name, String Group) — возобновить выполнение задачи шедулера в указанной группе джобов. Восстановление происходит путём запуска соответствующего триггера (см. resumeTrigger)  
-   pauseTrigger(String name, String Group) — останавливает триггер в соответствующей группе    
-   resumeTrigger(String name, String Group) — возобновляет работу триггера в соответсвующей группе    
-   pauseAll — останавливает все задачи шедулера (pauseJobs(String group) — только у конкретной группы)    
-   resumeAll — возобновляет запуск всех задач шедулера (см. также resumeJobs)  
## Интеграция Quartz + Spring
Spring обеспечивает поддержку классов Quartz и предоставляет следующие классы:
-   QuartzJobBean – простая реализация интерфейса org.quartz.Job, в которой необходимо реализовать метод executeInternal, в котором определяется действие для выполнения.    
-   JobDetailFactoryBean – фабрика бинов для создания org.quartz.JobDetail. Вы можете сконфигурировать "job class" и "job data" используя bean-style.    
-   SimpleTriggerFactoryBean – фабрика бинов для создания org.quartz.SimpleTrigger    
-   CronTriggerFactoryBean – фабрика бинов для создания org.quartz.CronTrigger    
-   SchedulerFactoryBean – фабрика бинов для создания org.quartz.Scheduler    

**Ссылки:**
- [spring-quartz-scheduler-example](https://www.mkyong.com/spring/spring-quartz-scheduler-example/)
- [spring-quartz-schedule](http://www.baeldung.com/spring-quartz-schedule)
- [spring-4-quartz-scheduler-integration-example](http://websystique.com/spring/spring-4-quartz-scheduler-integration-example/)
- [quartz/spring-quartz-scheduler-example](https://examples.javacodegeeks.com/enterprise-java/quartz/spring-quartz-scheduler-example/)
## Интеграция Quartz + Spring + Wepsphere
Используя Quatz с WebSphere Application Server можно столкнуться с проблемами доступа к jndi-ресурсам или попытки участия в JTA-транзакции. Вы получаете эксепшн со следующим содержимым:

Exceptionjavax.naming.ConfigurationException: A JNDI operation on a “java:” name cannot be completed because the server runtime is not able to associate the operation’s thread with any J2EE application component. This condition can occur when the JNDI client using the “java:” name is not executed on the thread of a server application request. Make sure that a J2EE application does not execute JNDI operations on “java:” names within static code blocks or in threads created by that J2EE application. Such code does not necessarily run on the thread of a server application request and therefore is not supported by JNDI operations on “java:” names. [Root exception is javax.naming.NameNotFoundException: Name comp/env/[…] not found in context “java:”.]

Такие пакеты как quartz и jdk таймеры стартуют неуправляемые потоки. Неуправляемые потоки не имеют доступа к информации контекста JEE.

Для того, чтобы сделать Quartz полностью поддерживаемым в WebSphere необходимо быть уверенным, что неуправляемые потоки не стартуются. Тогда ваши джобы получают доступ к JNDI ресурсам и могут участвовать в JTA транзакциях (которые требуют поиска jndi java:comp/UserTransaction).

Это возможно благодаря указанию пользовательского thread pool (пула потоков), который делегирует CommonJ управление по созданию потоков. Однако, Quartz стартует несколько внутренних потоков, из которых стартуются актуальные джобы. Если внутренние quartz-потоки являются неуправляемыми, JEE контекст теряется и не может быть распространён на потоки джобов, даже если они являются управляемыми.

Две проблемы QTZ-113 «Quartz стартует неуправляемые потоки и J2EE контекст не распространяется на потоки джобов» и QTZ-194 «Quartz стартует неуправляемые потоки в JobStoreSupport» были исправлены в версии Quartz 2.1, который представляет новый интерфейс **org.quartz.spi.ThreadExecutor**.

Для того, чтобы внутренние quartz-потки были управляемыми необходимо установить два свойства в вашей конфигурации quartz:

    org.quartz.threadExecutor.class=org.quartz.commonj.WorkManagerThreadExecutor
    org.quartz.threadExecutor.workManagerName=wm/default

Spring предлагает поддержку Quartz используя SchedulerFactoryBean:

    <bean id="quartzScheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
      <property name="taskExecutor" ref="taskExecutor" />
      <property name="quartzProperties"> 
      <props>  
        <prop key="org.quartz.jobStore.class">org.quartz.simpl.RAMJobStore</prop>
        <prop key="org.quartz.threadExecutor.class">org.quartz.commonj.WorkManagerThreadExecutor</prop>  
        <prop key="org.quartz.threadExecutor.workManagerName">wm/default</prop> 
      </props>  
      </property>  
    </bean>

Вы можете установить значение **task executor** в **org.springframework.scheduling.commonj.WorkManagerTaskExecutor**. Этот класс делегирует полномочия CommonJ Work Manager. Который доступен внутри WebSphere. Так потоки, которые будут созданы для ваших quartz-джобов, оборачиваются в commonj Work объекты, и будут управляться сервером приложений:

    <bean id="taskExecutor"  class="org.springframework.scheduling.commonj.WorkManagerTaskExecutor">
      <property name="resourceRef" value="true" />  
      <property name="workManagerName" value="wm/default" />
    </bean>

WorkManager в WorkManagerThreadPool инициализируется до старта планировщика из потока Java EE (таких как инициализация сервлета). WorkManagerThreadPool может затем создать поток-демон, который будет обрабатывать все планируемые задания по созданию и планированию новых Work-объектов. В этом случае планировщик (в своем собственном потоке) передает задачи потоку управления (Work-демону).

Т.е. всего два места где необходимо сконфигурировать quartz с commonj. Первое spring-конфиг, taskExecutor в SchedulerFactoryBean, второе определяется в property-файле в свойстве org.quartz.threadExecutor.class.

Настройка в spring позваляет quartz использовать пул потоков websphere для работы собственного пула потоков, настройка в property-файле используется для потока шедулера.

**Ссылки**
 - [Using Spring and Hibernate with WebSphere Application Server](https://www.ibm.com/developerworks/websphere/techjournal/0609_alcott/0609_alcott.html)
- [quartz-websphere-spring](https://gybas.com/2014/quartz-websphere-spring/)
- [QTZ-113](https://jira.terracotta.org/jira/browse/QTZ-113)
- [QTZ-194](https://jira.terracotta.org/jira/browse/QTZ-194)
- [quartz-websphere-spring](https://github.com/sgybas/quartz-websphere-spring)
- [unmanaged-threads-spring-quartz-websphere-hibernate](https://stackoverflow.com/questions/175880/unmanaged-threads-spring-quartz-websphere-hibernate)
## Примеры
[timer-sample](https://github.com/OlgaFedorova/timer-sample)
