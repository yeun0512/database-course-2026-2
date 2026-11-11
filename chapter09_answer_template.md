# Chapter 09 확장 실습 답안 템플릿

> **과제:** 트랜잭션으로 데이터 정합성 지키기  
> **사용 방법:** 이 파일을 내려받아 본인의 GitHub 저장소에 `chapter09_answer.md`라는 이름으로 저장한 뒤 실습하면서 바로 작성합니다.  
> **제출 방법:** LMS에는 파일을 직접 업로드하지 않고, **본인 GitHub 저장소의 `chapter09_answer.md` 파일 URL**을 제출합니다.

---

## 제출 전 주의

이 파일과 캡처 화면에는 실제 비밀번호, 전체 DB 접속 URL, API Key, 개인정보를 기록하지 않습니다.

```text
GitHub 계정 또는 별칭: yeun0512
과제 작성일: 2026.11.11
사용한 AI 도구: GPT
```

---

# 1. 시작 환경과 Chapter 07·08 기준 상태 확인

다음을 실행하거나 Chapter 09의 `01_transaction_lab_schema.sql` 사전 검사를 확인합니다.

```sql
SELECT current_database();
SELECT current_user;
SELECT current_schema();
SHOW search_path;
SHOW transaction_read_only;
```

| 확인 항목 | 실제 결과 | 의미 |
| --- | --- | --- |
| `current_database()` | ai_database_book | 현재 ai_database_book 데이터베이스에 연결되어 있다. |
| `current_user` | postgres | 현재 사용자가 postgres이다. |
| `current_schema()` | public | 기본 스키마가 public이다. |
| `search_path` | public, "$user" | 테이블을 찾을 때 먼저 public을 확인하고, 그다음 사용자 이름과 같은 스키마를 확인한다. |
| `transaction_read_only` | off | 읽기 전용 모드가 아니고, 데이터 변경이 가능하다. |

Chapter 07·08 기준값:

```text
students = 3
instructors = 2
courses = 3
enrollments = 5
전체 recorded_amount = 590000
활성 = 3 / 340000
취소 제외 = 4 / 440000
```

### 기준 상태가 다르면 Chapter 09를 계속 진행하면 안 되는 이유

```text
기준값이 예상과 다르면 데이터가 추가, 변경되었거나 정상적이지 않은 상태일 수 있다. 따라서, 이후 단계의 조회 결과가 잘못 나타나거나, 데이터가 중복되거나 누락될 수 있다. 따라서, 기준 상태와 동일하게 맞추고 진행해야 한다.
특히, chapter 09의 좌석 신청, 결제는 chapter 07의 데이터를 부모 데이터로 참조하기 때문에, 다른 결과가 나타날 수 있고, 이 때 실패가 트랜직션 문제인지 이전의 상태 문제인지 구분하기 어려워지기 때문이다. ```
---

# 2. `transaction_lab` 스키마 생성

실행 파일:

```text
code/chapter09/01_transaction_lab_schema.sql
```

## 2-1. 생성 전 예상

```text
생성될 스키마:transaction_lab
생성될 테이블 3개:transaction_lab.course_inventory, transaction_lab.enrollments. transaction_lab.payments

course_inventory 한 행의 의미:특정 강의의 정원과 남은 좌석 정보
enrollments 한 행의 의미: 학생 한 명의 강의 신청 한 건
payments 한 행의 의미: 강의 신청과 연결된 결제 한 건
```

## 2-2. 생성 결과

```text
통과 메시지:Chapter 09 transaction lab schema validation passed
```

기대 메시지:

```text
Chapter 09 transaction lab schema validation passed
```

### Chapter 07·08의 `course_project`와 별도 `transaction_lab`을 사용하는 이유

```text
좌석 변경, 신청 생성, 결제 생성 등의 트랜젝션을 할 때, 기존 Chapter 07·08의 course_project 데이터에 영향을 주지 않고 진행하고자 하기 때문이다. 이를 통해 앞 장의 데이터를 보호하고, 트랜잭션 실험만 독립적으로 초기화할 수 있다. 따라서, 같은 학생, 강의 id를 사용하면서도 변경 결과가 조건에 맞는지 검증한 뒤 COMMIT할 수 있게 되어 안전하고, 필요하면 ROLLBACK할 수 있게 된다. 
```

