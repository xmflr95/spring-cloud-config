# MSA(MicroService Architecture)

#### Eureka Random 설정

```md
eureka:
  instance:
    instance-id: ${spring.application.name}:${spring.application.instance_id:${random.value}}
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://127.0.0.1:8761/eureka
```

### Spring Security 설정

 * step1: 애플리케이션에 spring security jar을 Dependency에 추가
 * step2: WebSecurityConfigurerAdapter를 상속받는 Security Configuration 클래스 생성
 * step3: Security Configuration 클래스에 @EnableWebSecurity 추가
 * step4: Authentication -> configure(AuthenticationManagerBuilder auth) 메서드를 재정의
 * step5: Password encode를 위한 BCryptPasswordEncoder 빈(Bean) 정의
 * step6: Authorization -> configure(HttpSecurity http) 메서드를 재정의

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

패스워드 암호화는 BCryptPasswordEncoder를 사용
 - Password를 해싱하기 위해 BCrypt 알고리즘 사용
 - 랜덤 Salt를 부여하여 여러번 Hash를 적용한 암호화 방식

자바 직렬화의 이유는 DB 저장등에서 마셜링 작업을 하기위함

### 로그인 인증
 * Spring Security를 이용한 로그인 요청 발생 시 작업을 처리해주는
   Filter 클래스 생성
 * UsernamePasswordAuthenticationFilter 상속
 * attemptAuthentication(), successfulAuthentication() 함수 구현

filters:
  - RewritePath=/user-service/(?<segment>.*), /$\{segment}
* 필터를 통해 MicroService로 요청되는 url에는 'user-service'등의 이름은 빠질 수 있게 해줌

#### JWT(Json Web Token) [https://jwt.io/]
---
전통적인 인증 시스템(쿠기, 세션)의 문제점
  - 세션과 쿠키는 모바일 Application에서 유효하게 사용 불가능(공유 불가)
    - 예를 들어 (React <-> Java Server) 는 세션 고유 불가
  - 렌더링된 HTML 페이지가 반환되지만, 모바일 애플리케이션에서는 JSON(or XML)과 같은 포멧 필요

그로인해 새로 나온것이 Token 기반 인증 시스템(JWT 등...)  
* JWT 
  - 인증 헤더 내에서 사용되는 토큰 포맷
  - 두 개의 시스템끼리 안전한 방법으로 통신 가능

* 장점
  - 클라이언트 독립적인 서비스(stateless)
  - CDN
  - No Cookie-Session(No CSRF, 사이트간 요청 위조)
  - 지속적인 토큰 저장 (한 서비스의 다른 서버간 인증을 다른 DB에 저장된 걸로 연속적으로 처리 가능)

 api gateway에 JWT 디펜던시를 추가하는 이유  
 서비스의 토큰값이 정상적인지 확인하기 위해서

 * 회원가입과 로그인은 인증(Authenticate)가 필요없음.
 * 필터가 필요한 부분은 회원정보 확인, 상태 확인 등 그외 나머지 기능에서 필요함

tip) 최신 자바 버전 javaEE 모듈 CORBA 모듈 Deprecated 문제 해결법
```xml
<dependency>
  <groupId>org.glassfish.jaxb</groupId>
  <artifactId>jaxb-runtime</artifactId>
</dependency>
```

## Spring Cloud Config
* 분산 시스템에서 서버, 클라이언트 구성에 필요한 설정 정보(application.yml)를 외부 시스템에서 관리
* 하나의 중앙화된 저장소에서 구성요소 관리 가능
* 각 서비스를 다시 빌드하지 않고, 바로 적응 가능
* 애플리케이션 배포 파이프라인을 통해 **DEV-UAT-PROD** 환경에 맞는 구성 정보 사용

-> 하나의 통일된 config파일을 구성하여 관리함

##### changed configuration values(설정 파일 변경 적용방법)
- 서버 재기동(서비스) -> spring cloud config 사용의미가 줄어듬
- Actuator refresh -> 재부팅없이 얻어오기 가능
- Spring cloud bus 사용 -> 

##### Spring Boot Actuator
 - Application 상태, 모니터링
 - Metric 수집을 위한 Http End point 제공
 - 마이크로 서비스의 수가 많으면 각각 서비스 관리가 힘듬(리프레쉬 관리)

