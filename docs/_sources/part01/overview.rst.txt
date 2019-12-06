기본 어플리케이션 Install
---------------------------------------------

Java
===================================
    * version은 openJDK1.8만 가능합니다.
    * 현재 Metatron에서는 1.8.0_232-b09를 테스트 어플리케이션으로 사용하고 있습니다.

MySQL
===================================
    * Metatron Discovery에서 내장 DB로 사용하고 있습니다. (내장DB는 H2DB로도 가능합니다.)
    * Version은 5.5 이상 가능합니다. (설치 또는 JDBC 지정된 디렉토리에 넣는것도 가능합나다. Metatron에서는 Config 파일에 내장DB 경로가 필요합니다.)

Zookeper
===================================
    * 암바리로 깔기를 권장합니다.
    * druid, Kafka, Hadoop에서 필요한 어플리케이션입니다.
    * 현재 Metatron에서는 3.4.8를 테스트 어플리케이션으로 사용하고 있습니다.
    * Install Apache Zookeeper to act as a Distributed Coordinator.

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
    * 배포를 지원하기 위해 설치를 하지만, 설치는 Option입니다.
    * 여러 서버에 하나하나 깔기 힘들기 때문에 한꺼번에 설정하려는 취지로 설치를 합니다.
    * Ansible is used to deploy Druid to clustered servers. Ansible communicates with client computers through SSH, so each server you want to manage should be accessible from the Ansible server without password.
