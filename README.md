# 기동명령어 
## 기동시 offset 설정
```
./standalone.sh -Djboss.server.base.dir=/opt/jboss/standalone -Djboss.socket.binding.port-offset=10000
```
## 기동시 ip 설정 
```
./standalone.sh -bmanagement 127.0.0.1
```

# standalone.xml 에서 인터페이스 부분
```
 interface 
 inet-address
 <any-address/>
```

# jboss-cli.sh

## 서버 재기동
```
[standalone@jboss1:19990 /] :reload
```
## 서비스 포트 변경
```
[standalone@jboss1:19990 /] cd socket-binding-group=standard-sockets
[standalone@jboss1:19990 socket-binding-group=standard-sockets] cd socket-binding=http
[standalone@jboss1:19990 socket-binding=http] :write-attribute(name=port,value=8081)
```
## jboss-cli.sh 특정 xml 설정 확인
```
[disconnected /] embed-server --server-config=exploring-cli.xml
```

## 작업 설명 
```
:read-operation-description(name=test-connection-in-pool)
```
## 특성 설명
```
:read-resource-description
```
## 데이터소스 풀 개수 세팅 
```
[standalone@embedded data-source=ExampleDS] :write-attribute(name=min-pool-size,value=5)
{"outcome" => "success"}
[standalone@embedded data-source=ExampleDS] :write-attribute(name=max-pool-size,value=10)
{"outcome" => "success"}
```
## 데이터소스 풀 개수 확인
```
[standalone@embedded data-source=ExampleDS] :read-attribute(name=min-pool-size)
{
    "outcome" => "success",
    "result" => 5
}
[standalone@embedded data-source=ExampleDS] :read-attribute(name=max-pool-size)
{
    "outcome" => "success",
    "result" => 10
}
```

## 시스템 속성 추가
```
[standalone@embedded data-source=ExampleDS] /system-property=env:add(value=production)
{"outcome" => "success"}
```
## 하위시스템 제거
```
[standalone@embedded data-source=ExampleDS] /subsystem=jsf:remove
{"outcome" => "success"}
```

## 로그 레벨 변경
```
[standalone@jboss1:19990 /] cd /subsystem=logging/console-handler=CONSOLE
[standalone@jboss1:19990 console-handler=CONSOLE] :write-attribute(name=
autoflush  enabled  encoding  filter  filter-spec  formatter  level  named-formatter  target
[standalone@jboss1:19990 console-handler=CONSOLE] :write-attribute(name=level,value=
ALL  CONFIG  DEBUG  ERROR  FATAL  FINE  FINER  FINEST  INFO  OFF  SEVERE  TRACE  WARN  WARNING
[standalone@jboss1:19990 console-handler=CONSOLE] :write-attribute(name=level,value=INFO
{"outcome" => "success"}
[standalone@jboss1:19990 console-handler=CONSOLE] :read-resource
```

# 어플리케이션 배치
##  파일시스템 배포자
```
예시) myapp.war.dodeploy
```
### 사용자 작성
```
.dodeploy
.skipdeploy
```
### 배포 스캐너 작성
```
.deployed
.isdeploying
.failed
.isundeploying
.undeployed
.pending
```

# 도메인 구성

## host-master.xml - IP 주소 세팅
```
    <interfaces>
        <interface name="management">
            <inet-address value="${jboss.bind.address.management:jboss1}"/>
        </interface>
    </interfaces>
```

##  domain.xml 패스워드 세팅
```
            <subsystem xmlns="urn:jboss:domain:messaging-activemq:1.0">
                <server name="default">
                    <cluster password="${jboss.messaging.cluster.password:JBoss@RedHat123}"/>
```