### Spring Cloud Bus
 - 분산시스템의 노드를 경량 메시지 브로커와 연결
 - 상태 및 구성에 대한 변경 사항을 연결된 노드에게 전달(Brodcast)
 - middleware 느낌
 - 각각의 actuator 작업을 하지 않더라도 한번에 동기화를 가능하게하는것

AMQP(Advanced Message Queuing Protocol) 
Spring Cloud Config Server + Spring Cloud Bus

##### 윈도우 환경 RabbitMQ 설치 방법
1. Erlang 다운로드 및 설치 -> 환경변수 설정 필요
2. RabbitMQ 다운로드 및 설치 -> 환경변수 설정 필요
3. RabbitMQ 플러그인 설치 -> cmd, powershell 등
    rabbitmq-plugins enable rabbitmq_management


#### Message Broker
---
**Message Broker**(메시지 브로커)는 **Publisher**(송신자)로부터 전달받은 메시지를 
**Subscriber**(수신자)로 전달해는 중간 역할이며 응용 소프트웨어간에 메시지를 교환할 수
있게 한다. 이 때 메시지가 적재되는 공간을 **Message Queue**(메시지 큐)라고 하며 메시지 
그룹을 **Topic**(토픽)이라고 한다.

예를들어 두 개의 서버가 존재, DW는 실시간 데이터 수집 관리하는 서버, AS는 이 데이터를
가공하여 사용하는 서버일 경우, AS에서 DW에 있는 데이터를 사용하기 위해서는 어떤 방법을 써야하는가?
**->** 일반적인 DB 저장 방법의 경우 DW에서 Oracle, MySQL과 같은 RDB에 저장 후, AS에서 이 
DB를 조회해서 쓰는 것이 일반적. 그러나 실시간 처리에서 해당 방법은 실시간으로 데이터가 
쌓이는 TABLE을 계속해서 조회해야하기 때문에 빠른 조회가 힘들다. 그걸 막기위해 INDEX를 걸면
INSERT 속도가 느려지므로 실시간 처리에 부적합하다
**메시지브로커**를 사용하는 방법은 DW에서 수집한 데이터를 바로 **메시지 큐**에 Publish하고
AS는 Subscribe하여 바로 사용가능하게 된다. 메시지 브로커를 사용하면 AS에서는 별도의
조회과정이 필요없이, 메시지 큐에 적재되는 메시지를 감시하고 있다가 메시지가 적재되면 바로
가져다가 사용 할 수 있다.

* 장점)
  - 구독하는 쪽에서는 별도의 조회과정이 필요없이 메시지 큐에 적재되는 메시지를 감시,
 바로 적재되면 바로 가져다 사용할 수 있다.
  - 실시간 데이터 처리 시 DB 조회하는 것보다 메시지 브로커를 이용하여 처리하는 것이
 성능이 뛰어남
* 단점)
  - DB를 사용하면 Query를 이용하여 원하는 데이터만 필터링하여 조회가 가능하지만, 메시지 브로커의
 경우에는 Queue를 적재된 그대로 사용하기 때문에 불가능함.
  - 따라서 적재 시 필터링된 데이터를 적재하던가 적재된 데이터를 Logstash를 이용하여 필터링해서 
 사용해야 한다. 또한, 메시지 큐에 적재된 메시지는 주로 7일을 보관하기 때문에 장기간 보관해야하는
 경우 별도의 저장소에 저장이 필요함

# Section2) 가상화 환경에서 프로젝트 설정

## 암호화 & 복호화  
Encryption & Decryption  
- Symmetric Encryption(Shared)
  - Using the same key
- Asymmetric Encryption(RSA Keypair)
  - Private and Public Key
  - Using Java keytool

**{cipher}** : 이게 붙은 해쉬값은 암호라는 뜻을가짐 ex) {cipher}a93997a5da8b87...

encrypt.key => 대칭키

##### Asymmetric Encryption
  * Public, Private Key 생성 -> JDK keytool 이용
  * $mkdir ${user.home}/Desktop/Work/keystore <- 키스토어 폴더 생성
  * $keytool -genkeypair -alias apiEncryptionKey -keyalg RSA \
  -dname "CN=Kenneth Lee, OU=API Development, O=joneconsulting.co.kr, L=Seoul, C=KR" \ 
  -keypass "1q2w3e4r" -keystore apiEncryptionKey.jks -storepass "1q2w3e4r"

