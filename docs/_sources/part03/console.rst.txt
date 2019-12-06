Console
---------------------------------------------
Metatron에서는 Engine을 위한 2가지 Console을 제공합니다.
    * 1. Metatron Discovery 내부의 Conosle
    * 2. Metatron Engine(Druid)에서 기본적으로 제공해주는 Console (Console 설치를 위해서는 Kafka가 필요합니다. druid에 설정에 kafka 경로를 넣어야 합니다)

Console URI
=====================================================
druid overlord - http://OVERLORD-HOST:8090/console.html
druid coordinator - http://COORDINATOR-HOST:8081 
druid broker - http://BROKER-HOST:8082


Connecting metatron Discovery
=====================================================
After the cluster setup, you need to add or update the configuration as shown below to connect with discovery.

    * polaris.engine: The setting used for connecting each druid node.
    * polaris.ingestion: The setting used for copying files to the middle manager on ingestion stage.
    * polaris.query: The setting used when the file download & loads the data source.

    .. code-block:: jproperties
       :linenos:

        polaris:
          engine:
            hostname:
              broker: http://{BrokerHostName}:8082
              overlord: http:// {Overlord Master HostName}:8090
              coordinator: http:// {Coordinator Master HostName}:8081
          ingestion:
            loader:
              remoteType: SSH
                localBaseDir: ${java.io.tmpdir:-/tmp}
                remoteDir: ${java.io.tmpdir:-/tmp}
                hosts:
                  middlemanager_hostname01:
                    port: 22
                    username: metatron
                    password: password
                  middlemanager_hostname02:
                    port: 22
                    username: metatron
                    password: pem:/tmp/metatron.pem
            query:
              loader:
                remoteType: SSH
                localBaseDir: ${java.io.tmpdir:-/tmp}
                remoteDir: ${java.io.tmpdir:-/tmp}
                  hosts:
                    broker_hostname01:
                      port: 22
                      username: metatron
                      password: password


※ ‘middlenamanager_hostname’ must match the hostname in the worker list on overload console.