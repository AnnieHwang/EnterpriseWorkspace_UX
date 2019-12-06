기본 Install 순서 가이드
---------------------------------------------

Step1. Java
===================================
Metatron Engine(Druid)를 위해서는 openJDK1.8만 사용 가능

Step2. MySQL
===================================
Metatron Discovery의 내장 DB로 사용됨

Step3. Zookeper
===================================
Druid, Kafka, Hadoop에서 필요함

Step4. Ansible (Option)
===================================
Ansible is used to deploy Druid to clustered servers. Ansible communicates with client computers through SSH, so each server you want to manage should be accessible from the Ansible server without password.
    * 배포 지원용임.
    * 여러 서버에 하나 하나 설치에 어려움이 있는 경우 한꺼번에 설정할 수 있어 설치함

Step5. Hadoop
===================================
분산환경에서 사용하기 위해서는 필요한 어플리케이션임 (단, 싱글일땐 필요없음)

    * 필요사항: Java, Zookeper가 필요함
    * 설치가 까다로움.(현재 Druid는 Hadoop 2.7로 compile되어 있음)
    * Hadoop과 함께 설치되는 Hive는 Metatron Discovery내에서 StagingDB(Slave DB)로 사용됨
    * Hive 이외의 yarn 등이 설치되어야 함

Step6. Metatron Engine (Druid)
===================================
Metatron Discovery에 필요함

    * Apache Druid와 다름. 사상은 같으나, 이미 소스가 많이 갈라

Step7. Kafka
===================================
Metatron Discovery 내 Engine Monitoring을 위해 필요함. 실시간 데이터 처리 시를 위해 필요함

Step8. Spark (Option)
===================================
Metatron Discovery 내 Preparation에 고속 처리를 위해 사용할 경우 필요함

Step9. Jupyter (Option)
===================================
Metatron Discovery 내 Notebook 사용에 필요함

Step10. Zeppelin (Option)
===================================
Metatron Discovery 내 Notebook 사용에 필요함

Step11. Metatron Discovery
===================================

Step12. Metatron Anomaly
===================================

Step13. Oozie
===================================
Metatron Integrator에 필요함.
배치 프로세스 어플리케이션임

Step14. Active MQ
===================================
Metatron Integrator에 필요함

Step15. Metatron Integrator
===================================