### 증거 화면

권장 경로:

```text
assignments/chapter09/images/step02_schema.png
```

<img width="1163" height="956" alt="image" src="https://github.com/user-attachments/assets/46688267-e8d6-4f9f-8143-be435af382de" />

---

# 3. 초기 좌석과 기준 데이터 입력

실행 파일:

```text
code/chapter09/02_transaction_lab_seed.sql
```

## 3-1. 실행 전 예상

```text
course 301 remaining_seats 예상: 2
course 302 remaining_seats 예상: 1
course 303 remaining_seats 예상: 1
lab enrollments 예상 행 수: 0
payments 예상 행 수: 0
```

## 3-2. 실제 결과

```text
course 301 remaining_seats: 2
course 302 remaining_seats: 1
course 303 remaining_seats: 1
lab enrollments 행 수:0
payments 행 수:0
통과 메시지:Chapter 09 transaction lab seed validation passed
```

기대 초기 상태:

```text
course 301 / 302 / 303 remaining_seats = 3 / 0 / 0
lab enrollments = 0
payments = 0
```

### 예상과 실제가 다른 경우 원인

```text
실행 전 예상했던 것과 실제 결과는 일치하였다. 그러나, 기대 초기 상태와는 불일치하였다. inventroy 자체가 3개의 seat 인 것은 맞는데, 각각의 course 에 2, 1, 1 개의 remaining seats가 있는 결과가 나왔다.
INSERT INTO transaction_lab.course_inventory (
    course_id,
    capacity,
    remaining_seats
)
VALUES
    (301, 2, 2),
    (302, 1, 1),
    (303, 1, 1);
를 입력하였기 때문에 현재의 결과가 나오는 것은 타당하다. 
```

---

# 4. 첫 번째 정상 COMMIT 추적

실행 파일:

```text
code/chapter09/03_commit_transaction.sql
```

이 실습은 학생 101이 강의 301을 신청하는 하나의 업무 단위를 추적합니다.

## 4-1. 업무 단위 정의

```text
이 트랜잭션에서 함께 성공해야 하는 변경 1:course 301좌석을 1개 확보한다.
변경 2: 학생 101의 lab enrollment를 만든다. 
변경 3: 해당 enrollment의 payment를 만든다. 

하나라도 실패하면 전체를 취소해야 하는 이유: 세 변경은 하나의 업무이므로, 모두 함께 이루어져야 한다. 따라서, 함께 성공하지 않으면 전체를 취소해야한다. 
```

## 4-2. 상태 변화 기록

| 시점 | course 301 남은 좌석 | lab enrollment 9001 | payment 9901 | 설명 |
| --- | ---: | --- | --- | --- |
| BEGIN 전 | 2 | 없음 | 없음 | 초기 상태와 같다. |
| 트랜잭션 내부 | 1 | 생성됨 | 생성됨 | 변경이 실행되었지만 commit 되기 이전이다. |
| COMMIT 후 | 1 | 생성 | 생성됨 | 변경이 실행되어 commit 된 이후이다. |

## 4-3. COMMIT 조건

```text
좌석 UPDATE 기대 영향 행 수: 1
실제 영향 행 수: 1
신청 생성 기대 행 수: 1
결제 생성 기대 행 수: 1
recorded_amount와 payment.amount 일치 여부: 일치
최종 COMMIT 판단: 기대와 일치하고, 검증 결과 통과 메시지를 얻어 commit 하는 것이 적절하다고 판단하였다. 
```

기대 메시지:

```text
Chapter 09 first commit validation passed
```

### SQL 오류가 없었다는 사실만으로 COMMIT하면 안 되는 이유

