# Chapter 08 확장 실습 답안 템플릿

> **과제:** JOIN과 집계로 서비스 질문에 답하기  
> **사용 방법:** 이 파일을 내려받아 본인의 GitHub 저장소에 `chapter08_answer.md`라는 이름으로 저장한 뒤 실습하면서 바로 작성합니다.  
> **제출 방법:** LMS에는 파일을 직접 업로드하지 않고, **본인 GitHub 저장소의 `chapter08_answer.md` 파일 URL**을 제출합니다.

---

## 제출 전 주의

이 파일과 캡처 화면에는 실제 비밀번호, 전체 DB 접속 URL, API Key, 개인정보를 기록하지 않습니다.

```text
GitHub 계정 또는 별칭:yeun0512
과제 작성일:2026.11.02
사용한 AI 도구:GPT
```

---

# 1. Chapter 07 기준 상태 확인

다음을 실행합니다.

```text
code/chapter08/00_check_course_project.sql
```

## 1-1. 사전 검사 결과

```text
검증 메시지:Chapter 08 prerequisite check passed

students 행 수:3
instructors 행 수: 2
courses 행 수: 3
enrollments 행 수: 5

전체 신청 건수: 5
전체 recorded_amount:590000
활성 신청 건수:3
활성 recorded_amount: 340000
취소 제외 신청 건수: 4
취소 제외 recorded_amount:440000
```

기준값:

```text
students = 3
instructors = 2
courses = 3
enrollments = 5

전체 = 5 / 590000
활성 = 3 / 340000
취소 제외 = 4 / 440000
```

### 기준값이 다르면 그대로 진행하면 안 되는 이유

```text
기준값이 예상과 다르면 데이터가 추가, 변경되었거나 정상적이지 않은 상태일 수 있다. 따라서, 이후 단계의 조회 결과가 잘못 나타나거나, 데이터가 중복되거나 누락될 수 있다. 따라서, 기준 상태와 동일하게 맞추고 진행해야 한다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter08/images/step01_prerequisite.png
```

<img width="1336" height="813" alt="image" src="https://github.com/user-attachments/assets/4ace1a82-ce24-4e11-a266-ebac4117f2de" />

---

# 2. 업무 질문을 SQL보다 먼저 정의하기

다음 세 질문을 각각 SQL 작성 전에 먼저 정의합니다.

## 질문 A

```text
업무 질문: 취소되지 않은 신청 수를 강의별로 보여주세요
결과 한 행의 의미: 강의 한 개
포함 상태: 신청, 수강중, 완료
제외 상태: 취소
JOIN할 테이블: courses, enrollments
JOIN 경로: courses.id = enrollments.course_id
INNER JOIN / LEFT JOIN 선택: LEFT JOIN
그 이유: 취소 제외 신청이 0건인 강의도 결과에 포함해 0건으로 보여야 하기 때문이다.  
예상 행 수: 3행
```

## 질문 B

```text
업무 질문: 취소되지 않은 신청 수를 학생별로 보여주세요
결과 한 행의 의미: 학생 한 명
포함 상태: 신청, 수강중, 완료
제외 상태: 취소
JOIN할 테이블:students, enrollments
JOIN 경로: students.id = enrollments.student_id
집계 대상: 취소 제외 수강신청의 ID
예상 결과: 3행, 2건/2건/0건
```

## 질문 C

```text
업무 질문: 강사 별 취소 제외 신청 수를 조회하도록 해주세요
결과 한 행의 의미: 강사 한 명
포함 상태: 신청, 수강중, 완료
제외 상태: 취소
0건인 부모도 보여야 하는가: 보여야 한다. 강사는 존재하지만, 취소되지 않은 신청 수가 0건인 강사가 존재할 수 있다. 이런 경우, 단순 LEFT JOIN을 한다면, 어떤 강사는 '취소되지 않은 신청'을 가지지 않아 누락될 수 있다. 누락이 되지 않도록 하는 것이 적절하므로, 0건인 부모도 보이도록 한다. 
NULL을 어떻게 해석할 것인가: 원본 강사 테이블에 저장된 값으로 해석해서는 안된다. 이는 JOIN 조건에 맞는 오른쪽 행이 없어서 결과 과정에서 생성된 NULL 이라고 해석해야 한다.
예상 결과: 4건/0건
```