## 도메인 컨트롤러 기동
```
./domain.sh -Djboss.domain.base.dir=/opt/machine1/domain/ --host-config=host-master.xml
```
## 호스트 컨트롤러 기동 
```
./domain.sh -Djboss.domain.base.dir=/opt/machine2/domain/ -Djboss.domain.master.address=jboss1 --host-config=host-slave.xml
```
## 도메인 기동
```
./domain.sh -Djboss.bind.address.management=192.168.0.14
./domain.sh -Djboss.bind.address.management=192.168.0.15 -Djboss.domain.master.address=192.168.0.14
./domain.sh -Djboss.bind.address.management=192.168.0.16 -Djboss.domain.master.address=192.168.0.14
./domain.sh -Djboss.domain.base.dir=/opt/jboss/domain --domain-config=mydomain.xml --host-config=myhost.xml
```
## 도메인 컨트롤러 장애를 대비하여 도메인 설정을 미리 backup
```
./domain.sh --backup -Djboss.domain.master.address=192.168.0.14
```
## 백업된 도메인 설정을 활용하여 도메인 컨트롤러가 없어도 슬레이브 호스트 실행
```
./domain.sh --cached-dc -Djboss.domain.master.address=192.168.0.14
```
## 슬레이브 호스트 컨트롤러에서 backup 컨트롤러 세팅 ( 다중 마스터 구성 )
```
<domain-controller>  
    <remote security-realm="ManagementRealm">  
          <discovery-options>  
              <static-discovery name="primary" host="172.16.81.100" port="9999"/>  
              <static-discovery name="backup" host="172.16.81.101" port="9999"/>  
		  </discovery-options>  
    </remote>  
</domain-controller>
```

## CLI - 서버 추가
```
[domain@jboss1:9990 host=host3] 
[domain@jboss1:9990 server-config] ./server-c:add(auto-start=true,group=main-server-group)
```
## CLI - memory 정보 확인
```
[domain@jboss1:9990 /] cd /host=host3/server=server-three/core-service=platform-mbean/type=
buffer-pool        compilation        memory             memory-pool        runtime
class-loading      garbage-collector  memory-manager     operating-system   threading

[domain@jboss1:9990 /] cd /host=host3/server=server-three/core-service=platform-mbean/type=memory:read-attribute(name=
heap-memory-usage                  object-name                        verbose
non-heap-memory-usage              object-pending-finalization-count

[domain@jboss1:9990 /] /host=host3/server=server-three/core-service=platform-mbean/type=memory:read-attribute(name=heap-memory-usage)
{
    "outcome" => "success",
    "result" => {
        "init" => 1048576000L,
        "used" => 300567688L,
        "committed" => 1005060096L,
        "max" => 1005060096L
    }
}
```

## CLI - server-group 생성/삭제
```
[domain@jboss1:9990 /] /server-group=production:add(profile=ha,socket-binding-group=ha-sockets)
[domain@jboss1:9990 /] /server-group=production:remove
```
## CLI - 구성파일 백업 
```
[domain@jboss1:9990 /] :take-snapshot
{
    "outcome" => "success",
    "result" => "/opt/machine1/domain/configuration/domain_xml_history/snapshot/20191114-000528925domain.xml"
}
```

## LAB - 도메인 컨트롤러 기동 
```
root@jboss1:/opt/jboss-eap-7.0/bin:> sudo -u jboss ./domain.sh -Djboss.domain.base.dir=/opt/domain/ --host-config=host-master.xml
```
## LAB - 호스트 컨트롤러 기동 절차

1. domain.xml 패스워드 수정
```
<cluster password="${jboss.messaging.cluster.password:JBoss@RedHat123}"/>
```
2. host-slave.xml 수정
```
# 호스트 이름 추가
<host name="serverb" xmlns="urn:jboss:domain:4.1">

# 도메인 컨트롤러에서 user-add.sh 로 계정 생성시 생겨난 secret 추가 
    <server-identities>
        <secret value="SkJvc3NAUmVkSGF0MTIz"/>
    </server-identities>

# 도메인 컨트롤러 서버 IP 변경 (ex. jboss1) 과 username 추가
    <domain-controller>
        <remote security-realm="ManagementRealm" username="jbossadm">
            <discovery-options>
                <static-discovery name="primary" protocol="${jboss.domain.master.protocol:remote}" host="${jboss.domain.master.address:jboss1}" port="${jboss.domain.master.port:9999}"/>
            </discovery-options>
        </remote>
    </domain-controller>

# 호스트 IP 변경 (ex. jboss3)
    <interface name="management">
        <inet-address value="${jboss.bind.address.management:jboss3}"/>
    </interface>
    <interface name="public">
        <inet-address value="${jboss.bind.address:jboss3}"/>
    </interface>
```