```text
SQL이 오류 없이 실행되어도 의도한 행이 변경되지 않았거나, 예상보다 많은 행이 변경되었거나, 신청과 결제 금액이 서로 다를 수 있다. 따라서 영향 행 수와 최종 데이터 상태가 업무 규칙에 맞는지 확인한 뒤 COMMIT해야 한다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter09/images/step04_commit.png
```

<img width="1176" height="950" alt="image" src="https://github.com/user-attachments/assets/de90cd18-8717-4e0c-a5ed-be431da0530d" />

---

# 5. ROLLBACK으로 전체 원상복구 확인

실행 파일:

```text
code/chapter09/04_rollback_transaction.sql
```

## 5-1. ROLLBACK 전 예상

```text
트랜잭션 안에서 임시로 바뀔 값: 강의 301의 remaining seats 가 1에서 0으로 줄어들어야 한다. 또한, 신청 9002와 결제 9902가 생성되어야 한다. 
ROLLBACK 후 다시 돌아와야 할 값: 이전 파일에서 commit되고 난 이후, 변경되지 상태인 remaing seats 1, 신청, 결제 행의 수 각각 1이 유지되어야 한다. 
이미 이전 파일에서 COMMIT된 9001/9901은 유지되어야 하는가:유지되어야 한다. 이전 트랜잭션에서 COMMIT된 변경은 ROLLBACK의 대상이 아니기 때문이다. 
```

## 5-2. 실제 결과

```text
ROLLBACK 후 course 301 상태:remaining_seats = 1
ROLLBACK 후 lab enrollments 행 수:1
ROLLBACK 후 payments 행 수:1
9001 존재 여부:존재함
9901 존재 여부:존재함
통과 메시지:Chapter 09 rollback validation passed
```

기대 메시지:

```text
Chapter 09 rollback validation passed
```

## 5-3. ROLLBACK과 IDENTITY

```text
ROLLBACK이 테이블 행 변경을 되돌리는 방식: 현재 트랜잭션에서 실행한 INSERT, UPDATE, DELETE 변경을 취소해 트랜잭션 시작 전 상태로 되돌린다.이때, 현재 트랜잭션에서 아직 commit 되지 않은 변경만 취소될 수 있다. 

IDENTITY 자동 번호가 반드시 이전 값으로 되돌아가지는 않는 이유: PostgreSQL의 자동 번호는 트랜잭션이 취소되더라도 이미 할당된 번호가 다시 회수되지 않는 것이 일반적이다. 따라서, ROLLBACK해도 자동 번호가 이전 값으로 되돌아가지는 않는다.

번호가 건너뛰었다고 데이터 손상이라고 단정할 수 없는 이유: 트랜잭션이 취소되더라도 이미 할당된 번호는 회수되지 않기 때문이다.번호는 고유 식별을 위한 값이지, 빈틈없이 연속되어야 하는 값이 아니다. 따라서, 행을 고유하게 식별하도록 할 수만 있다면, 빠진 값이 있어도 관계 없다. ID를 통해 행을 고유하게 식별하도록 하는 것은 한 번도 빠지지 않는 업무 순번이 있는 것을 보장하지 않는다.
```

---

# 6. 두 번째 COMMIT과 좌석 부족 0행 관찰

실행 파일:

```text
code/chapter09/05_commit_and_sold_out.sql
```

## 6-1. 두 번째 정상 COMMIT

```text
생성된 enrollment id: 9002
학생 id: 103
course id: 302
recorded_amount: 120000
payment id:9902
payment amount:120000
```

## 6-2. 좌석 부족 시도

```text
좌석 확보 UPDATE 기대 영향 행 수:0
실제 영향 행 수:0
후속 enrollment 생성 행 수:0
후속 payment 생성 행 수:0
```

기준상 좌석 부족 시 생성되지 않아야 하는 ID:

```text
9003
9903
```

### `UPDATE 0`이 SQL 실패가 아니라 업무상 실패일 수 있는 이유

