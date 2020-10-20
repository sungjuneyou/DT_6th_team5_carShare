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



## 이벤트스토밍
* MSAEz 로 모델링한 이벤트스토밍 결과:  
![image](https://user-images.githubusercontent.com/42608068/96539757-c9085580-12d6-11eb-8721-8bb7e0601d53.png)

### 이벤트 도출 
![제목 없음1](https://user-images.githubusercontent.com/42608068/96541160-60bb7300-12da-11eb-8eda-4beb637fa24f.png)

### 부적격 이벤트 탈락
![제목 없음2](https://user-images.githubusercontent.com/42608068/96541195-6fa22580-12da-11eb-94c0-9efb0947e5aa.png)

### 액터, 커맨드 부착하여 읽기 좋게
![제목 없음3](https://user-images.githubusercontent.com/42608068/96541203-77fa6080-12da-11eb-8a8a-50a018a72961.png)

### 어그리게잇으로 묶기
![제목 없음3](https://user-images.githubusercontent.com/42608068/96402579-423d7500-1211-11eb-9784-c1f7e1c4b2fb.png)

### 바운디드 컨텍스트로 묶기
![제목 없음4](https://user-images.githubusercontent.com/42608068/96541221-86487c80-12da-11eb-96c9-705fd7957d01.png)

### 도메인분리
![제목 없음5](https://user-images.githubusercontent.com/42608068/96541235-919ba800-12da-11eb-8c49-84655f2ca88e.png)

### 폴리시 부착 
![제목 없음6](https://user-images.githubusercontent.com/42608068/96541251-99f3e300-12da-11eb-99f9-8a9027a7b855.png)

### 폴리시의 이동과 컨텍스트 매핑 (점선은 Pub/Sub, 실선은 Req/Resp)
![제목 없음7](https://user-images.githubusercontent.com/42608068/96541279-a11af100-12da-11eb-9d0d-3cf209f7216b.png)

### 완성된 모형
![제목 없음11](https://user-images.githubusercontent.com/42608068/96543764-0b826000-12e0-11eb-9296-112459a6027b.png)

### 완성본에 대한 기능적 요구사항을 커버하는지 검증
![제목 없음12](https://user-images.githubusercontent.com/42608068/96543922-72077e00-12e0-11eb-91bf-ae6aaf5e8fbb.png)
    
    - 고객이 공유차를 선택하여 렌탈한다 (ok)
    - 고객이 결제하여 접수한다 (ok)
    - 업체가 공유차를 고객위치로 가져다놓는다 (ok)

![제목 없음13](https://user-images.githubusercontent.com/42608068/96543936-79c72280-12e0-11eb-98a2-0c67478f6926.png)

    - 고객이 주문을 취소할 수 있다 (ok)
    - 렌탈이 취소되면 배송이 취소된다 (ok)

![제목 없음14](https://user-images.githubusercontent.com/42608068/96543997-9cf1d200-12e0-11eb-9a71-9aa743f7de44.png)
   
    - 고객이 자신의 렌탈 정보를 조회한다 (ok)
    
### 완성본에 대한 비기능적 요구사항을 커버하는지 검증
![image](https://user-images.githubusercontent.com/487999/79684184-5c9a9400-826a-11ea-8d87-2ed1e44f4562.png)

    1. 트랜잭션
    - 고객의 주문에 따라 결제가 진행된다(결제가 정상적으로 완료되지 않으면 주문이 되지 않는다) > Sync
    - 고객의 결제 완료에 따라 배송이 진행된다 > Async
    2. 장애격리
    - 배송 서비스에 장애가 발생하더라도 주문 및 결제는 정상적으로 처리 가능하다 > Async(event driven)
    - 서킷 브레이킹 프레임워크 > istio-injection + DestinationRule
    3. 성능
    - 고객은 본인의 상태 정보를 확인할 수 있다 > CQRS

## 헥사고날 아키텍처 다이어그램 도출
![제목없음21](https://user-images.githubusercontent.com/42608068/96549943-260e0680-12eb-11eb-8119-394cb324883d.png)

    - Chris Richardson, MSA Patterns 참고하여 Inbound adaptor와 Outbound adaptor를 구분함
    - 호출관계에서 PubSub 과 Req/Resp 를 구분함
    - 서브 도메인과 바운디드 컨텍스트의 분리:  각 팀의 KPI 별로 아래와 같이 관심 구현 스토리를 나눠가짐

   
# 구현

## 시나리오 테스트결과

| 기능 | 이벤트 Payload |
|---|:---:|
| 1.관리자가 병원 정보( 병원이름, 예약일, 가능인원수)를 등록한다. |![image](https://user-images.githubusercontent.com/25805562/91837451-3577b880-ec87-11ea-88b1-dc4e9d74790d.png)|
| 2.고객이 건강검진을 예약을 요청한다. </br>3.해당 병원의 검진가능 인원이 감소한다. (Sync)</br>4.예약 완료로 변경된다. (Async)</br>5.예약관리의 해당 내역의 상태가 등록된다. |![image](https://user-images.githubusercontent.com/25805562/91837806-7a035400-ec87-11ea-8966-09403bd5e7eb.png)|
| 6.고객이 건강검진 예약을 취소한다.</br>7.취소 시, 병원의 검진가능 인원이 증가한다. (Async)</br>8.예약관리의 해당 내역의 상태가 예약 취소로 변경된다. | ![image](https://user-images.githubusercontent.com/25805562/91837990-c2227680-ec87-11ea-9fb1-530410922532.png) |
| 9.관리자가 병원 정보를 삭제한다.</br>10.해당 병원에 예약한 예약자의 상태를 예약 강제 취소 변경한다. (Async)</br>11.예약관리의 해당 내역의 상태가 예약 강제 취소로 변경된다. | ![image](https://user-images.githubusercontent.com/25805562/91838119-f007bb00-ec87-11ea-9edd-38d9963f9ee0.png) | 
| 12.건강검진 예약내역 상태를 조회한다.| ![image](https://user-images.githubusercontent.com/25805562/91838415-6ad0d600-ec88-11ea-9df8-1c6895fe6d75.png) |
