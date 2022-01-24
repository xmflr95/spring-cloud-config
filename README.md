# MSA

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


