Install
---------------------------------------------

 metatron integrator 는 hadoop 내에 데이터를 수집,처리할 수 있는 tool 로써 oozie 엔진을 기반으로 동작한다.

 oozie 는 hadoop component(MR, Hive, Spark, Exec, ...) 를 사용하기 때문에 hadoop component 들의 설치가 우선적으로 문제없이 동작하는 환경을 구성하여야 한다. 이를 바탕으로 oozie 가 설치되고 integrator 가 설치되어야 한다.



1. Requirements
=====================================================
    * CentOS 6.x 이상 권장
    * Java 1.8 이상
    * Hadoop cluster 2.6.x 이상 권장
    * mysql (mariadb) 5.5 이상 권장
    * oozie 4.3.0 이상
    * activemq 5.15.6 이상

2. Install oozie
=====================================================
 oozie 는 4.3.0 버전 이상을 설치하여야 하며, 설치와 실행에 대한 사항은 오피셜 사이트(http://oozie.apache.org/)를 참조.

 oozie server configuration 에는 timezone 과 proxy user 설정등이 추가되어 있어야 한다.

2-1. oozie conf 내 proxy user 추가
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
아래와 같이 integrator application 이 실행되는 계정(metatron)에 대해서 proxy user 를 추가한다.

    .. code-block:: xml
       :linenos:

        <property>
            <name>oozie.service.ProxyUserService.proxyuser.metatron.hosts</name>
            <value>*</value>
        </property>
        <property>
            <name>oozie.service.ProxyUserService.proxyuser.metatron.groups</name>
            <value>*</value>
        </property>

2-2. oozie conf 내 JPA 설정
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
oozie meta data 가 저장되는 DBMS 에 대한 설정을 정의한다.

    .. code-block:: xml
       :linenos:

        <property>
            <name>oozie.service.JPAService.jdbc.driver</name>
            <value>com.mysql.jdbc.Driver</value>
        </property>
        <property>
            <name>oozie.service.JPAService.jdbc.url</name>
            <value>jdbc:mysql://{DBMS Server IP}:{DBMS Server Port}/oozie</value>
        </property>
        <property>
            <name>oozie.service.JPAService.jdbc.username</name>
            <value>oozie_user</value>
        </property>

        <property>
            <name>oozie.service.JPAService.jdbc.password</name>
            <value>oozie_userpassword</value>
        </property>

2-3. oozie server timezone 설정
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 서버가 동작하는 지역 timezone 에 맞게 timezone 설정도 추가 되어야 한다.

    .. code-block:: xml
       :linenos:

        <property>
            <name>oozie.processing.timezone</name>
            <value>GMT+0900</value>
        </property>

2-4. oozie conf 내 JMS 설정
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
job 의 시작과 종료등 이벤트를 integrator 가 수신할 수 있도록 JMS 설정을 추가하여야 한다. oozie external library 중에 JMS 관련한 class 를 oozie.service.ext 에 등록하여야 하고 oozie.jms.producer.connection.properties 에는 JMS publish 할 수 있도록 설정을 추가하여야 한다.

설정 중에 {activemq server IP} 는 activemq 가 설치되어 있는 서버 IP 로 대체되어야 한다.

    .. code-block:: xml
       :linenos:

        <property>
            <name>oozie.services.ext</name>
                <value>
                 org.apache.oozie.service.JMSAccessorService,
                 org.apache.oozie.service.JMSTopicService,
                 org.apache.oozie.service.EventHandlerService,
                 org.apache.oozie.sla.service.SLAService
                </value>
        </property>
        <property>
            <name>oozie.jms.producer.connection.properties</name>
            <value>java.naming.factory.initial#org.apache.activemq.jndi.ActiveMQInitialContextFactory;java.naming.provider.url#tcp://{activemq server IP}:61616;connectionFactoryNames#ConnectionFactory</value>
        </property>

2-5. oozie port open
oozie와 integrator 는 rest API 를 통해서 통신하기 때문에 integrator → oozie direction 방향으로 oozie listen port(11000)에 대해 방화벽 오픈이 필요하다.

3. Install Integrator
=====================================================
Integrator application 의 설치 경로는 기본적으로 /home/metatron/servers/metatron-integrator 를 사용하며, 변경이 필요한 경우 integrator.sh 파일내에 경로를 수정하여야 한다.

3-1. oozie listen port open
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
oozie와 integrator 는 rest API 를 통해서 통신하기 때문에 integrator → oozie direction 방향으로 oozie listen port(11000)에 대해 방화벽 오픈이 필요하다.

3-2. hadoop cluster 내 webhdfs port open
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
integrator 는 oozie xml 파일 혹은 hql, library 파일등을 업로드하기 위해서  web hdfs 포트가 열려있어야 한다. 기본 포트인 50070 포트에 대해 integrator 가 접근할 수 있도록 방화벽 설정이 필요하다.

3-3. application.yaml 설정
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
integrator 의 설정은 yaml 파일로 정의한다. yaml 파일은 section 으로 구성되어 있으며 각 section 의 역할은 아래와 같다.

section name	설명	비고
spring.datasource	integrator meta 정보를 저장하는 DB 서버 정보를 입력한다.
spring.jpa	integrator meta 정보를 저장할 DB 의 정보를 입력한다.
spring.mail	Job noti, SLA noti 를 받을 mail 서버 정보를 입력한다.
spring.activemq	oozie 서버와 JMS 연동을 위한 activemq 서버 정보를 입력한다.
spring.hadoop	hadoop cluster 정보를 입력한다.
spring.oozie	oozie 서버 정보를 입력한다.
spring.hive	hive server 정보를 입력한다.
spring.druid	druid task 를 위한 discovery 설정 정보를 입력한다.
spring.polaris	druid task 를 위한 discovery 설정 정보를 입력한다.
notification.slack	Job noti, SLA noti 를 받을 slack 설정 정보를 입력한다.
notification.sms	Job noti, SLA noti 를 SMS 로 발송할 수 있도록 sms 서버 정보를 입력한다.
server.port	integrator 서버 listen port 를 설정한다.
zuul.route	metatron discovery 인증과 관련된 설정으로 discovery 서버 정보를 입력한다.
위와 같은 section 별로 설정된 yaml 파일의 sample 이다.

    .. code-block:: jproperties
       :linenos:

        spring:
            profiles: prod
            devtools:
                restart:
                    enabled: false
                livereload:
                    enabled: false
            datasource:                         # integrator metadata DBMS 서버 정보
                type: com.zaxxer.hikari.HikariDataSource
                url: jdbc:mysql://{dbserver IP}:{dbserver port}/integrator?useUnicode=true&characterEncoding=utf8&useSSL=false           # db server 정보에 맞춰서 수정
                username: integrator                                                                                                     # db server 정보에 맞춰서 수정
                password: xxxxxxxxxx                                                                                                     # db server 정보에 맞춰서 수정
                hikari:
                    data-source-properties:
                        cachePrepStmts: true
                        prepStmtCacheSize: 250
                        prepStmtCacheSqlLimit: 2048
                        useServerPrepStmts: true
            jpa:
                database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
                database: MYSQL
                show-sql: false
                hibernate:
                  naming-strategy: org.hibernate.cfg.EJB3NamingStrategy
                  naming:
                    physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
                properties:
                    hibernate.id.new_generator_mappings: true
                    hibernate.cache.use_second_level_cache: false
                    hibernate.cache.use_query_cache: false
                    hibernate.generate_statistics: false
            mail:                               # Job event , SLA event 발생시 메일 발송을 위한 SMTP 서버 정보
                host: localhost
                port: 25
                username:
                password:
                sendAsMail: "Oozie Notification <no_reply@sk.com>"
            thymeleaf:
                cache: true
            http:                               # web upload file temporary saving path
              multipart:
                location: /home/metatron/servers/metatron-integrator/upload_temp
                max-file-size: 1000Mb
                max-request-size: 1000Mb
            activemq:                           # JMS 수신을 위한 activemq 서버 정보
              broker-url: tcp://{activemq server IP}:61615
              topic: metatron
              user: metatron
              password:
            jms:
              pub-sub-domain: true
            hadoop :                            # hadoop namenode 정보
              url : http://hadoop-namenode
              user.name : metatron
              nameNodePort : 50070
              dataNodePort : 50075
              dataNodeUrlUsed : true
              defaultPath : /webhdfs/v1
              tunneling : false
            oozie :                             # oozie server 정보
              oozieUrl : http://{oozie server IP}                                       # oozie server ip ex) http://1.2.3.4
              ooziePort : 11000                                                         # oozie server port. default 11000

              timezone.offset : T+0900
              timezone.location : Asia/Seoul

              hdfs.workflowSavingPath : /user/metatron/integrator/workflow              # workflow saving HDFS path
              hdfs.coordinatorSavingPath : /user/metatron/integrator/coordinator        # coordinator saving path
              hdfs.taskSavingPath : /user/metatron/integrator/task                      # task library temporary saving path
              hdfs.defaultPath : /user/metatron                                         # base path

              jobConf.nameNode : hdfs://{HDFS name service}                             # workflow namenode variable. ex) hdfs://metatron-cluster
              jobConf.jobTracker : {Yarn resourcemanager address}                       # workflow job tracker(YARN) server variable ex) metatron-hadoop-01:8032
              jobConf.oozie.use.system.libpath : true                                   # oozie.use.system.libpath variable
              jobConf.oozie.libpath : /user/oozie/share/lib                             # oozie.libpath variable (HDFS Path)
              jobConf.impersonate.mode : true                                           # Is user.name equals application runner
              jobConf.impersonate.name : metatron                                       # user.name variable
              jobConf.defaultQueue.name : default                                       # workflow queue name variables

            druid:                             # druid task 정보
              userid: admin                                                             # discovery oauth token 획득을 위한 admin 계정 id
              password: admin															# discovery oauth token 획득을 위한 admin 계정 password
              discoveryUrl: http://{discovery server IP}:{discovery port}               # discovery oauth token 획득을 위한 discovery 서버 정보
              authorization: Basic cG9sYXJpc19jbGllbnQ6cG9sYXJpcw==                     # discovery oauth token 획득을 위한 request header 내 Authorization 값
              jarPath: /user/metatron/integrator/ext_scripts/metatron-integrator-druid.jar       # druid task(metatron-integrator-druid.jar)의 HDFS 상 경로

        server:
            port: 8280                    # integrator server listen port(service port)
            compression:
                enabled: true
                mime-types: text/html,text/xml,text/plain,text/css, application/javascript, application/json
                min-response-size: 1024

        liquibase:
            contexts: prod

        notification:                         # JMS 를 통해 수신한 Job event , SLA event 를 전달하기 위한 messaging server 정보
          slack:                              # Slack 정보
            webhookUrl:                       # Setup & Configure Incoming-Webhook URL in Slack > Apps
            botName:                          # (Optional) if empty, use default value that configured in Slack App
            botEmoji:                         # (Optional) if empty, use default value that configured in Slack App
            notiChannel:                      # (Optional) if empty, use default value that configured in Slack App
          sms:                                # SMS 서버 정보
            requestUrl: "http://{sms server IP}:{sms server port}/esc-channel/etc/sms/sendsmsinfo"
            senderNumber: "01000000000"        # The message sender's number
            receiverNumber: "010000000"

        security:                            # discovery 통합 로그인을 위한 인증서버 정보(discovery 서버 정보)
          basic:
            enabled: false
          oauth2:
            # OAuth 서버 Client 정보
            client:
              client-id: polaris_client
              client-secret: polaris
            resource:                        # oauth url
              token-info-uri: http://{metatron discovery server IP}:{metatron discovery server port}/oauth/check_token

        logging:
            level:
                ROOT: INFO
                com.sample: INFO
                io.github.jhipster: INFO

        jhipster:                   # jhipster 서버 정보(설정 불필요)
            http:
                version: V_1_1 # To use HTTP/2 you will need SSL support (see above the "server.ssl" configuration)
                cache: # Used by the CachingHttpHeadersFilter
                    timeToLiveInDays: 1461
            mail: # specific JHipster mail property, for standard properties see MailProperties
                from: sample@localhost
                base-url: http://my-server-url-to-change # Modify according to your server's URL
            metrics: # DropWizard Metrics configuration, used by MetricsConfiguration
                jmx.enabled: true
                graphite:
                    enabled: false
                    host: localhost
                    port: 2003
                    prefix: sample
                prometheus:
                    enabled: false
                    endpoint: /prometheusMetrics
                logs: # Reports Dropwizard metrics in the logs
                    enabled: false
                    report-frequency: 60 # in seconds
            logging:
                logstash: # Forward logs to logstash over a socket, used by LoggingConfiguration
                    enabled: false
                    host: localhost
                    port: 5000
                    queue-size: 512

        zuul:                          # zuul 서버 정보
          routes:
            oauth:
              path: /oauth/**
              url: http://{metatron discovery server IP}:{metatron discovery server port}              # require metatron discovery server IP, port
              sensitiveHeaders: Cookie,Set-Cookie
              stripPrefix: false
            stomp:
              path: /stomp/**
              url: http://{metatron discovery server IP}:{metatron discovery server port}              # require metatron discovery server IP, port
              sensitiveHeaders: Cookie,Set-Cookie
              strip-prefix: true
            api:
              path: /api/**
              url: http://{metatron discovery server IP}:{metatron discovery server port}              # require metatron discovery server IP, port
              sensitiveHeaders: Cookie,Set-Cookie
              stripPrefix: false

        application:


3-3. Make application directory in HDFS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
yaml 파일에서 아래와 같이 application 에서 사용하는 HDFS 상의 디렉토리를 정의하게 된다.

    .. code-block:: jproperties
       :linenos:

        hdfs.workflowSavingPath : /user/metatron/integrator/workflow
        hdfs.coordinatorSavingPath : /user/metatron/integrator/coordinator
        hdfs.taskSavingPath : /user/metatron/integrator/task
        hdfs.defaultPath : /user/metatron

yaml 파일에서 설정한 workflow, coordinator, task 경로에 맞춰 HDFS 내에  directory 를 생성하여야 한다. 위에서 설정한 경로를 hadoop cluster 내에 hadoop fs -mkdir 명령어로 생성해 주면 된다.

3-4. integrator library upload to HDFS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
integrator 에서 사용하는 library 는 HDFS 에 업로드되어야 한다. ext_scripts 는 /user/metatron/integrator/ext_scripts 로 업로드한다.

3-5. application run script(integrator.sh) 확인
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
application 실행 스크립트인 integrator.sh 파일내에 application.yaml 의 profile 을 설정하여야 한다. 일반적인 경우 prod 라는 profile 을 사용하지만 yaml 파일 내에 복수개의 profile 을 정의 하는 경우는 profile name 을 지정해 주어야 한다.

spring.profiles.active 항목에 application.yaml 에 설정한 profile 명을 넣어서 yaml 파일 profile 을 불러올 수 있도록 설정하여야 한다.



또한, application 의 설치 경로(/home/metatron/servers/metatron-integrator)를 변경하는 경우에도 shell script 내 base dir 정보를 변경해 주어야 한다.

3-6. application start
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
integrator.sh start / stop 명령어를 통해서 application 을 실행하거나 종료할 수 있다.