3. 호스트 컨트롤러 기동
```
root@jboss3:/opt/jboss-eap-7.0/bin:> sudo -u jboss ./domain.sh -Djboss.domain.base.dir=/opt/domain/ --host-config=host-slave.xml
```
# 도메인에서 어플리케이션 배포

## CLI 를 사용하여 어플리케이션 배포

모든 서버 그룹에 배포
> deploy /path/to/example.war --all-server-groups

특정 서버 그룹에 배포
> deploy /path/to/example.war --server-group=Group1,Group2,Group3

배포된 서버 그룹을 추적하여 배포 취소
> undeploy example.war --all-relevant-server-groups

특정 서버 그룹에서 배포 취소
> undeploy example.war --server-group=Group1,Group2

모든 서버 그룹에서 배포 취소하면서 content 저장소 유지
> undeploy example.war --all-relevant-server-groups --keep-content

특정 서버 그룹에서 배포 취소하면서 content 저장소 유지 (일부 그룹에서 배포 취소할 경우 content 유지 필수)
> undeploy example.war --server-group=Group1 --keep-content


## CLI 서버 구성 관리

서버 생성
> /host=servera/server-config=serverA:add(auto-start=true,group=Group1,socket-binding-port-offset=100)

서버 기동
> /host=servera/server-config=serverA:start

서버 중지
> /host=servera/server-config=serverA:stop

서버 그룹의 모든 서버 기동
> /server-group=Group1:start-servers(blocking=true)

서버 그룹의 모든 서버 중지
> /server-group=Group1:stop-servers(blocking=true)

blocking=true 속성은 서버들의 시작/중지 작업이 완료되면 다음 프롬프트로 넘어가도록 합니다.

서버 삭제
> /host=servera/server-config=serverA:remove

호스트 컨트롤러 재기동
> /host=serverb:shutdown(restart=true)


방화벽 해제
> sudo firewall-cmd --zone=public --add-port 8080/tcp --permanent

# 데이터 소스 구성

## JDBC 드라이버 구성

### JDBC 모듈 생성 
```
[disconnected /] module add --name=com.mysql --resources=/home/jboss/mysql-connector-java.jar --dependencies=javax.api,javax.transaction.api
```
### 모듈 제거
```
[disconnected /] module remove --name=com.mysql
```
### JDBC 드라이버 생성
```
/subsystem=datasources/jdbc-driver=mysql:add(driver-module-name=com.mysql,driver-name=mysql,driver-class-name=com.mysql.jdbc.Driver)  
```

### 데이터소스 생성 (도메인 구성 - profile 선택)
```
data-source add --profile=full-ha --name=world --driver-name=mysql --jndi-name=java:jboss/city --connection-url=jdbc:mysql://jboss1:3306//world --user-name=root --password=JBoss@RedHat123
```

### XA 데이터소스 생성 (standalone 구성)
```
xa-data-source add --name=mysqlxa --jndi-name=java:jboss/cityxa --driver-name=mysql --xa-datasource-class=com.mysql.jdbc.jdbc2.optional.MysqlXADataSource --user-name=root --password=JBoss@RedHat123 --xa-datasource-properties=[{"ServerName"=>"jboss1"},{"DatabaseName"=>"world"}]
```

# 로깅

## 비동기 핸들러 구성
FILE_BY_SIZE 라는 파일 핸들러가 사전에 구성된 경우
```
/profile=full-ha/subsystem=logging/async-handler=ASYNC:add(level=INFO,queue-length=1024,overflow-action=BLOCK,subhandlers=[FILE_BY_SIZE])
```
루트 로거에 ASYNC 핸들러 추가
```
/profile=full-ha/subsystem=logging/root-logger=ROOT:add-handler(name=ASYNC)
```

## 새로운 경로 변수 생성
```
/path=custom.log.dir:add(path=/var/log/jboss)
```

