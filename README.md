# 기동명령어 
## 기동시 offset 설정
./standalone.sh -Djboss.server.base.dir=/opt/jboss/standalone -Djboss.socket.binding.port-offset=10000

## 기동시 ip 설정 
./standalone.sh -bmanagement 127.0.0.1

# standalone.xml 에서 인터페이스 부분
interface 
inet-address
<any-address/>

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