---

# 3. INNER JOIN과 다중 JOIN

## 3-1. 신청 한 건마다 학생 이름과 강의 제목 조회

실행 전 예상:

```text
결과 한 행 = 수강신청 한 건에 대한 학생, 강의 이름
예상 행 수 = 5
JOIN 경로 = enrollments.student_id -> students.id, enrollments.course_id -> courses.id
```

내가 실행한 SQL:

```sql
SELECT
    e.id AS enrollment_id,
    s.name AS student_name,
    c.title AS course_title,
    e.status
FROM course_project.enrollments AS e
INNER JOIN course_project.students AS s
    ON e.student_id = s.id
INNER JOIN course_project.courses AS c
    ON e.course_id = c.id
ORDER BY e.id;

```

실제 결과:

```text
실제 행 수: 5
예상과 일치 여부: 일치한다.
```

### 학생 이름이 여러 번 보이는 것이 중복 오류가 아닐 수 있는 이유

```text
학생 이름이 여러 번 보이는 것은 한 학생이 여러 강의에 신청할 수 있기 때문이다. 즉, 학생과 신청이 일대다의 관계를 가지기 때문이다. 결과 한 행의 기준은 학생 한 명이 아니라 수강신청 한 건이므로, 같은 학생 이름이 반복되어도 중복 오류라고 볼 수 없다.
```

## 3-2. 학생·강의·강사까지 연결

```text
결과 한 행 = 수강신청 한 건에 대한 학생, 강의, 강사 이름
강사까지 가는 JOIN 경로 = enrollments -> courses -> instructors
```

```sql
SELECT
    e.id AS enrollment_id,
    s.name AS student_name,
    c.title AS course_title,
    i.name AS instructor_name,
    e.status,
    e.recorded_amount,
    e.enrolled_at
FROM course_project.enrollments AS e
JOIN course_project.students AS s
    ON e.student_id = s.id
JOIN course_project.courses AS c
    ON e.course_id = c.id
JOIN course_project.instructors AS i
    ON c.instructor_id = i.id
ORDER BY e.id;
```

실제 행 수:

```text
5행
```

### 증거 화면

권장 경로:

```text
assignments/chapter08/images/step03_inner_join.png
```

<img width="1372" height="974" alt="image" src="https://github.com/user-attachments/assets/f81a087b-a56a-4e1f-bdf6-ea8473e37af0" />

---

# 4. LEFT JOIN과 0건 표현

## 4-1. 강의별 취소 제외 신청 수

신청이 없는 강의도 결과에 남도록 작성합니다.

실행 전:

```text
결과 한 행 = 강의 한 개
강의 303의 예상 실제 신청 수 = 0건
강의 303의 예상 고유 학생 수 = 0건
강의 303의 예상 recorded_amount = 0
```

내 SQL:

```sql
SELECT
    c.id,
    c.title,
    COUNT(e.id) AS non_cancelled_count
FROM course_project.courses AS c
LEFT JOIN course_project.enrollments AS e
    ON c.id = e.course_id
   AND e.status <> '취소'
GROUP BY c.id, c.title
ORDER BY c.id;
```

실제 결과:

```text
강의 301:2
강의 302:2
강의 303:0
```

## 4-2. `COUNT(*)`와 `COUNT(e.id)` 비교

강의 303을 기준으로 작성합니다.

```text
COUNT(*) 결과: 1
COUNT(e.id) 결과: 0
COUNT(DISTINCT e.student_id) 결과: 0
```

### 왜 `COUNT(*) = 1`인데 실제 신청 수는 0일 수 있나요?

```text
LEFT JOIN은 연결되는 신청이 없어도 강의 303의 행을 결과에 남긴다. 이때 오른쪽 신청 정보는 NULL인 행 하나로 표시되므로 COUNT(*)는 그 행을 세어 1이 된다. 반면 COUNT(e.id)는 NULL이 아닌 신청 ID만 세므로 0이다.
```

### 자식 사건 수를 셀 때 `COUNT(child.id)`가 더 적절한 이유

```text
LEFT JOIN 결과에는 실제 자식 행이 없어도 부모 행이 남을 수 있다. COUNT(*)는 그 부모 행까지 세지만, COUNT(child.id)는 실제로 연결된 자식 행만 세기 때문이다.
```

---

