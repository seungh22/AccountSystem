# Spring Boot와 Java를 활용한 계좌 관리 시스템
## 1. 소개
Account System은 사용자와 계좌의 정보를 저장하고 있으며,
외부 시스템에서 거래를 요청할 경우 거래 정보를 받아서 계좌에서 잔액을 거래 금액만큼 줄이거나(결제),
거래 금액만큼 늘리는(결제 취소) 거래 관리 기능을 제공하는 시스템이다.

크게 사용자, 계좌, 거래의 정보를 저장해야 한다.

사용자는 신규 등록, 해지, 중지, 사용자 정보 조회 등의 기능을 제공해야 하지만
최초 버전에서는 빠른 서비스 오픈을 위해 사용자 등록, 해지, 중지 기능은 제공하지 않고 DB로 수기 입력한다.

계좌는 계좌 추가, 해지, 확인 기능을 제공한다. 한 사용자는 최대 10개의 계좌를 가질 수 있으며 그 이상의 계좌는 생성하지 못한다.
계좌 번호는 10자리의 정수로 이루어지며 중복이 불가능하다.
빠르고 안정적인 진행을 위해 계좌번호는 순차 증가하도록 한다.

거래는 잔액 사용, 잔액 사용 취소, 거래 확인 기능을 제공한다.
거래금액을 늘리거나 줄이는 과정에서 여러 쓰레드 혹은 인스턴스에서 같은 계좌에 접근할 경우,
동시성 이슈로 인한 lost update가 발생할 수 있으므로 이 부분을 해결해야 한다.

## 2. 요구사항
- Spring Boot와 Java 사용
- 단위테스트 작성
- H2 DB(Memory DB 모드) 사용
- Spring Data Jpa를 활용해서 DB 접근
- Embedded Redis 활용
- JSON 타입으로 API Request Body와 Response Body 표현
- 각 API들은 별도의 요청과 응답 객체 구조를 가짐
- 요청 실패시 공통된 구조의 응답 사용

## 3. API 명세서
### 1. 계좌 생성
- 파라미터
> 사용자 아이디, 초기 잔액

- 결과
  - 실패
    > 사용자 없는 경우, 사용자 아이디와 계좌 소유주가 다른 경우, 계좌가 이미 해지 상태인 경우, 잔액이 있는 경우
  - 성공
    > 사용자 아이디, 계좌번호, 해지일시를 json 형식으로 응답

>[!warning]
> 계좌 번호는 10자리 정수이며 동일 계좌 번호가 있다면 순차 증가된 번호로 생성한다.
---
### 2. 계좌 해지
- 파라미터
> 사용자 아이디, 계좌 번호

- 결과
  - 실패
    > 사용자 없는 경우, 사용자 아이디와 계좌 소유주가 다른 경우, 계좌가 이미 해지 상태인 경우, 잔액이 있는 경우
  - 성공
    > 사용자 아이디, 계좌번호, 해지일시를 json 형식으로 응답
---
### 3. 계좌 확인
- 파라미터
> 사용자 아이디

- 결과
  - 실패
    > 사용자 없는 경우
  - 성공
    > 계좌번호, 잔액 정보를 json 형식으로 응답
---
### 4. 결제
- 파라미터
> 사용자 아이디, 계좌 번호, 거래 금액

- 결과
  - 실패
    > 사용자 없는 경우, 사용자 아이디와 계좌 소유주가 다른 경우, 계좌가 이미 해지 상태인 경우, 거래금액이 잔액보다 큰 경우, 거래금액이 너무 작거나 큰 경우
  - 성공
    > 계좌번호, 거래결과, 거래ID, 거래금액, 거래일시를 json 형식으로 응답
---
### 5. 결제 취소
- 파라미터
> 거래ID, 계좌번호, 거래금액

- 결과
  - 실패
    > 원거래 금액과 취소 금액이 다른 경우, 트랜잭션이 해당 계좌의 거래가 아닌경우
  - 성공
    > 계좌번호, 거래결과, 거래ID, 취소된 거래금액, 거래일시를 json 형식으로 응답
---
### 6. 결제 내역 확인
- 파라미터
> 거래ID

- 결과
  - 실패
    > 해당 거래ID가 없는 경우
  - 성공
    > 계좌번호, 거래종류(결제, 결제취소), 거래결과, 거래ID, 거래금액, 거래일시

> + 실패한 거래도 확인할 수 있도록 한다.
---