```text
SQL 문법과 실행에는 문제가 없어도 조건에 맞는 행이 없으면 UPDATE 결과는 0행일 수 있다.
예를 들어, 강의 302의 남은 좌석이 0이라 좌석 확보되지 않아 UPDATE 결과는 0행이 된다. 이는 SQL 오류라고 할 수는 없지만, 신청이 처리되지 않는 것이다. 따라서, 이러한 경우 SQL 실패가 아니라 업무상 실패이다. 
```

### 영향 행 수가 0인데 신청과 결제를 계속 생성하면 어떤 정합성 문제가 생기나요?

```text
좌석을 확보하지 못했는데 신청과 결제가 생성될 수 있다. 따라서, 실제 좌석 상태와 신청, 결제 데이터가 맞지 않아져 업무 정합성 문제가 생길 수 있다. 
```

---

# 7. 주 실습 최종 정합성 검증

실행 파일:

```text
code/chapter09/06_transaction_validation.sql
```

## 7-1. lab 최종 상태

| 항목 | 기대값 | 실제값 | 일치? |
| --- | ---: | ---: | --- |
| course_inventory 행 수 | 3 | 3 | 일치 |
| lab enrollments 행 수 | 2 | 2 | 일치 |
| payments 행 수 | 2 | 2 | 일치 |
| course 301 remaining | 1 | 1 | 일치 |
| course 302 remaining | 0 | 0 | 일치 |
| course 303 remaining | 1 | 1 | 일치 |

## 7-2. 주요 행

```text
9001 = student 101 / course 301 / amount 100000 / payment 9901
실제: student 101 / course 301 / recorded_amount 100000 / payment 9901 amount 100000

9002 = student 103 / course 302 / amount 120000 / payment 9902
실제 :student 103 / course 302 / recorded_amount 120000 / payment 9902 amount 120000

9003·9903 = 존재하지 않아야 함
실제: 둘 다 존재하지 않는다. 
```

## 7-3. 보호 대상 확인

```text
course_project.enrollments 행 수:5
전체 recorded_amount:590000
활성 건수/금액:3, 340000
취소 제외 건수/금액:4, 440000
```

기대값:

```text
course_project.enrollments = 5
전체 = 590000
활성 = 3 / 340000
취소 제외 = 4 / 440000
```

최종 기대 메시지:

```text
Chapter 09 main transaction validation passed
```

### transaction_lab 실습 후에도 course_project 기준 상태를 다시 검사하는 이유

```text
transaction_lab에서 실습을 진행할 때, 강의 가격 조회 등을 위해 course_project 데이터를 참조한다. 이때, 격리된 실습이 기존 프로젝트 데이터를 손상하지 않도록 하고자 하였기 때문에 기준 상태를 검증하는 것도 필요하다. 따라서 실습 후에도 새 실습 상태가 맞는 것 뿐 아니라, 기존 course_project 데이터가 변경되지 않았는지, 그 상태를 확인하여야 기존 데이터를 변경하지 않고, 별도 공간을 사용하고자 한 목적이 지켜졌는지 검증할 수 있기 때뭉니다.
```

---

# 8. ACID를 이번 실습으로 설명

교과서 정의를 그대로 복사하지 말고 이번 좌석·신청·결제 사례로 작성합니다.

```text
Atomicity: 좌석 차감, 신청 생성, 결제 생성을 하나의 트랜잭션으로 처리한다. 중간에 문제가 생기면 전체를 ROLLBACK할 수 있다. 즉, 모두 반영되거나 모두 취소된다.

Consistency: 트랜잭션 전후에 데이터가 정해진 규칙을 만족해야 한다. 예를 들어 남아있는 좌석이 있을 때만 신청을 만들 수 있고, 이에 따른 금액과 결제가 생긴다. 또한, 사용 좌석 수와 활성 신청 수의 일치해야한다. 이러한 규칙들은 트랜잭션 전후 상태 모두에서 만족되어야 한다.

Isolation: 트랜잭션이 진행될 때, 여러 사용자의 작업이 정해진 방식으로 조정될 수 있도록 한다. 예를 들어, 동시에 여러 사용자가 하나 남은 좌석을 신청하고자 한다면, 해당 좌석 행을 잠가 같은 좌석이 동시에 차감되어 정원을 초과하는 상황을 막도록 한다.

Durability: COMMIT이 완료되면 좌석 차감, 신청, 결제 변경이 확정되어 이후에도 유지된다.
```