## 핸들러 생성
```
/profile=full-ha/subsystem=logging/size-rotating-file-handler=BOOKSTORE_LOG_HANDLER:add(file={"relative-to"=>"custom.log.dir", "path"=>"${jboss.server.name}/bookstore.log"}, enabled=true, append=true, autoflush=true, rotate-size=1m, max-backup-index=5, level=DEBUG)
```

## 새 로거 범주 생성
```
/profile=full-ha/subsystem=logging/logger=javax.sql:add(category=javax.sql, level=ALL, handlers=["BOOKSTORE_LOG_HANDLER"])
```


# 메시징 하위 시스템

```
JMS 하위 모니터링
/profile=full-ha/subsystem=messaging-activemq/server=default/jms-queue=ExpiryQueue:read-attribute(name=message-count)

대기 중인 메시지 나열
/profile=full-ha/subsystem=messaging-activemq/server=default/jms-queue=DLQ:list-messages

pooled-connection-factory 생성
/profile=full-ha/subsystem=messaging-activemq/server=default/pooled-connection-factory=custom:add(connectors=[in-vm], entries=[java:/jms/CustomCF])

관리콘솔에서 JMS QUEUE 생성

JMS Queue 내용 확인
/profile=full-ha/subsystem=messaging-activemq/server=default/jms-queue=TestQueue:read-resource(include-runtime=true)

/profile=full-ha/subsystem=messaging-activemq/server=default/jms-queue=TestQueue:list-messages

메세징 권한 확인
/profile=full-ha/subsystem=messaging-activemq/server=default/security-setting=#/role=guest:read-resource

권한 추가 (remove는 삭제)
/profile=full-ha/subsystem=messaging-activemq/server=default/security-setting=#/role=sender:add(send=true)

```

## 메시지 전달 실패 처리

```
[domain@jboss1:9990 server=default] ./address-setting=#:read-resource
{
    "outcome" => "success",
    "result" => {
        "address-full-policy" => "PAGE",
        "auto-create-jms-queues" => false,
        "auto-delete-jms-queues" => false,
        "dead-letter-address" => "jms.queue.DLQ", (실패한 메시지가 이동)
        "expiry-address" => "jms.queue.ExpiryQueue",
        "expiry-delay" => -1L,
        "last-value-queue" => false,
        "max-delivery-attempts" => 10,  (10번의 시도 후)
        "max-redelivery-delay" => 0L,
        "max-size-bytes" => 10485760L,
        "message-counter-history-day-limit" => 10,
        "page-max-cache-size" => 5,
        "page-size-bytes" => 2097152L,
        "redelivery-delay" => 0L,   (지연 없음)
        "redelivery-multiplier" => 1.0,
        "redistribution-delay" => 1000L,
        "send-to-dla-on-no-route" => false,
        "slow-consumer-check-period" => 5L,
        "slow-consumer-policy" => "NOTIFY",
        "slow-consumer-threshold" => -1L
    }
}
```

## 메시지 큐의 address setting 시 실패 처리 적용
```
[domain@jboss1:9990 server=default] ./address-setting=jms.queue.testQueue:add(redelivery-delay=5000, dead-letter-address=jms.queue.DLQ, max-delivery-attempts=3)
```

## 오래 걸린 메시지를 다른 큐로 이동
```
[domain@jboss1:9990 server=default] ./address-setting=jms.queue.testQueue:add(expiry-delay=1200000, expiry-address=jms.queue.ExpiryQueue)
```

## servera.1 에서 그룹의 모든 인스턴스를 표시하는지 확인 
```
/host=servera/server=servera.1/subsystem=messaging-activemq/server=default/cluster-connection=my-cluster:read-attribute(name=topology)
```


# 보안

## 데이터베이스 보안 도메인 구성
### DB 가 미리 구성되어야 함
```
/subsystem=security/security-domain=db-domain:add(cache-type=default)

/subsystem=security/security-domain=db-domain/authentication=classic:add
(login-modules=[{"code"=>"Database", "flag"=>"required", "module-options"=>[
("dsJndiName"=>"java:jboss/city"), 
("principalsQuery"=>"SELECT password FROM users WHERE username = ?"), 
("rolesQuery"=>"SELECT role, 'Roles' FROM roles WHERE username = ?"), 
("hashAlgorithm"=>"SHA-256"), ("hashEncoding"=>"base64")]}])
```