# 5. `LEFT JOIN`에서 `ON`과 `WHERE` 조건 비교

취소 제외 신청만 연결한다고 가정합니다.

## 5-1. 조건을 `ON`에 둔 경우

```sql
SELECT
    s.id,
    s.name,
    COUNT(e.id) AS non_cancelled_count
FROM course_project.students AS s
LEFT JOIN course_project.enrollments AS e
    ON s.id = e.student_id
   AND e.status <> '취소'
GROUP BY s.id, s.name
ORDER BY s.id;
```

```text
결과 학생 수: 3명
박서연 포함 여부: 포함된다.
```

## 5-2. 조건을 `WHERE`에 둔 경우

```sql
SELECT
    s.id,
    s.name,
    COUNT(e.id) AS non_cancelled_count
FROM course_project.students AS s
LEFT JOIN course_project.enrollments AS e
    ON s.id = e.student_id
WHERE e.status <> '취소'
GROUP BY s.id, s.name
ORDER BY s.id;
```

```text
결과 학생 수: 2명
박서연 포함 여부: 포함되지 않는다. 
```

## 5-3. 차이 설명

```text
ON 조건이 LEFT JOIN의 오른쪽 연결 대상을 제한하는 방식: 왼쪽 행을 유지하면서 JOIN 대상만을 제한한다. 
WHERE 조건이 JOIN 이후 결과 행을 제거하는 방식: JOIN 결과가 만들어진 이후, 참이 아닌 행을 제거한다. 

이번 사례에서 ON = 3명, WHERE = 2명이 되는 이유: '박서연'은 취소 제외 신청을 가지지 않는다. 따라서, JOIN을 하는 경우 오른쪽 행이 e.status가 NULL이 된다.
ON 조건을 사용하는 경우, NULL이더라도 왼쪽 행이 유지된다. 따라서, 전체 학생 수인 3명으로 조회된다. 그러나 WHERE 조건을 사용하는 경우, 오른쪽 행이 NULL이라면, 참으로 평가되지 않기 때문에 제거될 것이다. 즉 '박서연'이 제거되어 2명으로 조회될 것이다. 
```

---

# 6. 신청이 없는 학생 찾기 — 두 방법 비교

## 방법 1. `LEFT JOIN ... IS NULL`

```sql
SELECT
    s.id,
    s.name,
    s.email
FROM course_project.students AS s
LEFT JOIN course_project.enrollments AS e
    ON s.id = e.student_id
   AND e.status <> '취소'
WHERE e.id IS NULL;
```

## 방법 2. `NOT EXISTS`

```sql
SELECT
    s.id,
    s.name,
    s.email
FROM course_project.students AS s
WHERE NOT EXISTS (
    SELECT 1
    FROM course_project.enrollments AS e
    WHERE e.student_id = s.id
      AND e.status <> '취소'
);
```

```text
방법 1 결과: 103/박서연/seoyeon@example.com
방법 2 결과: 103/박서연/seoyeon@example.com
두 결과가 같은가: 네
찾아진 학생: 박서연
```

### 두 방식의 공통 의미를 자신의 말로 설명

```text
두 방법 모두 취소되지 않은 수강신청이 하나도 없는 학생을 찾는다. LEFT JOIN 방식은 연결된 신청 ID가 NULL인 학생을 찾고, NOT EXISTS 방식은 조건에 맞는 신청이 존재하지 않는 학생을 찾는다.
```

---

# 7. 기본 집계 검산

다음 결과를 직접 확인합니다.

| 분석 범위 | 예상 건수 | 실제 건수 | 예상 금액 | 실제 금액 | 일치? |
| --- | ---: | ---: | ---: | ---: | --- |
| 전체 신청 | 5 | 5 | 590000 | 590000 | 일치 |
| 활성 신청 | 3 | 3 | 340000 | 340000 | 일치 |
| 취소 제외 | 4 | 4 | 440000 | 440000 | 일치 |
| 취소 | 1 | 1 | 150000 | 150000 | 일치 |

## 7-1. 전체 평균 `recorded_amount`

```text
예상 평균:118000
실제 평균:118000
```

## 7-2. 취소 제외 평균

```text
예상 평균:110000
실제 평균:110000
```

### `recorded_amount`를 실제 회계 매출이라고 부르면 안 되는 이유

