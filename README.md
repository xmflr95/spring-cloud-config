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
  * 모든 시스템으로 데이터를 실시간으로 전송하여 처리할 수 있는 시스템
  * 데이터가 많아지더라도 확장이 용이한 시스템

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
    
