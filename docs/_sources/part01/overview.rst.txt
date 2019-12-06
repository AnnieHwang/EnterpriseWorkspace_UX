Install
---------------------------------------------

Java
===================================
    * version은 openJDK1.8만 가능합니다.
    * 현재 Metatron에서는 1.8.0_232-b09를 테스트 어플리케이션으로 사용하고 있습니다.

MySQL
===================================
    * Metatron Discovery에서 내장 DB로 사용하고 있습니다. (내장DB는 H2DB로도 가능합니다.)
    * Version은 5.5 이상 가능합니다. (설치 또는 JDBC 지정된 디렉토리에 넣는것도 가능합나다. Metatron에서는 Config 파일에 내장DB 경로가 필요합니다.)

설정
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
mysql 또는 mariadb (5.5 기준) 를 설치하거나 사전에 설치한 곳에 아래와 같이 druid metadata 전용 databases 를 생성합니다.
해당 database는 다른 서버에서 접근할 수 있도록 access권한을 부여해야 합니다.

    * database name : druid
    * user : druid
    * password : druid

    .. code-block:: jproperties
       :linenos:

        create database druid CHARACTER SET utf8;
        grant all privileges on druid.* TO druid@localhost identified by 'druid';
        grant all privileges on druid.* TO druid@'%' identified by 'druid';
        flush privileges;

Zookeper
===================================
    * 암바리로 깔기를 권장합니다.
    * druid, Kafka, Hadoop에서 필요한 어플리케이션입니다.
    * 현재 Metatron에서는 3.4.8를 테스트 어플리케이션으로 사용하고 있습니다. (3.4.10 이상????)
    * Install Apache Zookeeper to act as a Distributed Coordinator.

설정
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
분산 코디네이터 역할을 위해 Apache Zookeeper 설치. 별도 운영상 가이드 없고, 3대 이상으로 설치 합니다.

    .. code-block:: jproperties
       :linenos:

        curl http://www.gtlib.gatech.edu/pub/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz -o zookeeper-3.4.10.tar.gz
        tar -xzf zookeeper-3.4.10.tar.gz
        cd zookeeper-3.4.10
        cp conf/zoo_sample.cfg conf/zoo.cfg
        ./bin/zkServer.sh start

Kafka
===================================
    * 암바리로 깔기를 권장합니다.
    * Version은 1.0 이상 사용이 가능합니다.
    * Metatron에서는 2점대를 테스트 어플리케이션으로 사용하고 있습니다. (1점대 보단 2점대가 성능이 훨씬 좋아 사용을 권장합니다.)

Spark
===================================
    * 암바리로 깔기를 권장합니다.
    * 설치는 Option 입니다.
    * Metatron Discovery 내 Preparation 기능에서 고성능 가속 기능을 사용하기 위헤 필요합니다. (만약 이 기능을 사용하지 않고 Hive를 사용하신다면 해당 어플리케이션은 필요없습니다.)


Ansible
===================================
    * Druid 배포를 지원하기 위해 설치를 하지만, 설치는 Option입니다.
    * 여러 서버에 하나하나 깔기 힘들기 때문에 한꺼번에 설정하려는 취지로 설치를 합니다.
    * CentOS 기준 설치를 확인 드립니다. OS가 다를 경우 다음 링크 참고:(https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
    * Yum 으로 설치가능하지만, Yum 으로 설치가 되지 않는 환경에서는 http://releases.ansible.com/ansible/rpm/release/ 에서 rpm 을 OS 버젼에 맞게 다운로드 받습니다.
    * 추가로, python-jinja2 가 Depnedency가 걸려 있을수 있으니, 필요시 먼저 설치해야합니다.