```text
이는 신청 시점에 기록한 금액이다. 따라서, 실제로 결제까지 성공했는지, 환불 등 취소되지는 않았는지를 고려하는 것은 아니다. 따라서, 실제 회계 매출이라고 부르면 안 된다.
```

---

# 8. `GROUP BY`, `HAVING`, `FILTER`

## 8-1. 상태별 신청 건수

```sql
SELECT
    status,
    COUNT(*) AS enrollment_count,
    SUM(recorded_amount) AS total_recorded_amount
FROM course_project.enrollments
GROUP BY status
ORDER BY CASE status
    WHEN '신청' THEN 1
    WHEN '수강중' THEN 2
    WHEN '완료' THEN 3
    WHEN '취소' THEN 4
    ELSE 99
END;
```

결과:

```text
신청:2
수강중:1
완료:1
취소:1
상태별 합계:5
```

### 상태별 건수 합이 전체 신청 5건과 맞는지 검산

```text
2+1+1+1=5로, 전체 신청 5건과 같다. 
```

## 8-2. 강의별 취소 제외 신청 수와 금액

```sql
SELECT
    c.id AS course_id,
    c.title AS course_title,
    COUNT(e.id) AS non_cancelled_count,
    COUNT(DISTINCT e.student_id) AS student_count,
    COALESCE(SUM(e.recorded_amount), 0) AS non_cancelled_recorded_amount
FROM course_project.courses AS c
LEFT JOIN course_project.enrollments AS e
    ON c.id = e.course_id
   AND e.status <> '취소'
GROUP BY c.id, c.title
ORDER BY c.id;
```

```text
강의 301:2
강의 302:2
강의 303:0
강의별 합계를 다시 더한 값:4
전체 취소 제외 기준 440000과 일치 여부:일치한다.
```

## 8-3. `HAVING` 사용

취소 제외 신청이 2건 이상인 강의를 조회합니다.

```sql
SELECT
    c.id,
    c.title,
    COUNT(e.id) AS enrollment_count
FROM course_project.courses AS c
JOIN course_project.enrollments AS e
    ON c.id = e.course_id
WHERE e.status <> '취소'
GROUP BY c.id, c.title
HAVING COUNT(e.id) >= 2
ORDER BY c.id;
```

```text
예상 강의 수:2
실제 강의 수:2
```

---

# 9. 과대 집계 오류 직접 관찰

강사 201의 강의 가격 합계를 구한다고 가정합니다.

## 9-1. 신청까지 JOIN해서 잘못 집계한 결과

```sql
SELECT
    i.id AS instructor_id,
    i.name AS instructor_name,
    SUM(c.price) AS wrong_course_price_sum
FROM course_project.instructors AS i
JOIN course_project.courses AS c
    ON i.id = c.instructor_id
JOIN course_project.enrollments AS e
    ON c.id = e.course_id
GROUP BY i.id, i.name
ORDER BY i.id;
```

```text
강사 201 잘못된 가격 합계: 440000
```

본문 기준:

```text
440000
```

## 9-2. 강의 수준에서 올바르게 집계

```sql
SELECT
    i.id AS instructor_id,
    i.name AS instructor_name,
    COALESCE(SUM(c.price), 0) AS course_price_sum
FROM course_project.instructors AS i
LEFT JOIN course_project.courses AS c
    ON i.id = c.instructor_id
GROUP BY i.id, i.name
ORDER BY i.id;
```

```text
강사 201 올바른 가격 합계:220000
```

본문 기준:

```text
220000
```

## 9-3. 왜 두 결과가 달라졌나요?

```text
JOIN 전 강의 행 수: 3개 
JOIN 후 강의가 반복된 이유: 각 강의의 신청 개수 (신청 행 수)만큼, 각 강의가 반복되기 때문이다. 
SUM이 무엇을 반복해서 더했는가: 신청이 여러 건 있는 강의의 경우, 신청 행마다 강의 가격을 반복해서 더했다. 
```

### `SUM(DISTINCT c.price)`를 일반적인 해결책으로 사용하면 안 되는 이유

```text
SUM(DISTINCT c.price)는 강의 행을 구별하는 것이 아니라 가격 값의 중복을 제거하는 것이다. 서로 다른 두 강의의 가격이 같으면 실제로 두 강의의 가격은 합산 되어야 함에도, 제거될 수 있다. 따라서, 강의 가격 합계를 구하려면 신청 테이블과 JOIN하지 않고 강의 수준에서 집계해야 한다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter08/images/step09_over_aggregation.png
```

