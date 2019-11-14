# 기동명령어 
## 기동시 offset 설정
./standalone.sh -Djboss.server.base.dir=/opt/jboss/standalone -Djboss.socket.binding.port-offset=10000

## 기동시 ip 설정 
./standalone.sh -bmanagement 127.0.0.1


# standalone.xml 에서 인터페이스 부분
```
interface 
inet-address
<any-address/>
```

# jboss-cli.sh

## 서버 재기동
[standalone@jboss1:19990 /] :reload

## 서비스 포트 변경
[standalone@jboss1:19990 /] cd socket-binding-group=standard-sockets
[standalone@jboss1:19990 socket-binding-group=standard-sockets] cd socket-binding=http
[standalone@jboss1:19990 socket-binding=http] :write-attribute(name=port,value=8081)

## jboss-cli.sh 특정 xml 설정 확인
[disconnected /] embed-server --server-config=exploring-cli.xml


## 작업 설명 
:read-operation-description(name=test-connection-in-pool)

## 특성 설명
:read-resource-description

## 데이터소스 풀 개수 세팅 
[standalone@embedded data-source=ExampleDS] :write-attribute(name=min-pool-size,value=5)
{"outcome" => "success"}
[standalone@embedded data-source=ExampleDS] :write-attribute(name=max-pool-size,value=10)
{"outcome" => "success"}

## 데이터소스 풀 개수 확인
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


## 시스템 속성 추가
[standalone@embedded data-source=ExampleDS] /system-property=env:add(value=production)
{"outcome" => "success"}

## 하위시스템 제거
[standalone@embedded data-source=ExampleDS] /subsystem=jsf:remove
{"outcome" => "success"}


## 로그 레벨 변경
[standalone@jboss1:19990 /] cd /subsystem=logging/console-handler=CONSOLE
[standalone@jboss1:19990 console-handler=CONSOLE] :write-attribute(name=
autoflush  enabled  encoding  filter  filter-spec  formatter  level  named-formatter  target
[standalone@jboss1:19990 console-handler=CONSOLE] :write-attribute(name=level,value=
ALL  CONFIG  DEBUG  ERROR  FATAL  FINE  FINER  FINEST  INFO  OFF  SEVERE  TRACE  WARN  WARNING
[standalone@jboss1:19990 console-handler=CONSOLE] :write-attribute(name=level,value=INFO
{"outcome" => "success"}
[standalone@jboss1:19990 console-handler=CONSOLE] :read-resource


# 어플리케이션 배치
##  파일시스템 배포자
예시) myapp.war.dodeploy
### 사용자 작성
.dodeploy
.skipdeploy
### 배포 스캐너 작성
.deployed
.isdeploying
.failed
.isundeploying
.undeployed
.pending




# 도메인 구성

## host-master.xml - IP 주소 세팅

>    <interfaces>
>        <interface name="management">
>            <inet-address value="${jboss.bind.address.management:jboss1}"/>
>        </interface>
>    </interfaces>

##  domain.xml 패스워드 세팅

>            <subsystem xmlns="urn:jboss:domain:messaging-activemq:1.0">
>                <server name="default">
>                    <cluster password="${jboss.messaging.cluster.password:JBoss@RedHat123}"/>

## 도메인 컨트롤러 기동
./domain.sh -Djboss.domain.base.dir=/opt/machine1/domain/ --host-config=host-master.xml

## 호스트 컨트롤러 기동 
./domain.sh -Djboss.domain.base.dir=/opt/machine2/domain/ -Djboss.domain.master.address=jboss1 --host-config=host-slave.xml

## 도메인 기동
./domain.sh -Djboss.bind.address.management=192.168.0.14
./domain.sh -Djboss.bind.address.management=192.168.0.15 -Djboss.domain.master.address=192.168.0.14
./domain.sh -Djboss.bind.address.management=192.168.0.16 -Djboss.domain.master.address=192.168.0.14
./domain.sh -Djboss.domain.base.dir=/opt/jboss/domain --domain-config=mydomain.xml --host-config=myhost.xml

## 도메인 컨트롤러 장애를 대비하여 도메인 설정을 미리 backup
./domain.sh --backup -Djboss.domain.master.address=192.168.0.14

## 백업된 도메인 설정을 활용하여 도메인 컨트롤러가 없어도 슬레이브 호스트 실행
./domain.sh --cached-dc -Djboss.domain.master.address=192.168.0.14

## 슬레이브 호스트 컨트롤러에서 backup 컨트롤러 세팅 ( 다중 마스터 구성 )
><domain-controller>  
>    <remote security-realm="ManagementRealm">  
>          <discovery-options>  
>              <static-discovery name="primary" host="172.16.81.100" port="9999"/>  
>              <static-discovery name="backup" host="172.16.81.101" port="9999"/>  
>          </discovery-options>  
>    </remote>  
></domain-controller>

## CLI - 서버 추가 
[domain@jboss1:9990 host=host3] 
[domain@jboss1:9990 server-config] ./server-c:add(auto-start=true,group=main-server-group)

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