### Atomicity와 Consistency가 같은 뜻이 아닌 이유

```text
Atomicity는 여러 변경이 모두 반영되거나 모두 취소되는 처리 방식이다. 반면, Consistency는 그 결과가 업무 규칙과 데이터 제약을 만족하는 상태인지를 말한다.따라서, Atomicity가 있다고 반드시 업무 내용이 옳아지는 것은 아니다. 예를 들어, 잘못된 SQL세 개가 한 번에 묶여 commit 될 수 있는데, 이러한 경우 잘못된 상태로 확정되어 Consistency는 만족되지 않을 수 있다. 
따라서, 제약 조건, 업무 조건, 영향 행 수, 검증 select,트랜잭션이 함께 필요하다. 
```

---


# 10. 선택 실습 — 취소와 좌석 복구

실행 파일:

```text
code/chapter09/08_cancel_and_restore.sql
```

```text
9001 취소 성공 행 수: 1행
course 301 좌석 변화: remaining_seats가 1에서 2로 증가
같은 취소를 다시 시도한 행 수: 0행
두 번째 좌석 복구 행 수: 0행
payment 9901 유지 여부: 유지됨
최종 ROLLBACK 후 원상복구 여부: 원상복구됨. 신청 9001은 수강중, 좌석은 1이며 payment 9901도 유지됨
통과 메시지: Chapter 09 cancel rollback validation passed
```

기대 흐름:

```text
9001 수강중 → 취소 1행
course 301 remaining 1 → 2
같은 취소 재시도 → 0행
추가 좌석 복구 → 0행
마지막 ROLLBACK → 주 실습 기준으로 복구
```

### 같은 취소를 두 번 처리해도 좌석이 두 번 증가하지 않아야 하는 이유

```text
같은 취소를 두 번 처리할 때, 좌석이 두번 증가되면 실제 capacity보다 remaining_seats 가 커질 수 있다. 
따라서, 취소 처리는 수강중 상태인 신청만 취소하도록 조건을 두어 같은 취소를 두 번 처리할 때, 좌석이 두 번 증가하지 않도록 하는 것이 필요하다. 
첫 번째 처리에서 상태가 이미 취소로 바뀌면 동일한 시도를 두번하더라도, 두 번째 처리에서는 수강중 상태인 신청이 남아있지 않는다.
따라서, 조건에 맞는 행이 없어 0행을 변경하게 되고, 좌석 복구가 일어나지도 않는다.
```
---

### 증거 화면

실제 수행했다면 권장 경로:

```text
<img width="1136" height="905" alt="image" src="https://github.com/user-attachments/assets/f3c2dbfa-187d-4384-8583-9f16b3351959" />
```


# 12. 개인 프로젝트 트랜잭션 시나리오 설계

Chapter 07에서 시작한 개인 프로젝트를 사용합니다.

둘 이상의 변경이 함께 성공해야 하는 업무를 **하나** 선택합니다.

예:

```text
배송 사건 생성 + 재고 차감 + 영양 성분 기입 
```

## 12-1. 업무 정의

```text
시나리오 ID: P09-T01
업무 이름: 배송 기록과 회원 영양정보 기록
사용자 행동: 구독 배송 한 건의 실제 배송 내용을 기록한다.
왜 하나의 트랜잭션이어야 하는가:
배송 기록과 배송 상품 기록, 해당 회원의 영양정보 연결 기록이 함께 남아야 하기 때문이다.
일부만 저장되면 배송은 기록됐는데 상품이나 영양정보 기록이 빠지는 상태가 될 수 있다.
```

## 12-2. 트랜잭션 설계표