<img width="1150" height="947" alt="image" src="https://github.com/user-attachments/assets/ba13289b-0b32-4e5a-9e15-ffd7c0346f6a" />

<img width="1164" height="956" alt="image" src="https://github.com/user-attachments/assets/f5d96629-6e5e-4656-8ded-fdd669381dbf" />

---

# 10. 상세 결과 ↔ 집계 결과 교차 검산

강의 하나를 선택합니다.

```text
선택한 course_id:301
강의 제목:데이터베이스 입문
```

## 10-1. 상세 신청 행 조회

```sql
SELECT
    e.id AS enrollment_id,
    e.student_id,
    e.status,
    e.recorded_amount
FROM course_project.enrollments AS e
WHERE e.course_id = 301
  AND e.status <> '취소'
ORDER BY e.id;
```

```text
상세 행 수:2
상세 recorded_amount를 직접 더한 값:100000+100000=200000
```

## 10-2. 집계 SQL

```sql
SELECT
    COUNT(*) AS enrollment_count,
    SUM(recorded_amount) AS total_recorded_amount
FROM course_project.enrollments
WHERE course_id = 301
  AND status <> '취소';
```

```text
집계 건수:2
집계 금액:200000
```

## 10-3. 비교

```text
상세 행 수와 COUNT 결과 일치 여부: 일치한다.
상세 금액 합과 SUM 결과 일치 여부: 일치한다. 
다르다면 원인: 일치한다. 
```

---

# 11. 자동 완료 게이트

다음을 실행합니다.

```text
code/chapter08/03_join_aggregation_validation.sql
```

```text
최종 검증 메시지:Chapter 08 join and aggregation validation passed
```

기대 메시지:

```text
Chapter 08 join and aggregation validation passed
```

### 자동 검증이 통과했어도 사람이 SQL 의미를 설명해야 하는 이유

```text
자동 검증은 정해진 데이터와 조건에서 결과가 예상된 것과 맞는지 확인한다. 그러나, 구체적으로 어떤 행을 세고 더하는지, JOIN과정에서 행이 잘못 세어지도록 나타나지는 않는지, 조건이 업무 규칙에 맞는지 확인하는 것이 필요하다. 따라서, 통과 이후에도 사람이 SQL 의미를 설명할 수 있어야 한다.
```

---

# 12. 개인 프로젝트 업무 질문 3개 만들기

Chapter 07에서 작성한 개인 프로젝트를 사용합니다.

| 질문 ID | 업무 질문 | 결과 한 행 | 포함/제외 범위 | JOIN 경로 | 집계 대상 | 검산 방법 |
| --- | --- | --- | --- | --- | --- | --- |
| P08-Q01 | 회원별 배송된 상품 수량은 얼마인가? | 회원 한 명 | 기록된 전체 배송 건 포함 | members -> subscription -> shipment -> shipment_items | shipment_items.quantity | 회원별 배송 상품 수량을 직접 더해 비교한다. |
| P08-Q02 | 업체별 등록 상품 수와 평균 가격은 얼마인가? | 업체 한 곳 | 등록된 전체 상품 | factory -> items | item.price의 평균, factory | 업체별로 등록된 상품 내역을 직접 확인하고, 그 가격의 평균을 구해서 비교한다. |
| P08-Q03 | 가입 목적 별, 목적이 변경되지 않은 회원 수는 몇명인가? | 목적 한 개 | 현재 해당 목적으로 기록된 회원 (변경되거나, 취소된 목적인 경우 제외한다) | purpose -> member_purpose | member_purpose | 목적 별 회원 목록에서 현재 목적이라고 명시된 회원에 대해 직접 세어 비교한다. |

## 12-1. 질문 1 SQL

```sql
SELECT
    m.id AS member_id,
    COALESCE(SUM(si.quantity), 0) AS shipped_item_quantity
FROM members AS m
LEFT JOIN subscription AS s
    ON m.id = s.member_id
LEFT JOIN shipment AS sh
    ON s.id = sh.subscription_id
LEFT JOIN shipment_items AS si
    ON sh.id = si.shipment_id
GROUP BY m.id
ORDER BY m.id;
```

