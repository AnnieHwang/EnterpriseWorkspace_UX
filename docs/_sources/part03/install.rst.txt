Install
---------------------------------------------
Metatron의 시계열 엔진입니다. 모든 Metatron 제품은 해당 엔진을 사용하고 있습니다.

    * 우리는 Apache Druid와 다름. (동일하게 설치하면 절대 안됨)
    * 소스는 discovery 올라갈때 그 안에 들어있음
    * 현재 버젼은 Metatron Discovery 버젼을 사용하고 있음
    * druid 안에 hadoop 라이브러리가 들어가 있음
    * 현재 Hadoop 2.7.3로 컴파일 됨. (만약 3버젼대를 사용하려면 해당 Hadoop 라이브러리로 컴파일 해야함)
    * 싱글과 분산환경 버젼은 동일함 (같은 라이브러리를 사용해도 됨)



버젼
=====================================================
    * 우리의 대표 2.7.3로 기준으로 default로 제공함 (암바리 기준으로 2.6.x임)
    * 3점대도 지원함. 이 경우 따로 building을 해야함 (현재 3.1.4 HDP 테스트 했음)
    * 버젼을 사이트 별로 따로 관리되어야 함 (물어보면 알려줄 것임)
    * 현재 hadoop 2.7대로 druid가 building 되어 있음. hadoop 3.x 대 버젼으로 사용하려면 druid를 building해야 함.
    * hadoop minor버젼에 영향을 많이 받기 때문에..위의 버젼이 아닌 경우, 테스트 해보기 바란다.

버젼호환 테스트
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
버젼을 변경하여 컴파일 하면 다음과 같은 테스트가 필요함 (정상작동되나 확인하기 위해서임)
테스트순서는 아래와 같습니다.

    * 파일 샘플 데이터: 세일즈데이터를 사용했음
    * 하이브 테스트 데이터 :XXX을 사용했음
    * 테스트 순서 : Druid 코디네이터, 오버로드 테스트 => 파일 적재 테스트 => 하이브 적재테스트



1.Configure directories in hdfs as below
=====================================================

    .. code-block:: console
       :linenos:

        hadoop fs -mkdir -p /druid/storage
        hadoop fs -mkdir -p /druid/logs

2.Download customized Druid & Unzip
=====================================================
Download and Unzip the binary file. This Link – for Hadoop 2.7.3

    .. code-block:: console
       :linenos:

        Example location>
        ./druid_dist/druid-0.9.1-SNAPSHOT.{discovery.version}-hadoop-2.7.3

3.Download deploy template shell & Unzip
=====================================================
Download and Unzip the binary file.  This Link

    .. code-block:: console
       :linenos:

        Example Location>
        ./druid_dist/druid_dist_v2

4.Ansible  Host Setting
=====================================================
Ansible keeps track of all of the servers that it known about through a “hosts”. We need to set up this file first before we can begin to communicate with our other computers. Refer to the host file under the ./druid_dist/druid_dist_v2 directory and set it in /etc/ansible/hosts.

5.Modify deploy environment shell
=====================================================

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


6.Create necessary directory by shell
=====================================================
./druid_dist/druid_dist_v2/setup_druid.sh

7.Configuring Druid’s property file
=====================================================
The major configuration has already been put in the "bootstrap" directory. Below is a list of things you need to change or update depending on your environment

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



./druid_dist/druid_dist_v2/druid_bootstrap/conf/druid/coordinator/runtime.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: bash
       :linenos:

        # Default host, port, service name.
        druid.service=druid/prod/coordinator
        druid.port=8081

./druid_dist/druid_dist_v2/druid_bootstrap/conf/druid/overlord/runtime.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: python
       :linenos:

        # Default host, port, service name.
        druid.service=druid/prod/overlord
        druid.port=8090

./druid_dist/druid_dist_v2/druid_bootstrap/conf/druid/broker/runtime.properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: python
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
    .. code-block:: python
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
    .. code-block:: python
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


8.Modify deploy envirnonment script
=====================================================

./druid_dist/druid_dist_v2/druid_bootstrap/scripts/druid-env.sh
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    .. code-block:: python
       :linenos:

        #!/bin/sh
        DRUID_HADOOP_CONF_DIR="/usr/lib/hadoop/etc/hadoop"
        DRUID_LOG_DIR=/home/metatron/data1/druid/var


9.Deploy to each cluster server
=====================================================
    .. code-block:: python
       :linenos:

        #Move modified conf and script file to binary directory
        ./druid_dist/druid_dist_v2/init.sh {binary directory name}

        #Deploy to each cluster server
        ./druid_dist/druid_dist_v2/dist.sh {binary directory name}

10.Start up Druid services
=====================================================
    .. code-block:: python
       :linenos:

        #Start up
        ./druid_dist/druid_dist_v2/run_druid.sh

        #Status monitoring
        ./druid_dist/druid_dist_v2/status_druid.sh

        #Stop druid
        ./druid_dist/druid_dist_v2/kill_druid.sh