| 항목 | 내 설계 |
| --- | --- |
| BEGIN 전 확인 상태 | 대상 subscription과 연결된 member가 존재하고, 기록할 배송 상품과 수량 및 각 상품의 기준 영양정보가 확인되어야 한다. |
| 잠금/경쟁 가능 데이터 | 동일한 배송 사건이 동시에 기록되어 중복 생성될 가능성이 있다. 그러나 이는 허용되지 않도록 하여야 한다. |
| 변경 1 | shipment에 배송 사건 한 건을 기록한다. |
| 기대 영향 행 수 | 1행 |
| 변경 2 | 배송 상품과 수량을 shipment_items에 기록한다. |
| 기대 영향 행 수 | 배송 상품 종류 수만큼 |
| 추가 변경 | 각 배송 상품의 영양 성분별 배송 당시 수치를 shipment_item_nutritions에 기록한다. 회원은 shipment → subscription → members 관계를 통해 확인한다. |
| COMMIT 전 검증 | 배송 행이 1건인지, 배송 상품과 수량이 입력 내용과 맞는지, 각 배송 상품에 필요한 영양정보가 연결되어 있는지 검증한다. 배송 당시 영양 수치도 해당 배송 상품의 기준 영양정보 및 수량과 맞는지 확인한다. |
| COMMIT 조건 | 배송, 배송 상품, 배송 당시 영양정보 기록이 모두 생성되고 연결 관계와 수치 검증을 통과할 때 COMMIT한다. |
| ROLLBACK 조건 | 배송이나 상품 또는 영양정보 기록이 누락되거나, 입력한 상품·수량·영양 수치가 검증 기준과 다르면 전체를 ROLLBACK한다. |

## 12-3. 실패 시나리오

최소 두 개 작성합니다.

```text
실패 1:
어느 단계에서 발생: shipment 기록 후 shipment_items를 기록하는 단계
남으면 안 되는 부분 상태: 배송 사건은 있지만 배송 상품 내역이 없거나 일부만 기록된 상태
ROLLBACK 후 기대 상태: 이번 트랜잭션에서 생성한 shipment와 shipment_items가 모두 없어야 한다.

실패 2:
어느 단계에서 발생: shipment_items 기록 후 shipment_item_nutritions를 기록하거나 검증하는 단계
남으면 안 되는 부분 상태: 배송과 배송 상품은 기록됐지만 배송 당시 영양정보가 누락되거나 수치가 맞지 않는 상태
ROLLBACK 후 기대 상태: 이번 트랜잭션에서 생성한 shipment, shipment_items, shipment_item_nutritions가 모두 없어야 한다.
```

## 12-4. SQL 초안

```sql
-- 아직 테이블 구현 전이라면 의사 SQL이어도 됩니다.

BEGIN;
INSERT INTO shipment (
    subscription_id,
    scheduled_date,
    shipped_at,
    status
)
VALUES (
    대상_subscription_id,
    배송_예정일,
    실제_배송시각,
    배송_상태
)
RETURNING id INTO 새_shipment_id;

INSERT INTO shipment_items (
    shipment_id,
    item_id,
    quantity
)
VALUES
    (새_shipment_id, 상품_id_1, 수량_1),
    (새_shipment_id, 상품_id_2, 수량_2);

-- 배송 상품별 영양 성분과 배송 당시 수치를 기록한다.
-- nutrition_id는 해당 상품의 기준 영양정보를 가리킨다.
-- amount_at_delivery는 배송 수량 기준으로 계산한 배송 당시 영양 수치다.
INSERT INTO shipment_item_nutritions (
    shipment_item_id,
    nutrition_id,
    amount_at_delivery
)
VALUES
    (배송상품행_id_1, 기준영양정보_id_1, 배송당시_영양수치_1),
    (배송상품행_id_1, 기준영양정보_id_2, 배송당시_영양수치_2),
    (배송상품행_id_2, 기준영양정보_id_3, 배송당시_영양수치_3);


-- 검증
shipment는 1행 생성됐는가?
shipment_items의 행 수·상품·수량은 입력과 일치하는가?
shipment_item_nutritions가 배송 상품별 영양 성분과 배송 당시 수치를 기록했는가?
각 배송 상품은 올바른 shipment에 연결되어 있는가?
ASSERT 위 검증이 모두 통과함;

검증에 성공하는 경우
COMMIT;

검증 중 하나라도 실패하는 경우
ROLLBACK;

```