```text
예상 결과: 회원별 배송 상품 수량 합계가 표시된다. 배송 상품이 없는 회원은 0이다.
실제 결과: 미실행
검산 결과: 미실행
```

## 12-2. 질문 2 SQL

```sql
SELECT
    f.id AS factory_id,
    COUNT(i.id) AS item_count,
    AVG(i.price) AS average_item_price
FROM factory AS f
LEFT JOIN items AS i
    ON f.id = i.factory_id
GROUP BY f.id
ORDER BY f.id;
```

```text
예상 결과: 업체별 상품 수와 평균 상품 가격이 표시된다. 
실제 결과: 미실행
검산 결과: 미실행
```

## 12-3. 질문 3 SQL

```sql
SELECT
    p.id AS purpose_id,
    COUNT(DISTINCT mp.member_id) AS member_count
FROM purpose AS p
LEFT JOIN member_purposes AS mp
    ON p.id = mp.purpose_id
GROUP BY p.id
ORDER BY p.id;
```

```text
예상 결과: 목적별로 그 목적을 기록한 고유 회원 수가 표시된다. 다만, 변경되었거나 취소된 회원인 경우 집계하지 않는다. 
실제 결과: 미실행
검산 결과: 미실행
```

> 아직 개인 프로젝트 테이블을 PostgreSQL로 완성하지 않았다면 SQL 초안과 예상 검산 방법까지만 작성하고 `미실행`이라고 명시합니다.

---

# 13. AI를 JOIN·집계 리뷰어로 활용

## 13-1. 내가 AI에게 전달한 질문

```text
나는 데이터베이스를 처음 배우는 학생이야.
이전에 도시락 정기 배송 프로젝트를 발전시키려고 해.
확정되지 않은 업무 규칙을 확정짓지 말고, 지금까지 결정된 내용만을 바탕으로 알려줘.

아직 프로젝트 테이블이 만들어지지 않은 상황에서, 앞으로 어떻게 프로젝트를 구성할지에 대해 미리 고안한 내용이야.
다음의 작성된 내용을 보고 결과 한 행, 상태 범위, JOIN 경로, INNER'LEFT 선택, COUNT 대상에 대해 검토해줘.
그리고, 과대 집계 위험이 있는지, 검산은 어떻게 할 수 있는지 방법 알려줘.

처음 배우는 학생이 이해하기 쉽게 알려줘.
다음은 현재까지 작성한 내용이야.

| P08-Q01 | 회원별 배송된 상품 수량은 얼마인가? | 회원 한 명 | 기록된 전체 배송 건 포함 | members -> subscription -> shipment -> shipment_items | shipment_items.quantity | 회원별 배송 상품 수량을 직접 더해 비교한다. |
| P08-Q02 | 업체별 등록 상품 수와 평균 가격은 얼마인가? | 업체 한 곳 | 등록된 전체 상품 | factory -> items | item.price의 평균, factory | 업체별로 등록된 상품 내역을 직접 확인하고, 그 가격의 평균을 구해서 비교한다. |
| P08-Q03 | 가입 목적 별, 목적이 변경되지 않은 회원 수는 몇명인가? | 목적 한 개 | 현재 해당 목적으로 기록된 회원 (변경되거나, 취소된 목적인 경우 제외한다) | purpose -> member_purpose | member_purpose | 목적 별 회원 목록에서 현재 목적이라고 명시된 회원에 대해 직접 세어 비교한다. |

## 12-1. 질문 1 SQL

```sql
SELECT
    m.id AS member_id,
    COALESCE(SUM(si.quantity), 0) AS shipped_item_quantity
FROM members AS m
LEFT JOIN subscription AS s
    ON m.id = s.member_id
LEFT JOIN shipment AS sh
    ON s.id = sh.subscription_id
LEFT JOIN shipment_items AS si
    ON sh.id = si.shipment_id
GROUP BY m.id
ORDER BY m.id;

## 12-2. 질문 2 SQL
```sql
SELECT
    f.id AS factory_id,
    COUNT(i.id) AS item_count,
    AVG(i.price) AS average_item_price
FROM factory AS f
LEFT JOIN items AS i
    ON f.id = i.factory_id
GROUP BY f.id
ORDER BY f.id;

## 12-3. 질문 3 SQL
```sql
SELECT
    p.id AS purpose_id,
    COUNT(DISTINCT mp.member_id) AS member_count
