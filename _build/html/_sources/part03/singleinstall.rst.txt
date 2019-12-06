분산 환경 Install
---------------------------------------------
Step1. Build
=====================================================
Druid 서버를 로컬에 설치시, 아래 다운로드링크를 통해 바이너리를 설치하면, Build 단계는 건너뛰어도 됩니다.
빌드보다는 최신 다운로드 본을 Metatron팀에 의뢰하여, 다운로드 받으시는것을 권고 드립니다.

    * 1. Source Code 다운로드

    .. code-block:: console
       :linenos:

        $ git clone https://{username}@tde.sktelecom.com/stash/scm/metatron/druid.git


    * 2. 첨부한 Build Script 를 통해 빌드 수행

    .. code-block:: console
       :linenos:

        $ git checkout latest
        $ build_druld.sh --tag={지정하고자하는식별명}


    * 3.빌드 성공 후, druid-0.9.1-SNAPSHOT.local.{Current Datetime}-bin.tar.gz 파일이 생성


Step2. Deploy
=====================================================

    * 1. Druid 용 디렉토리를 생성합니다.
    * 2. Build 시 생성되었던 바이너리 파일을 druid 용 디렉토리에 압축을 해제합니다. (

    .. code-block:: console
       :linenos:

       $ tar zxf druid-0.9.1-SNAPSHOT.local.{Current Datetime}-bin.tar.gz

- 압축을 해제하면, druid 바이너리가 포함된 디렉토리가 생성됩니다.

    * 3. 첨부한 initial 용 파일도 압축해제합니다. (기본 스크립트 및 sales 데이터 소스 포함)

    .. code-block:: console
       :linenos:

       $ tar zxf druid-init.tar.gz

- 압축을 해제하면 druid_bootstrap_local 디렉토리와, init.sh 스크립트가 생성됩니다.

    * 4. Init 스크립트를 수행합니다.

    .. code-block:: console
       :linenos:

       $ ./init.sh [druid 바이너리 디렉토리]

- 스크립트를 실행하면, druid 라는 druid 바이너리 디렉토리 와 링크된 디렉토리가 생성되며, 내부에 실행/중지 스크립트 및 데이터 파일이 포함이 됩니다.

    * 5. 서버를 실행합니다.

    .. code-block:: console
       :linenos:

       $ cd druid
       $ ./start-single.sh | ./stop-single.sh

- 로그를 확인하시려면, log/single.log 파일을 확인해주세요. (2017-08-08T07:00:45,071 INFO [main] org.eclipse.jetty.server.Server - Started @11120ms 가 나타나면 성공)
※ 추후 바이너리가 배포 되면 압축해제 후, 3번 수행사항을 제외하고 반복하시면됩니다.

Step3. 접속확인
=====================================================
    * druid overlord - http://localhost:8090/console.html
    * druid coordinator - http://localhost:8081