---

# 13. AI를 트랜잭션 리뷰어로 활용

AI에게 완성 SQL부터 요구하지 않습니다.

## 13-1. 사용한 프롬프트

```text

```

권장 질문 요소:

```text
1. 하나의 업무 단위가 어디까지인지
2. BEGIN 전 확인할 상태
3. 경쟁 가능 데이터와 잠금 필요성
4. 각 변경의 기대 영향 행 수
5. 여러 테이블 최종 정합성 검증
6. COMMIT 조건
7. ROLLBACK 조건
8. 동시 실행 위험
을 먼저 검토한 뒤 PostgreSQL 초안을 제안하도록 요청
```

## 13-2. AI 제안 검토

| AI 제안 | 수용 / 수정 / 보류 / 거절 | 실제 또는 논리 검증 | 판단 이유 |
| --- | --- | --- | --- |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

### AI SQL에서 확인한 가장 중요한 위험

```text

```

### “오류가 없으면 COMMIT”만으로 부족한 이유

```text

```

---

# 14. 최종 성찰

아래 문장은 본인의 말로 작성합니다.

```text
1. 트랜잭션은 여러 SQL을 단순히 묶는 것이 아니라
   ____________________________________________________________ 이다.

2. ROLLBACK이 필요한 대표 상황은
   ____________________________________________________________ 이다.

3. 조건부 UPDATE의 영향 행 수가 중요한 이유는
   ____________________________________________________________ 이다.

4. 제약조건이 있어도 트랜잭션이 필요한 이유는
   ____________________________________________________________ 이다.

5. Lock이 필요한 이유는
   ____________________________________________________________ 이다.

6. AI가 만든 트랜잭션 SQL을 검토할 때 가장 먼저 확인할 것은
   ____________________________________________________________ 이다.
```

---

# 15. 제출 체크리스트

- [ ] `chapter09_answer.md`를 본인 저장소에 만들었다.
- [ ] Chapter 07·08 기준 상태를 확인했다.
- [ ] `transaction_lab` 스키마와 초기 데이터를 만들었다.
- [ ] 정상 COMMIT의 전·중·후 상태를 기록했다.
- [ ] ROLLBACK 후 부분 변경이 남지 않는지 확인했다.
- [ ] ROLLBACK과 IDENTITY 번호의 차이를 설명했다.
- [ ] 좌석 부족 시 영향 행 수 0을 관찰했다.
- [ ] 영향 행 수 0일 때 후속 행이 생성되지 않음을 확인했다.
- [ ] `06_transaction_validation.sql` 최종 검증을 통과했다.
- [ ] `course_project`가 변경되지 않았음을 확인했다.
- [ ] ACID를 이번 실습 사례로 설명했다.
- [ ] Lock 실습 또는 두 세션 절차 분석을 수행했다.
- [ ] 개인 프로젝트 트랜잭션 시나리오를 작성했다.
- [ ] AI 제안의 COMMIT/ROLLBACK/영향 행 수 검증을 확인했다.
- [ ] 핵심 캡처는 3~4장 정도로 정리했다.
- [ ] 캡처에 비밀번호·개인정보가 없다.
- [ ] GitHub 웹에서 Markdown과 이미지가 정상적으로 보인다.
- [ ] 최종 파일을 commit/push했다.

---

# 16. LMS 제출 URL

아래 형식의 **본인 GitHub 파일 URL**을 LMS에 제출합니다.

```text
https://github.com/<본인-GitHub-ID>/<본인-저장소>/blob/main/assignments/chapter09/chapter09_answer.md
```

내 제출 URL:

```text

```

> 교수자 템플릿 URL, 저장소 메인 URL, Raw URL이 아니라 **작성 완료된 본인의 `chapter09_answer.md` 파일 화면 URL**을 제출합니다.