### 속성 수정 방법
```
/subsystem=security/security-domain=db-domain/authentication=classic/login-module=Database
[standalone@jboss1:9990 login-module=Database] :map-put(name=module-options, key="dsJndiName", value="java:jboss/city")
```

## 어플리케이션 보안 도메인 구성
### jboss-web.xml 안에 security-domain 내용 필요
```
<jboss-web>
    <security-domain>my-domain</security-domain>
</jboss-web>
```
### web.xml 에 security-constraint, security-role, login-config 내용 필요
```
<security-constraint>
  <web-resource-collection>
    <web-resource-name>ALL resources</web-resource-name>
    <url-pattern>/*</url-pattern>
    <http-method>GET</http-method>
    <http-method>POST</http-method>
  </web-resource-collection>
  <auth-constraint>
    <role-name>*</role-name>
  </auth-constraint>
</security-constraint>

<security-role>
  <role-name>*</role-name>
</security-role>

<login-config>
  <auth-method>BASIC</auth-method>
  <realm-name>my-domain</realm-name>
</login-config>
```

## LDAP 보안 구성
```
# docker LDAP 샘플 구성
docker run -d -e DOMAIN=jboss1 -e PASSWORD=admin --name=ldap --net=host -v /var/lib/ldap/data:/var/lib/ldap -v /var/lib/ldap/conf:/etc/ldap/slapd.d siji/openldap:2.4.42
```
```
ldapsearch -x -D "cn=admin,dc=jboss1" -w admin -b "dc=jboss1"
```

### LDAP 보안 도메인 구성
```
/subsystem=security/security-domain=ldap:add(cache-type=default)

/subsystem=security/security-domain=ldap/authentication=classic:add
```
```
/subsystem=security/security-domain=ldap/authentication=classic/login-module=ldap_login:add(code=ldap, flag=required, module-options=[("java.naming.factory.initial"=>"com.sun.jndi.ldap.LdapCtxFactory"), ("java.naming.provider.url"=>"ldap://jboss1:389"), ("java.naming.security.authentication"=>"simple"), ("principalDNPrefix"=>"uid="), ("principalDNSuffix"=>",ou=users,dc=jboss1"), ("rolesCtxDN"=>"ou=Roles,dc=jboss1"), ("uidAttributeID"=>"uid"), ("matchOnUserDN"=>"true"), ("roleAttributeID"=>"cn"), ("roleAttributeIsDN"=>"false")])

```

## JMS 대상 보안

### 메시징 보안 도메인

로컬 어플리케이션에 대한 인증을 활성화하려면 override-in-vm-security 특성을 false 로 변경
```
/profile=full-ha/subsystem=messaging-activemq/server=default:write-attribute(name=override-in-vm-security, value=false)
```

### 역할 확인

```
[domain@jboss1:9990 server=default] /profile=full-ha/subsystem=messaging-activemq/server=default/security-setting=#/role=guest:read-resource
{
    "outcome" => "success",
    "result" => {
        "consume" => true,
        "create-durable-queue" => false,
        "create-non-durable-queue" => true,
        "delete-durable-queue" => false,
        "delete-non-durable-queue" => true,
        "manage" => false,
        "send" => true
    }
} 
```

### 토픽 or 큐 보안

jms-topic 생성
```
/profile=full-ha/subsystem=messaging-activemq/server=default/security-setting=jms.topic.StockQuotes.#:add
```
jms-topic 에 대한 send(전송), consume(사용) 권한을 publisherAndConsumer 역할에 부여함.
```
/profile=full-ha/subsystem=messaging-activemq/server=default/security-setting=jms.topic.StockQuotes.#/role=publisherAndConsumer:add(send=true,consume=true)
```

## 암호 자격 증명 모음 구성