FROM purpose AS p
LEFT JOIN member_purposes AS mp
    ON p.id = mp.purpose_id
GROUP BY p.id
ORDER BY p.id;

```

## 13-2. 내 SQL과 AI SQL 비교

| 검토 항목 | 내 판단/SQL | AI 제안 | 최종 선택 | 이유 |
| --- | --- | --- | --- | --- |
| 결과 한 행 | 	Q1: 회원 한 명, Q2: 업체 한 곳, Q3: 목적 한 개 | 모두 적절하다고 제안받음. | 수용 | 한 행의 의미를 재정의 할 것을 제안 받지 않았기 때문이다. |
| 상태 범위 | Q1: 기록된 전체 배송 건, Q2: 등록된 전체 상품, Q3: 현재 목적이며 변경·취소된 목적은 제외 | Q1은 전체 배송 기록에 미배송·취소가 포함될 수 있다고 확인. Q3은 SQL에 현재·취소 여부 조건이 없음 | Q1은 배송 상태 고려하지 않은, 전체 배송건으로 확정함, Q3는 회원과 목적 테이블의 연결 과정에서 현재 목적인지, 변경된 목적일지를 어떻게 관리할지를 확정해야할 것이라고 판단함. 목적의 상태를 목적, 취소로 구분하여 관리하기로 결정함. 이를 반영하여 다시 작성하기로 결정함. | Q1의 경우 질문이 명확히 설정되지 않았다고 생각되어, 이를 구체화하였다. Q3의 경우 목적 상태 관리 방안이 명확하지 않았다고 생각하여, 이를 구체화하였다. |
| JOIN 경로 | Q1: members -> subscription -> shipment -> shipment_items, Q2: factory -> items, Q3: purpose -> member_purposes | 모두 적절하다고 제안받음. | 수용 | 수정할 것을 제안 받지 않았기 때문이다. |
| INNER/LEFT 선택 | 모두 LEFT | 부모 행도 결과에 남겨야 하므로 적절하다고 제안받음. | LEFT JOIN 유지 | 수정할 것을 제안 받지 않았기 때문이다. |
| COUNT 대상 | Q1: shipment_items의 상품 수량, Q2: 업체 별 상품 id, Q3: 목적 별 회원 id | Q1에서 상품 행 개수가 아니라 수량의 합을 표현하고 있어 수정이 필요하다고 제안받음. | 수정 | AI가 지적한 문제는 어떤 상태를 기준으로 집계할지 명확히 설정하지 않았을 때인데, Q1에 대해서는 배송 상태를 고려하지 않고, 전체 배송 건으로 집계하고자 결정하였기 때문이다. |
| 과대 집계 위험 | - | 현재 경로에서 단정하기 어렵다고 제시했음. | 수용 | 수정할 내용이 없었기 때문이다. |
| 상세 검산 방법 | 직접 배송 상품, 업체 별 상품, 목적 별 회원을 세어본다. | 각 질문에서 한 대상의 원본 내역을 조회해 직접 계산한 값과 비교할 것을 제안받음. | 수용 | 구체적으로 어떻게 확인할 것인지에 대한 제안이기 때문에 수용하였다. |

### AI가 만든 SQL에서 발견한 위험 또는 확인한 점

```text
Q1은 “배송된 상품”이라고 표현했지만 SQL은 기록된 모든 배송 건을 합산한다고 제안받았다.
이에, 배송 상태와 관련 없이 합하도록 수정하였다. 

Q2는 달리 제안받은 내용이 없다.