일반적으로 암호화 할때는 Private Key 사용, 복호화시에는 Public Key 사용
```yaml
  # window에서 location 설정 시
  key-store:
    location: file:///${user.home}/Desktop/Work/keystore/apiEncryptionKey.jks
  # 위 처럼 file:/// slash를 3개 넣어줘야함(윈도우는 드라이브를 명시하는 부분이 있기 떄문)
```  
* keytool 명령어 예시
```bash
keytool -genkeypair -alias apiEncryptionKey -keyalg RSA -dname "CN=Kenneth Lee, OU=API Development, O=joneconsulting.co.kr, L=Seoul, C=KR" -keypass "test1234" -keystore apiEncryption.jks -storepass "test1234"
```  
* keytool: Key 정보 확인
```bash
keytool -list -keystore apiEncryption.jks -v (-v 옵션을 제거하면 짧은 정보만 표시)
```
* 키 export/import  
```bash
--export
keytool -export -alias apiEncryptionKey -keystore apiEncryptionKey.jks -rfc -file trustServer.cer
--import (Public Key 생성/서명 가능)
keytool -import -alias trustServer -file trustServer.cer -keystore publicKey.jks
```  
## 마이크로서비스(MSA)간 통신
* **Communication Types**
  * Synchronous HTTP communication
  * Asynchronous communication over AMQP(비동기방식)

* **Feign Web Service Client**
  * FeignClient -> HTTP Client
    - REST Call을 추상화 한 Spring Cloud Netflix Library
  * 사용방법
    - 호출하려는 HTTP Endpoint에 대한 Interface를 생성
    - @FeignClient 선언
  * Load Banlanced 지원

* Feign Client 적용
  * Spring Cloud Netflix 라이브러리 추가
  * @FeignClient Interface 생성
  * UserServiceImpl.java에서 Feign Client 사용
 
### Feign Client
* 장점
  * 코드의 양이 적음
  * 직관성이 높음
* 단점
  * 다른 측면에서는 다른 서비스 제작자가 아닌 경우에는 어떤 서비스인지
  헷갈릴 가능성이 높음

* Feign Client 사용 시 발생한 로그 추적
* Feign 예외 처리
  * FeignException 처리
  * ErrorDecoder 구현


* **Multiple Orders Service**
 다중 클라이언트 요청이 들어왔을 경우의 처리
  * Order Service 2개 기동
    * Users의 요청 분산 처리
    * Orders 데이터도 분산 저장 -> 동기화 문제!
    * 하나의 Database를 사용(아주 간단한 해결법, Monolthic), 트랜잭션 관리가 매우 필요함
    * Database간의 동기화 -> 메시지 큐잉 서버를 이용하여 동기화
    * Kafka Connector + DB (복합 처리)
      * 서로 다른 인스턴스 데이터를 메시지 큐에 저장(미들웨어)
      * 그 데이터를 하나의 단일 DB에 저장
      * 이후 동일한 각각의 서비스에서 DB에 있는 데이터를 가져가 사요

## Apache Kafka
> 오픈 소스 메시지 브로커
  실시간 데이터 피드를 관리하기 위해 통일된 높은 처리량(대용량), 낮은 지연 시간을 지닌 플랫폼 제공  

* 기존 서비스들의 구성
  * End-to-End 연결 방식의 아키텍처
  * 데이터 연동의 복잡성 증가 (HW, 운영체제, 장애 등)
  * 서로 다른 데이터 Pipeline 연결 구조
  * 확장이 어려운 구조

* 카프카의 탄생 이유
  * **모든 시스템으로 데이터를 실시간으로 전송하여 처리할 수 있는 시스템**
  * **데이터가 많아지더라도 확장이 용이한 시스템**

* 카프카의 특징
  * Producer/Consumer 분리
  * 메시지를 여러 Consumer에게 허용
  * 높은 처리량을 위한 메시지 최적화
  * Scale-out 가능
  * Eco-system

  ##### Kafka Broker
  * 실행 된 Kafka 애플리케이션 서버
  * 3대 이상의 Broker Cluster 구성
  * Zookeeper 연동
    * 역할: 메타데이터(Broker ID, Controller ID 등) 저장
    * Controller 정보 저장
  * n개 Broker 중 1대는 Controller 기능 수행
    * Controller 역할 
      * 각 Broker에게 담당 파티션 할당 수행
      * Broker 정상 동작 모니터링 관리
  