### 키스토어 생성
```
keytool -genseckey -alias vault -keyalg AES -storetype jceks -keysize 128 -keystore /opt/domain/vault.keystore
```
### 아래 파일에서 의존성 추가
/opt/jboss-eap-7.0/modules/system/layers/base/org/picketbox/main/module.xml
```
    <dependencies>
        <module name="sun.jdk"/>
```

### vault.sh 실행
```
jboss@jboss1:~:> /opt/jboss-eap-7.0/bin/vault.sh --keystore /opt/domain/vault.keystore --keystore-password password --alias vault --vault-block valut --attribute password --sec-attr JBoss@RedHat123 --enc-dir /opt/domain/ --iteration 50 --salt 12345678
WARNING JBOSS_HOME may be pointing to a different installation - unpredictable results may occur.

=========================================================================

  JBoss Vault

  JBOSS_HOME: /opt/jboss-eap-7.0

  JAVA: java

=========================================================================

Nov 19, 2019 12:12:26 AM org.picketbox.plugins.vault.PicketBoxSecurityVault init
INFO: PBOX00361: Default Security Vault Implementation Initialized and Ready
WFLYSEC0047: Secured attribute value has been stored in Vault.
Please make note of the following:
********************************************
Vault Block:valut
Attribute Name:password
Configuration should be done as follows:
VAULT::valut::password::1
********************************************
WFLYSEC0048: Vault Configuration in WildFly configuration file:
********************************************
...
</extensions>
<vault>
  <vault-option name="KEYSTORE_URL" value="/opt/domain/vault.keystore"/>
  <vault-option name="KEYSTORE_PASSWORD" value="MASK-31x/z0Xn83H4JaL0h5eK/N"/>
  <vault-option name="KEYSTORE_ALIAS" value="vault"/>
  <vault-option name="SALT" value="12345678"/>
  <vault-option name="ITERATION_COUNT" value="50"/>
  <vault-option name="ENC_FILE_DIR" value="/opt/domain/"/>
</vault><management> ...
```

### messaging-acrivemq 하위 시스템의 기본 보안 도메인: other
```
[domain@jboss1:9990 /] /profile=full-ha/subsystem=messaging-activemq/server=default:read-attribute(name=security-domain)
{
    "outcome" => "success",
    "result" => "other"
}
```

# JVM 구성
## server 개별 설정
```
/host=serverb/server-config=serverb.2/jvm=serverb.2-jvm:add(heap-size=1200m, max-heap-size=1200m, jvm-options=["-XX:+AggressiveOpts"])
```
WAS 재기동
```
/host=serverb/server-config=serverb.2:restart(blocking=true)
```

## host 에서 default JVM 설정 변경
```
/host=servera/jvm=default:write-attribute(name=heap-size,value=512m)
/host=servera/jvm=default:write-attribute(name=max-heap-size,value=512m)
```

## 그룹 범위 JVM 구성
```
/server-group=Group1/jvm=group1-jvm:add(heap-size=512m, max-heap-size=1024m, jvm-options=["-server"])
```

# 웹 하위 시스템 구성

## Root Context 변경
### 기본 Root Application / 위치 제거
```
/subsystem=undertow/server=default-server/host=default-host/location=\/:remove

reload
```

### sample.war 어플리케이션에 / 위치 매핑
```
/subsystem=undertow/server=default-server/host=default-host:write-attribute

reload
```

## 버퍼 캐시 
사이즈 변경
```
/subsystem=undertow/buffer-cache=default:write-attribute(name=buffer-size, value=2048)
```
## 서블릿 컨테이너 구성 정보
```
[standalone@jboss1:9990 /] /subsystem=undertow/servlet-container=default:read-resource
{
    "outcome" => "success",
    "result" => {
        "allow-non-standard-wrappers" => false,
        "default-buffer-cache" => "default",
        "default-encoding" => undefined,
        "default-session-timeout" => 30,   (세션 타임아웃 30분)
        "directory-listing" => undefined,
        "disable-caching-for-secured-pages" => true,
        "eager-filter-initialization" => false,
        "ignore-flush" => false,
        "max-sessions" => undefined,  (최대 세션 개수)
        "proactive-authentication" => true,
        "session-id-length" => 30,    (세션ID 문자열 개수)
        "stack-trace-on-error" => "local-only",
        "use-listener-encoding" => false,
        "mime-mapping" => undefined,
        "setting" => {
            "jsp" => undefined,
            "websockets" => undefined
        },
        "welcome-file" => undefined
    },
    "response-headers" => {"process-state" => "reload-required"}
}
```

