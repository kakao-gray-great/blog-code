
Elastic APM은 **무료 개방형 애플리케이션 성능 모니터링** 프로그램입니다.

애플리케이션이 정확히 어디에서 시간을 사용하는지 파악하여 신속하게 문제를 수정하고 성능을 개선해 나갈 수 있습니다.

저는 제가 현재 진행하고 있는 `jagoga` 프로젝트에 Elastic APM을 설치해보았습니다.

Spring Boot의 웹 서버를 Elasticsearch, Kibana를 이용해서 APM 연결해보겠습니다.

## 1. docker-compose 구성하기

https://www.elastic.co/kr/downloads/apm#ga-release

Elastic 공식 사이트를 보게 되면 친절하게 순서가 나와 있습니다.

![](https://i.imgur.com/mRrcXPO.png)

Elasticsearch, Kibana를 설치하기 위해서 링크를 들어가 보겠습니다.

Elasticsearch를 위한 다운로드 목록들이 나오는 것을 볼 수 있습니다.

저희는 Docker를 사용할 것이기에 아래 빨간 네모 링크를 눌러줍니다.

![](https://i.imgur.com/J4KbTs1.png)

Docker에서 Elasticsearch를 설치하기 위한 방법이 나오고 이것을 저희 프로젝트에 맞게 작성해주겠습니다.

kibana도 동일하게 진행해줍니다.

```yml=
version: '2'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    container_name: elasticsearch
    environment:
      - node.name=elasticsearch
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=elasticsearch
      - cluster.initial_master_nodes=elasticsearch
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./volume/elasticsearch-volume:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    container_name: kibana
    ports:
      - 5601:5601
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    networks:
      - elastic

  mysql:
    image: mysql:5.7
    volumes:
      - ./volume/jagoga-mysql-volume:/var/lib/mysql
    restart: always
    container_name: mysql_db
    environment:
      MYSQL_ROOT_PASSWORD: jagogaqwer1234
      MYSQL_DATABASE: jagoga_db
      MYSQL_USER: admin
      MYSQL_PASSWORD: jagogaqwer1234
    ports:
      - "3307:3306"

volumes:
  elasticsearch-volume:
    driver: local
  jagoga-mysql-volume:
    driver: local

networks:
  elastic:
    driver: bridge

```

Elasticsearch, Kibana, APM을 진행했다면 docker-compose 명령을 사용하여 도커를 실행시켜줍니다.

```
dockcer-compose up
```

## 2. APM Server 다운로드

http://localhost:5601/ 에 접속하여, Add your data -> APM을 선택 후, APM으로 이동합니다.

![](https://i.imgur.com/jEGdIkx.png)

![](https://i.imgur.com/f6LE3Yc.png)

저는 MAC 유저라 macOS를 선택하고 가이드대로 진행해줍니다.

![](https://i.imgur.com/7WEM20F.png)

### 2-1. Dowload and unpack APM Server

터미널에 아래의 명령어들을 순서대로 입력해줍니다.

```
curl -L -O https://artifacts.elastic.co/downloads/apm-server/apm-server-7.15.0-darwin-x86_64.tar.gz

tar xzvf apm-server-7.15.0-darwin-x86_64.tar.gz

cd apm-server-7.15.0-darwin-x86_64/
```

![](https://i.imgur.com/2lZY7wf.png)

### 2-2. Edit the configuration

방금 설치한 apm-server 디렉터리로 이동하여 apm-server.yml 파일을 수정해줍니다.

```yaml=
#-------------------------- Elasticsearch output --------------------------
output.elasticsearch:
  # Array of hosts to connect to.
  # Scheme and port can be left out and will be set to the default (`http` and `9200`).
  # In case you specify and additional path, the scheme is required: `http://localhost:9200/path`.
  # IPv6 addresses should always be defined as: `https://[2001:db8::1]:9200`.
  hosts: ["localhost:9200"] # localhost:9200 으로 hosts 지정
```

### 2-3. Start APM Server

설정을 마치셨다면 아래 명령어를 통해 APM Server를 실행시켜주세요.

```
./apm-server -e 
```

## 3. APM agent 다운로드

Spring 기반으로 프로젝트를 진행하기 때문에 Java를 선택하였습니다.

![](https://i.imgur.com/uW0dWRd.png)

APM agent도 위의 가이드대로 진행해줍니다.

### 3-1. Download the APM agent

Maven Central 링크로 들어가 jar 파일을 직접 다운로드 받아줍니다.

![](https://i.imgur.com/wCCZFVJ.png)

![](https://i.imgur.com/el2NZRl.png)


### 3-2. Start your application with the javaagent flag

`jagoga` 프로젝트 서버 구동 시 agent를 함께 실행하게 할 수 있습니다.

Intellij 상단에 Run - Edit Configurations...를 선택합니다.

![](https://i.imgur.com/3kOH9oh.png)

VM Options에 javaagent flag를 입력해줍니다.

```java=
-javaagent:/Users/bennie/SourceCode/Project/f-lab/jagoga/elastic-apm-agent-1.26.0.jar -Delastic.apm.service_name=jagoga 
    -Delastic.apm.server_url=http://localhost:8200 
    -Delastic.apm.application_packages=com.project.jagoga 
    -Delastic.apm.transaction_sample_rate=1
```
![](https://i.imgur.com/h9OkeeX.png)

이렇게 구성이 완료되면 Application을 실행시켜줍니다.

그 후 우측 하단에 **Launch APM**을 통해 APM을 실행시킬 수 있습니다.

![](https://i.imgur.com/u0gWbLz.png)

## 4. APM 사용

APM 페이지에 `jagoga` 프로젝트가 정상적으로 보여지는 것을 확인할 수 있습니다.

![](https://i.imgur.com/K9MNQpJ.png)

POSTMAN을 사용하여 데이터를 전송해보겠습니다.

![](https://i.imgur.com/mHiapaE.png)

아래와 같이 APM이 정상적으로 동작하는 것을 확인할 수 있습니다.

![](https://i.imgur.com/lVqgsxv.png)


## 참고

* https://www.elastic.co/kr/apm/
* https://github.com/f-lab-edu/blue-delivery/blob/main/compose/docker-compose.yml
* https://cheese10yun.github.io/elk-apm-1/
