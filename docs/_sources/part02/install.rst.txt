Install
---------------------------------------------
Version
=====================================================
    * 빌드시 Hadoop dependency는 apache hadoop 2.6이상 지원, 권고 버전 2.7 입니다.
    * Metatron에서는 2.7.3를 테스트 버젼으로 사용하고 있습니다. 또한 Metatron Engine (Druid)에서 2.7.3을 기준으로 Compile하여 사용하고 있습니다.
    * 단, 암바리 기준으로 해당 버젼은 2.6.x입니다.
    * Metatron Discovery에서는 Hadooop 3점대도 지원합니다. 만약 사용을 원하는 경우 Druid를 해당 버젼으로 Compile하여야 합니다.
    * apache hadoop 3.0이상인 경우 https://tde.sktelecom.com/stash/projects/METATRON/repos/druid/commits/ef86bf47a86253e354490e04a8f18f77415e8661 cherry-pick해야합니다.
    * CDH 인 경우 다음과 같이 druid내 pom.xml을 수정 한후 mvn clean package -DskipTests=true -Dhadoop.compile.version=2.6.0-cdh5.12.1 로 빌드하여야 합니다.
    * 현재 Metatron에서는 Hadoop 3.1.4 HDP에서 테스트를 완료하였습니다.
    * Hadoop minor버젼에 영향을 많이 받기 때문에, 저희가 테스트해보지 않은 Hadoop Version에 대해서는 반드시 아래와 같은 절차로 테스트 해보기 권장합니다.

Install Test
=====================================================
    * 파일데이터: 세일즈데이터
    * 하이브테스트데이트 : ???
    * 순서 : 코디네이터, 오버로드테스트 => 파일 적재 테스트 => 하이브 적재테스트