## 파일 핸들러
생성
```
/subsystem=undertow/configuration=handler/file=photos:add(path=/var/photos, directory-listing=true)
```
가상호스트에 첨부
```
/subsystem=undertow/server=default-server/host=default-host/location=photos:add(handler=photos)
```

## 필터
추가
```
/subsystem=undertow/configuration=filter/gzip=gzip:add
```

## HTTPS 서비스 구성
키스토어 파일 생성
```
keytool -genkeypair -alias appserver -storetype jks -keyalg RSA -keysize 2048 -keystore identity.jks
```
키스토어 인증서 확인
```
keytool -list -v -keystore ./identity.jks
```
HTTPS 용도의 Realm 구성
```
/core-service=management/security-realm=HTTPSRealm:add

/core-service=management/security-realm=HTTPSRealm/server-identity=ssl:add(keystore-path=/opt/identity.jks, keystore-password=changeit, alias=appserver)

reload
```
HTTPS 리스너 구성
```
/subsystem=undertow/server=default-server/https-listener=https:add(socket-binding=https, security-realm=HTTPSRealm)
```

## AJP 활성화
```
/subsystem=undertow/server=default-server/ajp-listener=ajp:add(socket-binding=ajp)
```

## HTTP 리스너 조정
최대 연결 수 200 
```
/subsystem=undertow/server=default-server/http-listener=default:write-attribute(name=max-connections, value=200)
```

## 도메인 구성에서 HTTPS 구성
SSL 연결 구성 만들기 (host설정)
```
/host=servera/core-service=management/security-realm=HTTPSRealm:add

/host=serverb/core-service=management/security-realm=HTTPSRealm:add

/host=servera/core-service=management/security-realm=HTTPSRealm/server-identity=ssl:add(keystore-path=/opt/keystore/identity.jks, keystore-password=changeit, alias=appserver)

/host=serverb/core-service=management/security-realm=HTTPSRealm/server-identity=ssl:add(keystore-path=/opt/keystore/identity.jks, keystore-password=changeit, alias=appserver)

/host=servera:reload

/host=serverb:reload
```

HTTPS 리스너 구성 (profile설정)
```
/profile=full-ha/subsystem=undertow/server=default-server/https-listener=https:add(socket-binding=https, security-realm=HTTPSRealm)
```

HTTPS 리스너 조정 (profile설정)
```
/profile=full-ha/subsystem=undertow/server=default-server/https-listener=https:write-attribute(name=max-connections, value=200)
```

# 클러스터된 어플리케이션 배포

## standalone-full-ha.xml 로 2대 인스턴스 구성
```
/opt/jboss-eap-7.0/bin/standalone.sh -Djboss.server.base.dir=/opt/jgroups-cluster1/ -Djboss.node.name=jgroups-cluster1 -c standalone-full-ha.xml

/opt/jboss-eap-7.0/bin/standalone.sh -Djboss.server.base.dir=/opt/jgroups-cluster2/ -Djboss.socket.binding.port-offset=100 -Djboss.node.name=jgroups-cluster2 -c standalone-full-ha.xml
```

