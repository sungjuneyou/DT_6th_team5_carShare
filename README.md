# DT_6th_team5_carShare (자동차 공유 서비스)

5팀 자동차 공유 서비스 CNA개발 실습을 위한 프로젝트

# 구현 Repository
 1. 접수관리 : https://github.com/YoungDukGe1000Won/carShareOrder.git
 1. 결제관리 : https://github.com/YoungDukGe1000Won/carSharePayment.git
 1. 배송관리 : https://github.com/YoungDukGe1000Won/carShareDelivery.git
 1. 고객페이지 : https://github.com/YoungDukGe1000Won/carShareStatusview.git
 1. 게이트웨이 : https://github.com/byounghoonmoon/carShareGateway.git


# Table of contents

- [서비스 시나리오](#서비스-시나리오)
  - [시나리오 테스트결과](#시나리오-테스트결과)
- [분석/설계](#분석설계)
- [구현](#구현)
  - [DDD 의 적용](#ddd-의-적용)
  - [Gateway 적용](#Gateway-적용)
  - [폴리글랏 퍼시스턴스](#폴리글랏-퍼시스턴스)
  - [동기식 호출 과 Fallback 처리](#동기식-호출-과-Fallback-처리)
  - [비동기식 호출 과 Eventual Consistency](#비동기식-호출-과-Eventual-Consistency)
- [운영](#운영)
  - [CI/CD 설정](#cicd설정)
  - [서킷 브레이킹 / 장애격리](#서킷-브레이킹-/-장애격리)
  - [오토스케일 아웃](#오토스케일-아웃)
  - [무정지 재배포](#무정지-재배포)
  

# 서비스 시나리오

## 기능적 요구사항
1. 고객이 공유차를 선택하여 렌탈한다.
1. 고객이 결제하여 접수한다.
1. 업체가 공유차를 고객위치로 가져다놓는다.
1. 고객이 렌탈을 취소할 수 있다.
1. 렌탈이 취소되면 배송이 취소된다.
1. 고객이 자신의 렌탈 정보를 조회한다.

## 비기능적 요구사항
1. 트랜잭션
    1. 결제가 되지 않은 주문건은 아예 접수가 성립되지 않아야 한다(Sync 호출)
1. 장애격리
    1. 배송관리 기능이 수행되지 않더라도 접수는 정상적으로 처리 가능하다(Async(event-driven), Eventual Consistency)
    1. 접수시스템이 과중되면 사용자를 잠시동안 받지 않고 결게를 잠시후에 하도록 유도한다(Circuit breaker, fallback)
1. 성능
    1. 고객이 본인의 렌탈 상태 및 이력을 접수시스템에서 확인할 수 있어야 한다(CQRS)


# 분석/설계

## AS-IS 조직 (Horizontally-Aligned)
![79684144-2a893200-826a-11ea-9a01-79927d3a0107](https://user-images.githubusercontent.com/42608068/96371393-84789f00-119c-11eb-80d9-ffbcab38ff84.png)


## TO-BE 조직 (Vertically-Aligned)



### 이벤트스토밍
* MSAEz 로 모델링한 이벤트스토밍 결과:  
![제목 없음](https://user-images.githubusercontent.com/42608068/96534281-5abd9600-12ca-11eb-88b9-37c11705aae0.png)

### 이벤트 도출 
![제목 없음1](https://user-images.githubusercontent.com/42608068/96401757-3650b380-120f-11eb-87dc-764e34ae453c.png)

### 부적격 이벤트 탈락
![제목 없음](https://user-images.githubusercontent.com/42608068/96401697-0e615000-120f-11eb-982d-2b57c7e2d692.png)

### 액터, 커맨드 부착하여 읽기 좋게
![제목 없음2](https://user-images.githubusercontent.com/42608068/96401865-7ca61280-120f-11eb-83d6-cb0f2970cb37.png)

### 어그리게잇으로 묶기
![제목 없음3](https://user-images.githubusercontent.com/42608068/96402579-423d7500-1211-11eb-9784-c1f7e1c4b2fb.png)

### 바운디드 컨텍스트로 묶기
![제목 없음4](https://user-images.githubusercontent.com/42608068/96402789-b9730900-1211-11eb-84a1-43062ff258fe.png)

### 도메인분리


### 폴리시 부착 


### 폴리시의 이동과 컨텍스트 매핑 (점선은 Pub/Sub, 실선은 Req/Resp)


### 완성된 모형


### 완성본에 대한 기능적/비기능적 요구사항을 커버하는지 검증