Q3는 작성된 질문과 SQL이 상이하다는 것을 지적받았다. 현재 목적만 세고 변경·취소된 목적은 제외한다고 했지만,SQL에 현재 여부나 취소 여부를 가리는 조건이 없었다. 이에, 먼저 목적에 대한 상태를 어떻게 관리할지부터 명확히 하지 않았다는 문제점을 발견하였다. 따라서, member_purposes 에서 목적의 상태를 목적, 취소로 관리하기로 결정하였다. 또한, 이 중 '목적' 상태만을 집계하도록 수정하고자 한다. 
```

### AI SQL이 실행 성공했다고 바로 정답이라고 할 수 없는 이유

```text
SQL이 실행된다는 것은 문법과 조건에 맞았다는 것일 뿐, 업무 질문에 맞도록 표현했음이 보장되는 것은 아니다. 아직 확정되지 않은 규칙 때문에 문제가 생겼거나, JOIN과정에서 같은 데이터가 반복되는 등 오류가 발생했을 수 있다. 따라서, 원본 자료를 직접 확인하는 것, SQL의 범위와 업무 기준이 맞는지 확인하는 것이 필요하다.
```

---

# 14. 최종 성찰

아래 문장은 본인의 말로 작성합니다.

```text
1. JOIN SQL을 작성하기 전에 가장 먼저 정해야 하는 것은
   결과 한 행이 무엇을 의미하는지이다.

2. LEFT JOIN에서 COUNT(*) 대신 COUNT(child.id)를 검토해야 하는 이유는 양 자의 결괏값이 실제 연결된 자식 행이 있는지, 없는지에 따라 달라지기 때문이다. COUNT(*)는 자식행이 없는 경우에도 부모행은 결과에 남지만, COUNT(child.id)는 연결이 있는 경우에만 센다. 따라서, 확인하고자 하는 값이 무엇인지에 따라 적절히 선택해야하기 때문이다. 

3. ON과 WHERE 조건 위치가 중요한 이유는 LEFT JOIN에서 ON과 WHERE의 조건 위치에 따라 자식행이 없는 부모를 제외하는지 아닌지가 달라지기 떄문이다. ON을 활용하는 경우, 자식행이 없는 경우에도 부모는 제외되지 않지만, WHERE은 자식행이 없는 부모가 제외될 수 있다. 따라서, 어떤 행을 남기고자 했는지에 따라 적절히 선택해야하기 때문이다.

4. 여러 1:N 관계를 JOIN한 뒤 바로 SUM하면 위험한 이유는
   관계가 어떻게 설정되어있는지에 따라, 같은 값이 반복될 수 있다. 이에 따라 합계가 실제보다 커질 수 있기 때문이다.

5. 집계 결과를 신뢰하기 전에 가장 좋은 검산 방법 중 하나는
   집계 대상의 행을 직접 조회하여, 집계 결과와 비교하는 이다.
```

---

# 15. 제출 체크리스트

- [0] `chapter08_answer.md`를 본인 저장소에 만들었다.
- [0] `00_check_course_project.sql`이 통과했다.
- [0] 업무 질문마다 결과 한 행을 먼저 정의했다.
- [0] INNER JOIN과 다중 JOIN을 실행했다.
- [0] LEFT JOIN에서 0건 부모를 확인했다.
- [0] `COUNT(*)`와 `COUNT(child.id)` 차이를 설명했다.
- [0] ON과 WHERE 조건 위치 차이를 직접 비교했다.
- [0] `LEFT JOIN ... IS NULL`과 `NOT EXISTS`를 비교했다.
- [0] 전체/활성/취소 제외 기준값을 직접 검산했다.
- [0] `GROUP BY`, `HAVING`을 사용했다.
- [0] 과대 집계 오류와 수정 결과를 비교했다.
- [0] 상세 결과와 집계 결과를 교차 검산했다.
- [0] `03_join_aggregation_validation.sql`이 통과했다.
- [0] 개인 프로젝트 업무 질문 3개를 작성했다.
- [-] AI SQL을 실행 성공 여부가 아니라 의미와 검산 결과로 평가했다.
- [0] 핵심 캡처는 3~4장 정도만 사용했다.
- [0] 비밀번호·개인정보·비밀정보가 없다.
- [0] GitHub 웹에서 Markdown과 이미지가 정상적으로 보인다.
- [0] 최종 답안을 commit/push했다.

---

# 16. LMS 제출 URL

아래 형식의 **본인 GitHub 파일 URL**을 LMS에 제출합니다.

```text
https://github.com/<본인-GitHub-ID>/<본인-저장소>/blob/main/assignments/chapter08/chapter08_answer.md
```

내 제출 URL:

```text
https://github.com/yeun0512/database-course-2026-2/edit/main/chapter08/chapter08_answer.md
```

> 저장소 메인 URL, 교수자 템플릿 URL, Raw URL이 아니라 **작성 완료된 본인 `chapter08_answer.md` 파일 화면 URL**을 제출합니다.
