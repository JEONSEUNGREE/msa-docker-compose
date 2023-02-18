# docker-container_1

MSA_PROJECT Docker
사용한 외부 포트 (호스트 포트에 충돌 나는 경우 docker compose ports 수정 필요)
도커 네트워크 사용으로 내부적으로 충돌 나지않음

1. rabbitmq : 1002, 1001
2. mysql: 1003
3. mysql-product: 1004
4. mysql-order: 1005
5. kafka-1: 3001
6. zipkin: 3011
7. prometheus: 3018
8. grafana: 3017
9. config-service: 10002
10. eureka-service: 8761
11. apigateway-service: 4001
12. nginx: 1221

4단계로 프로젝트 실행)

1. 도커 설치

2. 도커 네트워크 생성
docker network create --gateway 172.18.0.1 --subnet 172.18.0.0/16 ecommerce-network

3. 도커 실행
all 디렉토리에서 docker compose up

4. 프론트 접속 포트 1221

사용한 도커 이미지들-----

1.리액트 - nginx로 배포에 사용
사용 예)
client -> apigateway(해당 서비스 서버 uri 매칭) -> 해당 서버 -> 리턴
server에서 config가 변경되면 client에 변경된 알림이 뜨도록 설정함.
http://localhost:1221/

2.카프카 사용 예)
유저 -> 상품과 주문간의 이벤트 형성 (상호간의 pub, sub), circuitbreak로 resilience4j사용

3.FeignClient 사용 예)
외부 통신 -> feignClient(restTemplate과 유사하다) 유레카에 등록된 서비스간의 통신에 사용됨 (현재는 상품 서버와 주문 서비스간에 사용됨)

4.RestDocs 전체적으로 주요 api 사용시 사용 예)
유저, 주문, 상품

5.SpringConfigSever 사용 예)
전체적인 프로퍼티 깃허브에 생성 -> configServer에서 로드 -> 유저, 주문, 상품 서버 로드시 configServer에서 정보 로드
https://github.com/JEONSEUNGREE/config-server_1

6.zipkin - 요청 온 스레드로 api 상태 확인 가능
사용 예)
http://172.18.0.1:3011/zipkin/

7.rabiit - git config 설정 변경 후 (commit , push) rabbitMq 구독중인(유저, 주문, 상품) 서버에 변경 된 프로퍼티 전달(업데이트)
spring bus 사용 POST로 http://172.18.0.1:4001/user-service/actuator/busrefresh 요청시 순차적으로 변경사항 적용
사용 예)
http://172.18.0.1:1002/
id: admin
pwd: admin

8.prometheus 스프링에서 호출된 api 정보 metrics로 수집 (prometheus.yml로 수집시간, 수집 서버 설정 가능)
사용 예)
http://172.18.0.1:3018 

9.grafana는 prometheus에서 수집한 정보를 도식화한다. prometheus에서 수집한 db를 연결해서 사용 (화면 dashboard는 grafa dashboard에서 커스텀한 대쉬보드 사용)
사용 예)
http://172.18.0.1:3017
대쉬보드
http://172.18.0.1:3017/d/4XuMd2Iiz/kubernetes-cluster-prometheus?orgId=1
id: admin
pwd: admin

프로젝트 서버

10.스프링 CONFIG 서버
깃허브
https://github.com/JEONSEUNGREE/config-server_1/
모든 등록 서버에 설정
http://172.18.0.1:10002/config-local/local
로컬 설정 게이트웨이 서버 (routes 동적으로 사용 시)
http://172.18.0.1:10002/gateway-local/local

11.REST_DOCS_API
유저
http://172.18.0.1:4001/user-service/docs/index.html
주문
http://172.18.0.1:4001/order-service/docs/index.html
상품
http://172.18.0.1:4001/product-service/docs/index.html













<!-- 
9
카프카와 주키퍼 간략 설명
10
프로듀서 : 카프카와 통신하면서 메시지를 보내는 역할
11
컨슈머 : 카프카와 통신하면서 메시지를 가져오는 역할
12
주키퍼 : 컨슈머와 통신, 카프카의 메타데이터 정보를 저장, 카프카의 상태관리 등 목적으로 이용
13
주키퍼는 클러스터가 과반수이상 다운되면 서비스 중지됨
14
ex) 총 3대 -> 2대 다운시 서비스 중지
15
    총 3대 -> 1대 다운시 서비스 유지
16
 -->
