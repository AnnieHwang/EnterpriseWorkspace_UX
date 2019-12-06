분산 환경 Install
---------------------------------------------
Step1. Configure directories in hdfs as below
=====================================================

    .. code-block:: console
       :linenos:

        hadoop fs -mkdir -p /druid/storage
        hadoop fs -mkdir -p /druid/logs

Step2. Metatron Engine(Druid) 다운로드
=====================================================
Download and Unzip the binary file. This Link – for Hadoop 2.7.3

    .. code-block:: console
       :linenos:

        Example location>
        ./druid_dist/druid-0.9.1-SNAPSHOT.{discovery.version}-hadoop-2.7.3

Step3. Download deploy template shell & Unzip
=====================================================
Download and Unzip the binary file.  This Link

    .. code-block:: console
       :linenos:

        Example Location>
        ./druid_dist/druid_dist_v2

Step4. Ansible Host 설정
=====================================================
    * druid_dist 내 hosts 내 druid 관련 설정을 참고하여 설치할 서버의 정보를 기입합니다.  (Ansible 서버에서 Druid cluster 에 자동 접속되는 환경을 구축해야한다. 또는 샘플에 위치한 vars 옵션을 사용하여 ssh 접속정보를 기입합니다.)
    * root 또는 sudo 권한이 있다면, /etc/ansible/hosts 내 설정하거나, INVENTORY 환경 변수를 지정하는 방법으로 설정합니다. (참고 : https://www.lesstif.com/pages/viewpage.action?pageId=22052879)

Step5. Modify deploy environment shell : Druid배포를 위한 설정 정보 변경 (env.sh)
=============================================================================================
    * 사전 계정 작업 : 배포를 위한 user hadoop group hadoop 계정 설정 설치 서버에서 각 서버간 ssh접속은 password없이 이루어져야 한다
    * /home/hadoop/servers/druid_dist : 설치 서버의 배포 디렉토리
    * /home/hadoop/servers/druid : 실제 클러스터에 copy되는 디렉토리
    * /dataXXX/druid/segment-cache : datanode위에 historical node가 올라가며 segment cache영역을 disk array 개수만큼 선언함


./druid_dist/druid_dist_v2/env.sh
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: jproperties
       :linenos:

        #!/bin/sh
        TARGET_SYSTEM="metatron-hadoop01"               #System Name
        BECOME_USER="metatron"                          #User for Druid excution
        BECOME_GROUP="metatron"                         #User Group for Druid excution
        BECOME_USER_GROUP="$BECOME_USER:$BECOME_GROUP"
        TARGET_DIR="/home/metatron/servers/druid"       #Target deploy directory
        DIST_DIR="/home/metatron/servers/druid_dist"    #Distribution directory
        ROOT_TEMP_DIR="/home/metatron/data1/druid"      #Temp directory
        TMP_DIR="$ROOT_TEMP_DIR/tmp"
        VAR_DIR="$ROOT_TEMP_DIR/var"
        SEGMENET_CACHE_DIR=("/home/metatron/data1/druid/segment-cache" "/home/metatron/data2/druid/segment-cache" "/home/metatron/data3/druid/segment-cache" "/home/metatron/data4/druid/segment-cache" "/home/metatron/data5/druid/segment-cache" "/home/metatron/data6/druid/segment-cache" "/home/metatron/data7/druid/segment-cache" "/home/metatron/data8/druid/segment-cache" "/home/metatron/data9/druid/segment-cache" "/home/metatron/data10/druid/segment-cache") #Local cache disk directory


Step6.Create necessary directory by shell - 필요 디렉토리 생성
=============================================================================================
./druid_dist/druid_dist_v2/setup_druid.sh

Step7.Configuring Druid’s property file - Druid Config 주요 설정 변경
=============================================================================================
The major configuration has already been put in the "bootstrap" directory. Below is a list of things you need to change or update depending on your environment

    * druid cluster 구성을 위한 meta system정보를 기입한다.
    * hdfs내에 디렉토리를 다음과 같이 구성한다
    * hadoop fs -mkdir -p /druid/storage
    * hadoop fs -mkdir -p /druid/logs

./druid_dist/druid_dist_v2/druid_bootstrap/conf/druid/_common/common.runtime.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: jproperties
       :linenos:

        # Zookeeper – zookeeper cluster server list
        druid.zk.service.host={zkhost01}:2181,{zkhost02}:2181,{zkhost03}:2181
        druid.zk.paths.base=/druid
        # Metadata Storage (MySQL or Maria DB connection info.)
        druid.metadata.storage.type=mysql
        druid.metadata.storage.connector.connectURI=jdbc:mysql://{mysql hostname}:{mysql port}/druid?useUnicode=true&characterEncoding=UTF-8
        druid.metadata.storage.connector.user=druid
        druid.metadata.storage.connector.password=druid
        # Deep storage
        # For HDFS:
        druid.storage.type=hdfs
        druid.storage.storageDirectory=/druid/storage
        # Indexing service logs
        # For HDFS:
        druid.indexer.logs.type=hdfs
        druid.indexer.logs.directory=hdfs://{clustername}/druid/logs
        # Query Cache, 50MB
        druid.cache.type=local
        druid.cache.sizeInBytes=52428800
        # Service discovery
        # Indexing service discovery. Update this if you change your overlord's "druid.service".
        druid.selectors.indexing.serviceName=druid/prod/overlord
        druid.selectors.coordinator.serviceName=druid/prod/coordinator


    * druid.extensions.loadList의 경우 extention중 필요로 하는 extension을 선택적으로 로딩하는 방식이다. 필수로 "mysql-metadata-storage", "druid-hdfs-storage"는 포함되어야 한다.
    * druid.query.groupBy.maxResults → groupBy Query의 최대 결과값을 지정한다. cardinality가 크면 메모리에 부담이 많이 가므로 주의해야 한다.
    * serviceName 의 경우 zookeeper의 service name이다. common에서는 overlord, coordinator 정보만 기재한다. 각 컴포넌트의 serviceName과 매핑되어야 한다.


./druid_dist/druid_dist_v2/druid_bootstrap/conf/druid/coordinator/runtime.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: bash
       :linenos:

        # Default host, port, service name.
        druid.service=druid/prod/coordinator
        druid.port=8081

./druid_dist/druid_dist_v2/druid_bootstrap/conf/druid/overlord/runtime.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: jproperties
       :linenos:

        # Default host, port, service name.
        druid.service=druid/prod/overlord
        druid.port=8090

./druid_dist/druid_dist_v2/druid_bootstrap/conf/druid/broker/runtime.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: jproperties
       :linenos:

        druid.service=druid/prod/broker
        druid.port=8082

        # HTTP server threads
        druid.broker.http.numConnections=20
        #druid.server.http.numThreads=8
        druid.server.http.numThreads=20

        # Processing threads and buffers, 512MB
        druid.processing.buffer.sizeBytes=536870912
        druid.processing.numThreads=24

        druid.server.http.maxIdleTime=PT10m
        druid.broker.http.readTimeout=PT30M

        # Query cache (we use a small local cache)
        druid.broker.cache.useCache=true
        druid.broker.cache.populateCache=true
        druid.broker.cache.unCacheable=["select", "groupBy"]

        # Query Result Count
        druid.query.groupBy.maxResults=100000000
        druid.query.groupBy.maxIntermediateRows=100000000

./druid_dist/druid_dist_v2/druid_bootstrap/conf/druid/historical/runtime.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: jproperties
       :linenos:

        druid.service=druid/prod/historical
        druid.port=8083

        # HTTP server threads
        druid.server.http.numThreads=15

        # Processing threads and buffers, 512MB
        druid.processing.buffer.sizeBytes=536870912
        druid.processing.numThreads=24

        # ex. 50G per disk
        druid.segmentCache.locations=[{"path":"/data1/druid/segment-cache","maxSize":53687091200},{"path":"/data2/druid/segment-cache","maxSize":53687091200},{"path":"/data3/druid/segment-cache","maxSize":53687091200},{"path":"/data4/druid/segment-cache","maxSize":53687091200},{"path":"/data5/druid/segment-cache","maxSize":53687091200},{"path":"/data6/druid/segment-cache","maxSize":53687091200},{"path":"/data7/druid/segment-cache","maxSize":53687091200},{"path":"/data8/druid/segment-cache","maxSize":53687091200},{"path":"/data9/druid/segment-cache","maxSize":53687091200},{"path":"/data10/druid/segment-cache","maxSize":53687091200}]
        druid.server.maxSize=536870912000

        druid.historical.cache.useCache=true
        druid.historical.cache.populateCache=true
        druid.historical.cache.unCacheable=["select", "groupby"]

        druid.query.groupBy.maxResults=100000000
        druid.query.groupBy.maxIntermediateRows=100000000

./druid_dist/druid_dist_v2/druid_bootstrap/conf/druid/middleManager/runtime.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: jproperties
       :linenos:

        # default port, service name.
        druid.service=druid/prod/middlemanager
        druid.port=8091

        # HTTP server threads
        druid.server.http.numThreads=12

        # Processing threads and buffers
        druid.processing.buffer.sizeBytes=256000000
        druid.processing.numThreads=2

        # Hadoop indexing
        druid.indexer.task.hadoopWorkingPath=/druid/indexing-tmp
        druid.indexer.task.defaultHadoopCoordinates=["org.apache.hadoop:hadoop-client:2.7.6"]

        # Task launch parameters
        druid.indexer.runner.javaOpts=-server -Xmx3g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Duser.timezone=UTC -Dfile.encoding=UTF-8 -Djava.util.logging.manager=org.apache.logging.log4j.jul.LogManager
        druid.indexer.task.baseTaskDir=/home/metatron/data1/druid/var/task

        # Peon properties
        druid.indexer.fork.property.druid.monitoring.monitors=["com.metamx.metrics.JvmMonitor"]
        druid.indexer.fork.property.druid.processing.buffer.sizeBytes=536870912
        druid.indexer.fork.property.druid.processing.numThreads=2
        druid.indexer.fork.property.druid.segmentCache.locations=[{"path": "/home/metatron/data1/druid/var/segment-cache", "maxSize": 0}]
        druid.indexer.fork.property.druid.server.http.numThreads=50

        druid.worker.capacity=20
        druid.worker.ip=localhost
        druid.worker.version=0


Step8.Modify deploy envirnonment script - Script 환경 설정 변경 (druid_bootstrap/scripts/druid_env.sh)
==========================================================================================================

./druid_dist/druid_dist_v2/druid_bootstrap/scripts/druid-env.sh
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: jproperties
       :linenos:

        #!/bin/sh
        DRUID_HADOOP_CONF_DIR="/usr/lib/hadoop/etc/hadoop"
        DRUID_LOG_DIR=/home/metatron/data1/druid/var


Step9. Deploy to each cluster server - 배포
=====================================================
설정이나 스크립트가 변경될때마다 먼저 druid_bootstrap 내 수정사항을 지정한후, 위의 두가지 방법을 반복하여 배포

    .. code-block:: console
       :linenos:

        #Move modified conf and script file to binary directory - Binary 내 변경한 설정 파일 및 Script 수정사항 전달
        ./druid_dist/druid_dist_v2/init.sh {binary directory name}

        #Deploy to each cluster server - 실제 서버에 배포
        ./druid_dist/druid_dist_v2/dist.sh {binary directory name}

Step10. Start up Druid services - 가동 절차
=====================================================
    .. code-block:: console
       :linenos:

        #Start up - Druid Cluster 가동
        ./druid_dist/druid_dist_v2/run_druid.sh

        #Status monitoring - Druid Cluster 상태 체크
        ./druid_dist/druid_dist_v2/status_druid.sh

        #Stop druid - Druid Cluster 중지
        ./druid_dist/druid_dist_v2/kill_druid.sh


Step11. 설치 점검
=====================================================
적재 테스트를 위하여 Sample 데이터(sales_samples.tar.gz)를 받아 아래와 같이 수행합니다.

sales_tab_delimeter.csv 를 middlemanager가 가동된 서버내 /tmp 디렉토리로 위치시킵니다.

    .. code-block:: console
       :linenos:

        $ scp sales_tab_delimeter.csv username@hostname:/tmp

적재 명령을 수행합니다. 적재후, 브라우져 상에서 http://{hostname}:8090 을 접속하여 적재 상태를 확인할수 있습니다.

    .. code-block:: console
       :linenos:

        $ curl -X POST -H 'content-type: application/json' -d @sales_ingestion.json http://{hostname}:8090/druid/indexer/v1/task

적재 결과를 확인합니다. 내부 셀 명령어를 보고 hostname 등을 수정하여 수행합니다.

    .. code-block:: console
       :linenos:

       $ ./count_sales_data.sh


Step12. 접속 확인
=====================================================
    * druid overlord - http://OVERLORD-HOST:8090/console.html
    * druid coordinator - http://COORDINATOR-HOST:8081 -> 확인사항 coordinator에 sales 데이터 소스를 확인합니다.
    * druid broker - http://BROKER-HOST:8082