## TCP 7600,7700 으로 통신 변경(기본 udp 45688)
tcpping 프로토콜 생성
```
batch
/subsystem=jgroups/stack=tcpping:add
/subsystem=jgroups/stack=tcpping:add-protocol(type=TCPPING)
/subsystem=jgroups/stack=tcpping/transport=TRANSPORT:add(socket-binding=jgroups-tcp, type=TCP)
run-batch

batch
/subsystem=jgroups/stack=tcpping/protocol=TCPPING/property=initial_hosts:add(value="jboss1[7600],jboss1[7700]")
/subsystem=jgroups/stack=tcpping/protocol=TCPPING/property=port_range:add(value=10)
/subsystem=jgroups/stack=tcpping:add-protocol(type=MERGE2)
/subsystem=jgroups/stack=tcpping:add-protocol(socket-binding=jgroups-tcp-fd, type=FD_SOCK)
/subsystem=jgroups/stack=tcpping:add-protocol(type=FD)
/subsystem=jgroups/stack=tcpping:add-protocol(type=VERIFY_SUSPECT)
/subsystem=jgroups/stack=tcpping:add-protocol(type=BARRIER)
/subsystem=jgroups/stack=tcpping:add-protocol(type=pbcast.NAKACK)
/subsystem=jgroups/stack=tcpping:add-protocol(type=UNICAST2)
/subsystem=jgroups/stack=tcpping:add-protocol(type=pbcast.STABLE)
/subsystem=jgroups/stack=tcpping:add-protocol(type=pbcast.GMS)
/subsystem=jgroups/stack=tcpping:add-protocol(type=UFC)
/subsystem=jgroups/stack=tcpping:add-protocol(type=MFC)
/subsystem=jgroups/stack=tcpping:add-protocol(type=FRAG2)
/subsystem=jgroups/stack=tcpping:add-protocol(type=RSVP)
/subsystem=jgroups/channel=ee:write-attribute(name=stack, value=tcpping)
run-batch
:reload
```

## 로드 밸런서 구성 (별도의 서버)
```
리버스 프록시 핸들러 생성
/subsystem=undertow/configuration=handler/reverse-proxy=cluster-handler:add

백엔드 서버 구성 server1 host/port 추가
/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=remote-server1:add(host=jboss1, port=8009)

리버스 프록시 핸들러에 server1 및 path 추가
/subsystem=undertow/configuration=handler/reverse-proxy=cluster-handler/host=server1:add(outbound-socket-binding=remote-server1, scheme=ajp, instance-id=server1, path=/cluster)

백엔드 서버 구성 server2 host/port 추가
/socket-binding-group=standard-sockets/remote-destination-outbound-socket-binding=remote-server2:add(host=jboss1, port=8109)

리버스 프록시 핸들러에 server2 및 path 추가
/subsystem=undertow/configuration=handler/reverse-proxy=cluster-handler/host=server2:add(outbound-socket-binding=remote-server2, scheme=ajp, instance-id=server2, path=/cluster)

리버스 프록시 핸들러에 관리할 path 경로 추가
/subsystem=undertow/server=default-server/host=default-host/location=\/cluster:add(handler=cluster-handler)
```


## 도메인 + 로드밸런서
로드밸런서 기동
```
/opt/jboss-eap-7.0/bin/standalone.sh -Djboss.server.base.dir=/opt/lb2/ -Djboss.bind.address=jboss1 -Djboss.socket.binding.port-offset=1000 -c standalone-ha.xml
```
도메인 컨트롤러에서 광고 비활성화
```
/profile=full-ha/subsystem=modcluster/mod-cluster-config=configuration:write-attribute(name=advertise, value=false)
```
로드 밸런서를 프록시 구성
```
/socket-binding-group=full-ha-sockets/remote-destination-outbound-socket-binding=lb:add(host=jboss1, port=9080)
```
프록시를 mod_cluster_config 구성에 추가
```
/profile=full-ha/subsystem=modcluster/mod-cluster-config=configuration:list-add(name=proxies, value=lb)
```
마스터 리로드
```
reload --host=master
```
로드 밸런서에서 속성 추가
```
/subsystem=modcluster/mod-cluster-config=configuration:write-attribute(name=advertise-security-key, value=redhat)
```
undertow - modcluster 필터 구성
```
/subsystem=undertow/configuration=filter/mod-cluster=modcluster:add(management-socket-binding=http, advertise-socket-binding=modcluster, security-key=redhat)
```
modcluster 필터를 undertow default-server 에 바인딩
```
/subsystem=undertow/server=default-server/host=default-host/filter-ref=modcluster:add

:reload
```