### Ecosystem (1)-Kafka Client
  * Kafka와 데이터를 주고 받기위해 사용하는 Java Library
    - [https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients]
  * Producer, Consumer, Admin, Stream 등 Kafka 관련 API 제공
  * 다양한 3rd party library 존재: C/C++, Node.js, Python, .NET 등
    - [https://cwiki.apache.org/confluence/display/KAFKA/Clients]

(Kafka Cluster) <<--->> (Kafka-client Application)

##### Kafka 서버 기동
  * Zookeeper 및 Kafka 서버 구동
    - $KAFKA_HOME/bin/zookeeper-server-start.sh $KAFKA_HOME/config/zookeeper.properties
    - $KAFKA_HOME/bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties
    - **windows**
    - .\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
    - .\bin\windows\kafka-server-start.bat .\config\server.properties
  * Topic 생성
    - $KAFKA_HOME/bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092 \ --partitions 1
  * Topic 목록 확인
    - $KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
  * Topic 정보 확인
    - $KAFKA_HOME/bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
  * 메시지 생산
    - $KAFKA_HOME/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic quickstart-events
  * 메시지 소비
    - $KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events \

### Ecosystem (2)-Kafka Connect
  * Kafka Connect를 통해 Data를 Import/Export 가능
  * 코드 없이 Configuration으로 데이터를 이동
  * Standalone mode, Distribution mode 지원
    - RESTful API 통해 지원
    - Stream 또는 Batch 형태로 데이터 전송 가능
    - 커스텀 Connector를 통한 다양하 Plugin 제공(File, S3, Hive, MySQL, etc...)
 * 데이터를 가져오는 쪽 : Connect Source
 * 데이터는 보내는 쪽 : Connect Sink
(Source System:Hive,jdbc)->(Kafka Connect Source)->(Kafka Cluster)->(Kafka Connect Sink:export)->(Target System:S3...)

###### MariaDB 설치
  * pom.xml mariaDB dependency 추가 후 적용 -> connection 오류가 날 경우 mysql connector를 dependency에 추가시키면 해결(mariadb 버전에 맞는 커넥터 버전 사용)

### Kafka Connect 설치
  * [https://confluent.io]
  * .\bin\windows\connect-distributed.bat .\etc\kafka\connect-distributed.properties

### 데이터 동기화 (1) - Orders -> Catalogs
  * Order Service에 요청 된 주문의 수량 정보를 Catalogs Service에 반영
  * Order Service에서 Kafka Topic으로 메시지 전송 -> Producer
  * Catalogs Service에서 Kafka Topic에 전송 된 메시지 취득 -> Consumer

##### Multiple Orders Service에서의 데이터 동기화
  * Orders Service 2개 기동
    - Users의 요청 분산 처리
    - Orders 데이터도 분산 저장 -> **동기화 문제**

### 데이터 통기화 (2) - Multiple Order Service
  * Order Service에 요청 된 주문 정보를 DB가 아니라 Kafka Topic으로 전송
  * Kafka Topic에 설정 된 Kafka Sink Connect를 사용해 단일 DB에 저장 -> 데이터 동기화
  * tip) 단일 DB 데이터 컬럼에 어느 인스턴스에서 발생한 것인지 보여주도록 기록하면 좀 더 풍부한 DATA를 가진 DB 구성 가능

# 장애 처리와 Microservice 분산 추적

> Microservice 통신 시 연쇄 오류 발생의 경우  
  - 다른 마이크로서비스와 통신을 하다 한 곳이라도 오류가 발생한 경우
  - 오류가 생긴 마이크로서비스로의 통신(요청) 진행을 막아야함
  - Feign Client 측에서는 에러 발생시 임시로 보여줄 데이터(디폴트값) 혹은 우회할 수 있는 데이터를 보여줘야한다.
  - 데이터가 없지만 오류는 보이지 않도록 처리한다.

### CircuitBreaker
  - [https://martinfowler.com/bliki/CircuitBreaker.html]
  - 장애가 발생하는 서비스에 반복적인 호출이 되지 못하게 차단
  - 특정 서비스가 정상적으로 동작하지 않을 경우 다른 기능으로 대체 수행 -> 장애 회피

##### Spring Cloud Betflix Hystrix
  * 현재는 개발은 하지않고 유지보수만 진행 중 - Resilience4j
  * Hystrix Dashboard/Turbine - Micrometer + Monitoring System
  
### resilience4j
  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
  </dependency>
  ```

### Microservice 분산 추적
  * Zipkin
    - [https://zipkin.io/]
    - Twitter에서 사용하는 분산 환경의 Timing 데이터 수집, 추적 시스템(오픈소스)
    - Google Drapper에서 발전하였으며, 분산환경에서의 시스템 병목 현상 파악
    - Collector, Query Service, Databasem WebUI로 구성
    - **span**
      - 하나의 요청에 사용되는 작업의 단위
      - 64 bit unique ID
    _ **Trace**
      - 트리 구조로 이뤄진 Span 셋
      - 하나의 요청에 대한 같은 Trace ID 발급
    
  * Spring Cloud Sleuth
    - 스프링 부트 애플리케이션을 Zipkin과 연동 (Zipkin에 데이터 전달)
    - 요청 값에 따른 Trace ID, Span ID 부여
    - Trace와 Span Ids를 로그에 추가 가능
      - servlet filter
      - rest template
      - scheduled actions
      - message channels
      - feign client

  * Spring Cloud Sleuth + Zipkin
  * Zipkin 기본 포트 9411번

### Turbine Server
  * 마이크로서비스에 설치된 Hystrix 클라이언트의 스트림을 통합
    - 마이크로서비스에서 생성되는 Hystrix 클라이언트 스트림 메시지를 터빈 서버로 수집
  * Hystrix 클라이언트에서 생성하는 스트림을 시각화
    - Web Dashboard

### Prometheus + Grafana
  * Prometheus
    - Metrics를 수집하고 모니터링 및 알람에 사용되는 오픈소스 애플리케이션
    - 2016년부터 CNCF에서 관리되는 2번쨰 공식 프로젝트
      - Level DB -> Time Series Database(TSDB)
    - Pull방식의 구조와 다양한 Metric Exporter 제공
    - 시계열 DB에 Metrics 저장 -> 조회 가능(Query)

  * Grafana
    - 데이터 시각화, 모니터링 및 분석을 위한 오픈소스 애플리케이션
    - 시계열 데이터를 시각화하기 위한 대시보드 제공

  - 프로메테우스에서 /actuator/prometheus로 데이터 수집을 한다
  - Grafana에서 연동을 통해 시각화를 한다.

###### Prometheus
  * [prometheus.io] -> Download -> 설치
  * prometheus.yml 파일 수정 - target 지정
  * Prometheus 서버 실행 (port: 9090(default))
  * ./prometheus --config.file=prometheus.yml (Linux/Mac | 윈도우는 exe 실행)  

###### Grafana
  * [grafana.com] -> Download -> 설치
  * Grafana 실행
    - http://127.0.0.1:3000
    - ID: admin, PW: admin
  * $./bin/grafana-server web (Linux/mac)
  * windows는 ${grafana.dir}/bin/grafana-server.exe 실행
  * Prometheus - Grafana 연동

# 애플리케이션 배포를 위한 Conatainer 가상화

### Virtualization(가상화)
  * 물리적인 컴퓨터 리소스를 다른 시스템이나 애플리케이션에 사용할 수 있도록 제공
    - 플랫폼 가상화
    - 리소스 가상화
  *  하이퍼바이저(Hypervisor)
    - Virtual Machine Manager(VMM)
    - 다수의 운영체제를 동시에 실행하기 위한 논리적 플랫폼
    - Tpye 1: Native or Bare-metal
    - Type 2: Hosted  
### Container Virtualization
  * OS Virualization
    - Host OS 위에 Guest OS 전체를 가상화
    - VMWare, VirtualBox
    - 자유도가 높으나, 시스템에 부하가 많고 느려짐
  * Container Virtualization
    - Host OS가 가진 리소스를 적게 사용하며, 필요한 프로세스 실행
    - 최소한의 라이브러리와 도구만 포함
    - Container의 생성 속도가 빠름

### Container Image
  * Container 실행에 필요한 설정값
    - **상태값X, Immutable**
  * Image를 가지고 실체화 -> Container

### Dockerfile
  * Docker Image를 생성하기 위한 스크립트 파일
  * 자체 DSL(Domain-Specific Language) 언어 사용 -> 이미지 생성과정 기술

### Docker Desktop
  * [https://www.docker.com/products/docker-desktop]
  * Docker 명령어
  ```sh
    docker info         # docker 정보
    docker images ls     # docker 이미지 리스트
    docker images -a    # (all) 상세정보 모두보기(deafult hides intermediate images)
    docker container ls # docker 컨테이너 리스트

    # 컨테이너 실행
    $ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
    -d # detached mode 흔히 말하는 백그라운드 모드
    -p # 호스트와 컨테이너의 포트를 연결(포워딩)
    -v # 호스트와 컨테이너의 디렉토리를 연결(마운트)
    -e # 컨테이너 내에서 사용할 환경변수 설정
    -name # 컨테이너 이름 설정
    -rm # 프로세스 종료 시 컨테이너 자동 제거
    -it # -i와 -t를 동시에 사용한 것으로 터미널 입력을 위한 옵션
    -link # 컨테이너 연결[컨테이너명:별칭]
    -exec # 이미지 실행
    -logs [컨테이너ID|컨테이머명] # 로그 보기
    # ex)
    docker run ubuntu:16.04
  ```
  * 도커 이미지(도커 허브) : [https://hub.docker.com/]  
### Docker 실행
  ```sh
    # 3306(호스트 포트):3306(컨테이너 포트), --name mysql(이름) mysql:5.7(버전)
    docker run -d -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql mysql:5.7
    docker exec -it mysql bash
  ```
### Dockerfile for Users Microservice
  * 도커 이미지 파일 만들기
  ```Dockerfile
    FROM openjdk:8-jdk-alpine

    VOLUME /tmp # 가상의 디렉토리

    COPY target/user-ws-0.1.jar user-service.jar # 주의:도커파일과 target의 디렉토리가 같아야함 (앞의 파일을 뒤의 파일로 복사)

    ENTRYPOINT ["java",
    "-Djava.security.egd=file:/dev/./urandom",
    "jar",
    "user-service.jar"]
  ```
  ```sh
    docker build -t username/user-service:1.0 #(-t: --tag, 태그)
    docker push username/user-service:1.0
    docker pull username/user-service:1.0
  ```

### Running MircoServices
  * IDEA + Local
  * JAR file + Local
  * Docker + Local
***
  * Docker + AWS EC2
  * Docker + Docker Swarm Mode + AWS EC2
  * Docker(CRI-O) + Kubernetes + AWS EC2 (거의 표준)

### Create Bridge Network
  * Bridge network
    - $ docker network create --driver bridge [브릿지 이름]
  * Host network
    - 네트워크를 호스트로 설정하면 호스트의 네트워크 환경을 그대로 사용
    - 포트 포워딩 없이 내부 어플리케이션 사용
  * None network
    - 네트워크를 사용하지 않음
    - lo 네트워크만 사용, 외부와 단절
  * 불필요한 도커 컨테이너 삭제 
    - docker system prune
  ```sh
    $ docker network create ecommerce-network
    $ docker network ls

    # gateway, subnetmask 설정 (하나의 네트워크에서 컨테이너들을 사용하기 위함)
    docker network create --gateway 172.18.0.1 --subnet 172.18.0.0/16 ecommerce-network
  ```

### Run RabbitMQ (Docker)
  ```sh
    docker run -d --name rabbitmq --network ecommerce-network \
    -p 15672:15672 -p 5672:5672 -p 15671:15671 -p 5671:5671 -p 4369:4369 \
    -e RABBITMQ_DEFAULT_USER=guest \
    -e RABBITMQ_DEFAULT_PASS=guest rabbitmq:management

    # 이미지가 없는 경우 다운로드 및 실행 동시에 (run)
    docker run -d --name rabbitmq --network ecommerce-network -p 15672:15672 -p 5672:5672 -p 15671:15671 -p 5671:5671 -p 4369:4369 -e RABBITMQ_DEFAULT_USER=guest -e RABBITMQ_DEFAULT_PASS=guest rabbitmq:management
  ```




