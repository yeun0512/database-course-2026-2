# Chapter 10 확장 실습 답안 템플릿

> **과제:** 실행 계획으로 인덱스 효과 검증하기  
> **사용 방법:** 이 파일을 내려받아 본인의 GitHub 저장소에 `chapter10_answer.md`라는 이름으로 저장한 뒤 실습하면서 바로 작성합니다.  
> **제출 방법:** LMS에는 파일을 직접 업로드하지 않고, **본인 GitHub 저장소의 `chapter10_answer.md` 파일 URL**을 제출합니다.

---

## 제출 전 주의

이 파일과 캡처 화면에는 실제 비밀번호, 전체 DB 접속 URL, API Key, 개인정보를 기록하지 않습니다.

```text
GitHub 계정 또는 별칭: yeun0512   
과제 작성일: 2026.11.18
사용한 AI 도구: GPT
```

---

# 1. PostgreSQL 버전과 시작 환경 확인

다음을 실행합니다.

```sql
SELECT version();
SELECT current_database();
SELECT current_user;
SELECT current_schema();
SHOW search_path;
```

| 확인 항목 | 실제 결과 | 의미 |
| --- | --- | --- |
| PostgreSQL 버전 | 18 | 사용 중인 PostgreSQL의 버전이 18이다. |
| `current_database()` | ai_database_book | 현재 ai_database_book 데이터베이스에 연결되어 있다. |
| `current_user` | postgres | 현재 사용자가 postgres이다. |
| `current_schema()` | public | 기본 스키마가 public이다. |
| `search_path` | public, "$user" | 테이블을 찾을 때 먼저 public을 확인하고, 그다음 사용자 이름과 같은 스키마를 확인한다. |

### PostgreSQL 버전을 기록해야 하는 이유

```text
PostgreSQL 버전에 따라 사용할 수 있는 기능이나 SQL 동작이 달라질 수 있기 때문이다.같은 SQL을 실행해도 버전에 따라 결과나 지원 여부가 달라질 수 있다.
예를 들어, PostgresSQL 16과 18의 기능 차이 때문에 동일 SQL을 활용하더라도 실행 계획이 달라질 수 있다. 
```

> 이 장의 자동 검증 기준은 PostgreSQL 16입니다. PostgreSQL 18 이상에서는 B-tree Skip Scan 등으로 동일 SQL의 실행 계획이 달라질 수 있습니다.

---

# 2. Chapter 07·08 기준 상태 확인

Chapter 10은 기존 `course_project`를 변경하지 않습니다.

확인 기준:

```text
students = 3
instructors = 2
courses = 3
enrollments = 5

전체 recorded_amount = 590000
활성 = 3건 / 340000
취소 제외 = 4건 / 440000
```

```text
1001 = 완료 / 100000
1004 = 취소 / 150000
1005 = 신청 / 120000
```

### 실제 확인 결과

```text
students: 3
instructors: 2
courses: 3
enrollments: 5
전체 recorded_amount: 590000
활성 신청 건수/금액: 3건 / 340000
취소 제외 건수/금액: 4건 / 440000
```

### 성능 실험을 기존 `course_project`에 대량 데이터를 넣지 않고 별도 스키마에서 하는 이유

```text
기존 course_project 데이터와 수업 프로젝트에 영향을 주지 않고, 실험용 데이터를 자유롭게 추가 및 학습하기 위함이다. 
특히, 본 장에서는 index를 활용하여 성능이 달라지는 것을 확인하는 것이 목표이다. 그러나 기존의 데이터는 너무 작기 때문에 전체를 읽는 것이 더 합리적이고, 이를 활용해서는 성능 실험을 하기에 적절하지 않다. 따라서, 대량 성능 실험을 하면서도 기존 프로젝트에 영향을 주지 않기 위해서 별도 스키마에서 한다.  
```

---

# 3. `performance_lab` 생성과 대량 데이터 확인

다음 파일을 순서대로 실행합니다.

```text
code/chapter10/01_performance_lab_schema.sql
code/chapter10/02_performance_lab_seed.sql
```

## 3-1. 생성 후 행 수

| 테이블 | 기대 행 수 | 실제 행 수 | 일치? |
| --- | ---: | ---: | --- |
| `performance_lab.students` | 10003 | 10003 | 일치 |
| `performance_lab.instructors` | 2 | 2 | 일치 |
| `performance_lab.courses` | 2003 | 2003 | 일치 |
| `performance_lab.enrollments` | 100005 | 100005 | 일치 |

## 3-2. 데이터 분포 확인

| 조건 | 기대 행 수 | 실제 행 수 | 대략적 비율 |
| --- | ---: | ---: | ---: |
| `performance5000@example.com` | 1 | 1 | 약 0.010% |
| `student_id = 5000` | 10 | 10 | 약 0.010% |
| `course_id = 1500` | 50 | 50 | 약 0.050% |
| `course_id = 1500 AND status='수강중'` | 15 | 15 | 약 0.015% |
| 전체 `status='수강중'` | 30001 | 30001 | 약 30.0% |

### 선택도가 낮은 조건과 많은 행을 반환하는 조건은 인덱스 판단에서 어떻게 다르게 볼 수 있나요?

```text
조건에 맞는 행이 전체에서 차지하는 비율이 작으면 선택도가 높은 것이다. 이때, 인덱스를 통해 필요한 행만 찾는 것이 유리할 수 있다. 반대로 조건에 맞는 행이 많으면 인덱스로 많은 행을 하나씩 찾는 것보다 테이블을 순차적으로 읽는 편이 효율적일 것이다 예를 들어, 10만 행 중에서 10행을 찾을 때는 인덱스를 통해 찾는 것이 유리하더라도 3만행을 찾을 때는, 순차적으로 읽는 것이 효율적이다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter10/images/step03_data_scale.png
```

`여기에 데이터 규모 확인 화면을 삽입하세요.`

---

# 4. 인덱스 생성 전 기준 계획 기록

다음 파일을 실행합니다.

```text
code/chapter10/03_baseline_explain.sql
```

> **중요:** `04_create_candidate_indexes.sql`을 먼저 실행하지 않습니다. 기준 계획을 잃으면 같은 조건의 전후 비교가 어려워집니다.

최소 3개 SQL의 실행 계획을 기록합니다.

## Query A

```text
업무 질문: 특정 이메일을 가진 학생을 조회한다.
WHERE / JOIN / ORDER BY / LIMIT: WHERE email = 'performance5000@example.com'
예상 반환 행 수: 1
실제 반환 행 수: 1
```

```sql
-- 대상 SQL
EXPLAIN (ANALYZE, BUFFERS)
SELECT id, name, email
FROM performance_lab.students
WHERE email = 'performance5000@example.com';
```

| 관찰 항목 | 기록 |
| --- | --- |
| 주요 Scan/계획 노드 |  |
| estimated rows | 1 |
| actual rows | 1 |
| Filter |  |
| Index Cond |  |
| Buffers hit/read |  |
| Planning Time |  |
| Execution Time |  |

## Query B

```text
업무 질문: 특정 학생의 신청 내역과 해당 학생·강의 정보를 함께 조회한다.
WHERE / JOIN / ORDER BY / LIMIT: WHERE e.student_id = 5000; students와 courses를 JOIN
예상 반환 행 수: 10
실제 반환 행 수: 10

EXPLAIN (ANALYZE, BUFFERS)
SELECT
    e.id,
    s.name AS student_name,
    c.title AS course_title,
    e.status,
    e.recorded_amount
FROM performance_lab.enrollments AS e
JOIN performance_lab.students AS s ON s.id = e.student_id
JOIN performance_lab.courses AS c ON c.id = e.course_id
WHERE e.student_id = 5000;
```

| 관찰 항목 | 기록 |
| --- | --- |
| 주요 Scan/계획 노드 |  |
| estimated rows | 30001 |
| actual rows | 30001 |
| Filter |  |
| Index Cond |  |
| Buffers hit/read |  |
| Execution Time |  |

## Query C

```text
업무 질문: 수강 중인 신청 내역을 모두 조회한다.
WHERE / JOIN / ORDER BY / LIMIT: WHERE status = '수강중'
예상 반환 행 수: 30001
실제 반환 행 수: 30001

EXPLAIN (ANALYZE, BUFFERS)
SELECT id, student_id, course_id, status
FROM performance_lab.enrollments
WHERE status = '수강중';
```

| 관찰 항목 | 기록 |
| --- | --- |
| 주요 Scan/계획 노드 |  |
| estimated rows | 10 |
| actual rows | 10 |
| Filter |  |
| Index Cond |  |
| Buffers hit/read |  |
| Execution Time |  |

### `cost`와 실제 실행 시간이 같은 개념이 아닌 이유

```text
cost는 PostgreSQL이 여러 실행 계획의 상대적인 효율을 비교하기 위해 사용하는 내부 추정 비용이다. 즉, cost=123.45라고 제시되더라도, 123ms의 시간 단위로 해석하면 안된다.
실제 실행 시간은 내부 비용 추정과는 달라질 수 있기 때문이다.
```

### `EXPLAIN ANALYZE`는 실제 SQL을 실행한다는 점을 왜 기억해야 하나요?

```text
EXPLAIN와 EXPLAIN (ANALYZE, BUFFERS)의 차이가 있음에 주목해야하기 때문이다. 전자는 실행계획을 예측하기만 하지만, EXPLAIN ANALYZE는 계획만 보여주는 것이 아니라 대상 SQL을 실제로 실행한다. 따라서 이 때의 SELECT는 조회 결과를 실제로 읽는 것이다. 이를 적용하면 INSERT, UPDATE, DELETE 가 있는 경우 데이터가 실제로 변경된다. 따라서, 실제 변경을 일으킬 수 있어 안전하지 않을 수 있음에 유의해야하기 때문이다. 
```

### 증거 화면

권장 경로:

```text
assignments/chapter10/images/step04_before_plan.png
```

`여기에 대표 기준 실행 계획을 삽입하세요.`

---

# 5. 후보 인덱스를 만들기 전에 이유 작성

본문 실험 후보는 다음 세 개입니다.

```text
idx_performance_courses_title
idx_performance_enrollments_student_id
idx_performance_enrollments_course_status
```

각 인덱스의 이유를 먼저 작성합니다.

| 후보 인덱스 | 대응 조회 패턴 | 예상 이점 | 컬럼 순서 이유 | 예상 비용/단점 |
| --- | --- | --- | --- | --- |
| `idx_performance_courses_title` | WHERE title = ?	 | 제목에 해당하는 강의를 빠르게 찾는 데 도움이 될 수 있음 | 단일 컬럼 인덱스이므로 컬럼 순서 선택은 없음 | 저장 공간을 사용하고, 강의 추가나 제목 변경 등이 있는 경우 인덱스 갱신 비용이 발생함 |
| `idx_performance_enrollments_student_id` | WHERE student_id = ? | 해당 학생의 신청 내역을 찾는 데 도움이 될 수 있음 | 단일 컬럼 인덱스이므로 컬럼 순서 선택은 없음 | 저장 공간을 사용하고, 신청 데이터 변경 시 인덱스 갱신 비용이 발생함 |
| `idx_performance_enrollments_course_status` | WHERE course_id = ? WHERE course_id = ? AND status = ?	| 특정 강의의 신청 내역에서 특정 상태의 신청으로 범위를 좁히는 데 도움이 될 수 있음	 | course_id를 앞에 두어야, PostgresSQL 16과 같은 환경에서도 course_id와 status를 함꼐 고려하도록 활용할 수 있음 | 저장 공간을 사용하고, 신청 데이터 변경 시 인덱스 갱신 비용이 발생함. 많은 행을 반환하면 이점이 작을 수 있음 |

### “중요한 컬럼이므로 인덱스를 만든다”는 설명이 부족한 이유

```text
컬럼이 중요하다는 사실만으로는 인덱스가 실제 조회에 도움이 되는지 알 수 없다. 조건에 맞는 행이 얼마나 되는지, 인덱스 유지 비용을 감수할 만큼 조회가 자주 일어나는지를 함께 고려해야 한다.또한, 어떤 조회에서 그 컬럼을 조건, 정렬 등에 활용하는지를 고려해야 한다. 
```

### `(course_id, status)`와 `(status, course_id)`가 항상 같은 효과가 아닌 이유

```text
복합 인덱스를 활용하는 경우, 컬럼 순서가 중요하기 떄문이다. 예를들어, (course_id, status)는 특정 강의의 신청 내역을 찾거나, 특정 강의 안에서 상태를 추가로 좁히는 조회에 적합하다. 반대로 (status, course_id)는 상태 조건을 먼저 사용하는 조회이다. 따라서, 특정 강의만 찾는 조회를 하고자 할 때에는, 앞쪽 컬럼인 status 조건이 없어 효율이 떨어질 수 있다. 선행 컬럼을 조건이 되는 것으로 구성하는 것이 적절하며, 두 인덱스의 효과가 항상 같아지는 것은 아니다. 또한, PostgreSQL의 버전에 따라 활용 방식이 달라질 수 있음에도 주의해야한다. 
```

---

# 6. 후보 인덱스 생성

다음을 실행합니다.

```text
code/chapter10/04_create_candidate_indexes.sql
```

생성 후 확인:

```text
후보 인덱스 수: 3
전체 인덱스 수: 9
```

본문 기준:

```text
자동 인덱스 = 6
후보 인덱스 = 3
전체 인덱스 = 9
```

### PRIMARY KEY나 UNIQUE가 이미 인덱스를 만들 수 있는데 같은 목적의 인덱스를 또 만들면 어떤 문제가 생기나요?

```text
이미 있는 인덱스와 같은 역할을 하는 인덱스가 중복되면 저장 공간을 더 사용한다. 또한, 데이터가 삽입, 수정, 삭제 등 변경되는 경우 두 인덱스를 모두 갱신해야 하여 이에 따른 비용이 증가한다. 비용이 증가함에도 불구하고, 조회 성능은 거의 좋아지지 않을 수 있기 때문에 기존 PRIMARY KEY·UNIQUE가 이미 인덱스를 만들 수 있는지 먼저 확인해야한다. 
```

---

# 7. 같은 SQL로 인덱스 후 재측정

다음 파일을 실행합니다.

```text
code/chapter10/05_after_index_explain.sql
```

Chapter 4에서 기록한 **동일 SQL**을 비교합니다.

## Query A 전후 비교

| 항목 | Before | After | 해석 |
| --- | --- | --- | --- |
| 주요 계획 노드 | Index Scan | Index Scan | UNIUE 조건 이미 있었음 |
| actual rows | 1 | 1 | 결과 행 수는 동일해야 함 |
| Buffers hit/read |  |  |  |
| Execution Time |  |  |  |
| Index Cond |  |  |  |

```text
결과 행이 동일했는가: 네. 모두 1행이다. 
읽은 버퍼가 줄었는가: 
계획이 바뀌었는가: 
실행 시간 한 번만으로 결론낼 수 있는가: 아니요. 캐시 상태나 시스템 부하 등에 따라 달라질 수 있기 때문입니다. 
```

## Query B 전후 비교

| 항목 | Before | After | 해석 |
| --- | --- | --- | --- |
| 주요 계획 노드 | Seq Scan | Index Scan |  |
| actual rows | 10 | 10 | 결과 행 수 동일해야 함 |
| Buffers hit/read |  |  |  |
| Execution Time |  |  |  |
| Index Cond |  |  |  |

## Query C 전후 비교

| 항목 | Before | After | 해석 |
| --- | --- | --- | --- |
| 주요 계획 노드 |  |  |  |
| actual rows | 30001 | 30001 | 결과 행 수 동일해야 함 |
| Buffers hit/read |  |  |  |
| Execution Time |  |  |  |
| Index Cond |  |  |  |

### `Index Scan`으로 바뀌었다는 사실만으로 성공이라고 할 수 없는 이유

```text
Index Scan으로 바뀌어도 실행 시간이 줄거나 읽는 버퍼가 감소한다고 보장할 수는 없다.
조건에 맞는 행이 많으면 인덱스를 따라 많은 행을 읽는 비용이 커질 수 있기 떄문이다. 따라서, 결과 행의 일치 여부, actual rows, 버퍼 사용량, 실행 시간 등을 함께 비교해야 한다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter10/images/step07_after_plan.png
```

`여기에 동일 SQL의 사후 실행 계획을 삽입하세요.`

---

# 8. `status` 단독 조회와 Seq Scan 해석

전체 `status = '수강중'`은 약 30%의 행을 반환합니다.

```text
예상 행 수 = 30001
실제 행 수 = 30001
주요 계획 노드 =
```

### 인덱스가 존재해도 PostgreSQL이 Seq Scan을 선택할 수 있는 이유

```text
status = '수강중'은 전체 신청 행의 약 30%를 반환한다.
많은 행을 읽어야 한다면 인덱스로 각 행의 위치를 찾아 테이블을 여러 번 읽는 것보다 테이블을 순차적으로 읽는 편이 효율적일 수 있다. 따라서, 인덱스가 존재하더라도 반환해야하는 행이 너무 많은 경우 Seq Scan을 선택할 수 있다. 
```

### “Seq Scan = 나쁜 계획”이라고 단정하면 안 되는 이유

```text
Seq Scan은 테이블을 순차적으로 읽는 방식이다. 반환할 행이 많거나 테이블이 작을 때는 인덱스를 거쳐 여러 행을 찾는 것보다 효율적일 수 있다.
따라서, 계획의 좋고 나쁨은 실제 반환 행 수, 버퍼 사용량, 실행 시간 등을 확인해야한다. 
```

### PostgreSQL 16과 18 이상에서 복합 B-tree 후행 컬럼 조건의 계획이 다를 수 있는 이유

```text
PostgreSQL 16에서는 (course_id, status) 인덱스에서 후행 컬럼인 status만 조건으로 쓰면 인덱스가 효율적으로 활용되지 않을 수 있다.
이와 달리, PostgreSQL 18 이상은 B-tree Skip Scan을 사용할 수 있다. 따라서, 선두 컬럼 조건이 없어도 후행 컬럼 조건으로 인덱스 탐색을 시도할 수 있다. 따라서 양 자의 계획이 달라질 수 있다. 
```

---

# 9. `ORDER BY`와 `LIMIT`에서 인덱스 관찰

`ORDER BY title`과 `ORDER BY title LIMIT 20` 계획을 비교합니다.

```text
ORDER BY title 계획:

ORDER BY title LIMIT 20 계획:
```

### LIMIT이 있을 때 PostgreSQL이 전체 정렬보다 인덱스 순서를 활용하는 것이 유리할 수 있는 이유

```text
B-tree 인덱스는 title 순서로 저장된다. 예를 들어, LIMIT 20이 있는 경우, 인덱스 순서로 읽을 수 있다면 전체 행을 정렬하지 않고도 앞선 20행을 찾은 뒤 멈출 수 있다.
이는 전체 정렬에 드는 작업을 줄이기 때문에 유리할 수 있다.
``

### 실제 계획에서 Sort 노드 또는 Index Scan을 어떻게 확인했나요?

```
```text
EXPLAIN 결과에서 Sort 노드가 있는지, Index Scan 또는 Index Only Scan이 선택되었는지 확인하였다.
LIMIT 쿼리는 Limit 노드 아래에 어떤 계획 노드가 있는지도 확인하였다. 
```

---

# 10. 인덱스 검토

다음을 실행합니다.

```text
code/chapter10/06_index_review.sql
```

## 10-1. 인덱스별 판단

| 인덱스 | 크기/사용 관찰 | 유지 / 보류 / 제거 | 판단 근거 |
| --- | --- | --- | --- |
| idx_performance_courses_title |  |  |  |
| idx_performance_enrollments_student_id |  |  |  |
| idx_performance_enrollments_course_status |  |  |  |

### `idx_scan = 0`이라는 이유 하나만으로 인덱스를 삭제하면 안 되는 이유

```text
idx_scan은 통계가 초기화된 뒤 관찰된 사용 횟수이므로, 관찰 기간이 짧거나 해당 조회가 아직 실행되지 않았다면 0일 수 있다. 실제 업무에서 드물지만 중요한 조회에 쓰일 수도 있으므로, 관찰 기간과 워크로드, 실행 계획, 인덱스가 제공하는 기능을 함께 확인해야 한다.
```

### 외래키 자식 컬럼 인덱스가 무결성 자체의 필수 조건은 아니지만 성능상 필요할 수 있는 이유

```text
외래키 제약은 자식 행이 존재하지 않는 부모 키를 참조하지 못하게 하지만, 자식 컬럼에 인덱스가 반드시 있어야 하는 것은 아니다. 다만 참조되는 부모 행을 수정하거나 삭제할 때 DB가 관련 자식 행을 확인해야 하므로, 자식 컬럼 인덱스가 없으면 큰 자식 테이블을 훑을 수 있다. 자식 컬럼을 조건으로 자주 조회하거나 JOIN할 때도 인덱스가 도움이 될 수 있다.
```

### 인덱스를 많이 만들었을 때 생기는 쓰기·저장 비용

```text
인덱스마다 저장 공간이 필요하고, 관련 행을 삽입·삭제하거나 인덱스 키 컬럼을 수정할 때마다 인덱스도 갱신해야 한다. 인덱스가 많아지면 쓰기 작업이 느려지고, 저장 공간과 캐시 사용량도 늘 수 있다.
```

---

# 11. 자동 완료 게이트

다음을 실행합니다.

```text
code/chapter10/07_result_validation.sql
```

```text
최종 검증 결과:Chapter 10 performance result validation passed
```

검증할 핵심 내용:

```text
performance_lab 기준 행 수 유지
조회 결과 행 수 유지
후보 인덱스 3개 존재
course_project 기준 상태 유지
```

### 실행 계획 비교와 별도로 결과 행 동일성을 검증해야 하는 이유

```text
실행 계획이 바뀌어도 조회 결과가 같아야 올바른 성능 비교가 가능하다. 계획만 비교하는 경우, 결과가 누락되거나 기존과 달라지는 문제를 놓치게 될 수 있다. 따라서, 실제 반환 행 수와 데이터 상태를 별도로 검증해야 한다.
```

---

# 12. 인덱스 만능론 반박

다음 주장 중 **두 개**를 골라 본문과 실제 실행 계획을 근거로 반박합니다.

```text
A. 인덱스는 많을수록 좋다.
B. 인덱스를 만들었는데 Seq Scan이면 실패다.
C. 모든 FK에는 무조건 같은 방식의 인덱스를 만든다.
D. 실행 시간이 한 번이라도 빨라졌으면 효과가 입증됐다.
E. 선택도가 낮으면 무조건 Index Scan이 나온다.
```

## 주장 1

```text
선택한 주장:A. 인덱스는 많을수록 좋다.

나의 반박:
인덱스가 많아지면 저장 공간을 더 차지한다. 또한, 데이터의 변경이 있는 경우 갱신해야함에 따른 비용이 늘어난다. 따라서, 조회에 도움이 되는 인덱스만 필요한 만큼 만들어야 한다.

실행 계획에서 확인한 근거:
인덱스 생성 전후의 동일 SQL 실행 계획을 비교한다. 이때, 인덱스 생성 전후의 후보 인덱스가 조회에 사용됐는지 여부를 확인한다.
또한, 실행 시간, 버퍼 사용량이 실제로 개선됐는지 확인한다. 이때, 실행 계획에서 인덱스 사용되면서, 개선이 이루어지는 경우 인덱스를 만드는 것이 적절할 것이다.
```

## 주장 2

```text
선택한 주장:B. 인덱스를 만들었는데 Seq Scan이면 실패다.

나의 반박:Seq Scan이 항상 나쁜 계획인 것은 아니다. 예를 들어,  status = '수강중' 조회는 100005행 중 30001행을 반환하므로, 많은 행을 읽는 경우 테이블을 순차적으로 읽는 편이 더 효율적일 수 있다. 또는, 데이터의 크기가 작은 경우 전체를 읽는 것이 더 효율적일 수도 있다. 

실행 계획에서 확인한 근거: 반드시 실패라고 판단하기 이전에, Seq Scan 이 적용되었을 이유를 확인한다. 가령, 너무 많은 행을 반환한 것인지를 확인한다. 예를 들어, status 단독 조회의 EXPLAIN 결과에서 주요 계획 노드와 actual rows를 확인한다. Seq Scan이 나타났다면 실제 반환 행 수를 확인하여 너무 많지 않지 않은지 검토한다. 
```

---

# 13. 개인 프로젝트 조회 패턴과 인덱스 후보

Chapter 07~09에서 발전시킨 개인 프로젝트를 사용합니다.

최소 2개의 **실제 반복 조회 질문**을 먼저 만듭니다.

| ID | 반복 조회 질문 | WHERE | JOIN | ORDER BY/LIMIT | 예상 반환 비율 | 후보 인덱스 |
| --- | --- | --- | --- | --- | --- | --- |
| P10-Q01 | 특정 회원의 배송 내역과 배송된 상품의 영양 정보를 자주 조회한다	 | 회원 ID로 해당 회원의 배송을 제한 | 회원과 배송, 배송과 배송 상품, 배송 상품과 영양 정보 연결	 | 배송일 최신순으로 정렬. 최근 1개월 내역 LIMIT 적용	 | 전체 배송 중 특정 회원의 배송이 차지하는 비율 | (회원 ID, 배송일) 복합인덱스 |
| P10-Q02 | 특정 상품들의 영양 성분을 비교하기 위해 상품별 영양 정보를 자주 조회한다 | 상품 ID로 해당 상품의 영양 정보를 제한 | 상품과 영양 정보 연결	 | 영양 성분명 순 정렬이 필요할 때 적용	 | 전체 상품 영양 정보 중 해당 상품의 정보가 차지하는 비율 | 상품 ID 인덱스 |

## 후보 1

```text
후보 인덱스: 배송 테이블의 회원 ID와 배송일 복합 인덱스
컬럼 순서: (회원 ID, 배송일)
이 조회에 도움이 될 것으로 예상한 이유: 회원 ID로 배송을 찾은 뒤 해당 회원의 배송 내역을 배송일 순으로 확인하는 조회에 대응할 수 있기 때문이다.
쓰기/저장 비용: 배송 추가·삭제 시 인덱스 갱신 비용이 들고 저장 공간을 사용한다. 회원 ID나 배송일을 수정할 때도 인덱스를 갱신해야 한다.
현재 바로 적용 / 후보로 보류: 후보로 보류
```

## 후보 2

```text
후보 인덱스: 상품별 영양 정보 테이블의 상품 ID 인덱스
컬럼 순서: (상품 ID)
이 조회에 도움이 될 것으로 예상한 이유: 특정 상품 ID에 해당하는 영양 정보 행을 찾는 조회에 대응할 수 있기 때문이다.
쓰기/저장 비용: 영양 정보 추가·삭제 시 인덱스 갱신 비용이 들고 저장 공간을 사용한다. 상품 ID를 수정할 때도 인덱스를 갱신해야 한다.
현재 바로 적용 / 후보로 보류: 후보로 보류
```

### 개인 프로젝트 데이터가 너무 적어 성능 검증이 어렵다면

필요한 데이터 규모: 회원,배송,배송 상품,상품별 영양 정보가 충분히 쌓여야 한다.
필요한 데이터 분포: 회원마다 배송 횟수가 다르고, 배송마다 포함되는 상품 수가 다르며, 상품별 영양 정보가 여러 건 연결되는 분포
비교할 SQL: 특정 회원의 배송 내역 조회와 특정 상품의 영양 정보 조회
비교할 지표: 결과 행 수, 실행 계획과 Index Cond, Buffers hit/read, Execution Time
현재 판단 상태: 후보
```

> 작은 데이터에서 Index Scan이 나오지 않는다고 억지로 설정을 바꾸어 특정 계획을 강제하지 않습니다.

---

# 14. AI를 실행 계획 리뷰어로 활용

AI에게 인덱스를 바로 추천하게 하지 않고 실제 실행 계획과 조회 문맥을 제공합니다.

## 14-1. AI에게 전달한 정보

```text
업무 질문:
PostgreSQL 버전:
테이블 행 수:
데이터 분포:
기존 인덱스:
SQL:
EXPLAIN (ANALYZE, BUFFERS) 핵심 결과:
```

## 14-2. AI 제안 검토

| AI 제안 | 수용 / 수정 / 보류 / 거절 | 실제 계획/데이터 근거 | 최종 판단 |
| --- | --- | --- | --- |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

### AI가 제안한 인덱스 중 만들지 않기로 한 것이 있다면 이유

```text

```

### AI가 PostgreSQL 버전이나 데이터 분포를 무시하고 단정한 내용이 있었나요?

```text

```

### AI가 만든 인덱스 제안을 실제 계획 없이 채택하면 위험한 이유

```text

```

---

# 15. 최종 성찰

아래 문장은 본인의 말로 작성합니다.

```text
1. 인덱스가 필요한지 판단할 때 가장 먼저 확인할 것은
   ____________________________________________________________ 이다.

2. 같은 SQL의 인덱스 전후를 비교할 때 통제해야 할 조건은
   ____________________________________________________________ 이다.

3. Seq Scan이 항상 나쁜 것이 아닌 이유는
   ____________________________________________________________ 이다.

4. 실행 시간 한 번보다 계획과 Buffers를 함께 보는 이유는
   ____________________________________________________________ 이다.

5. 내 개인 프로젝트에서 아직 인덱스를 보류한 후보가 있다면 그 이유는
   ____________________________________________________________ 이다.
```

---

# 16. 제출 체크리스트

- [ ] `chapter10_answer.md`를 본인 저장소에 만들었다.
- [ ] PostgreSQL 버전을 기록했다.
- [ ] Chapter 07·08 기준 상태를 확인했다.
- [ ] `performance_lab`의 10003 / 2 / 2003 / 100005 기준을 확인했다.
- [ ] `03_baseline_explain.sql`을 후보 인덱스 생성 전에 실행했다.
- [ ] 기준 실행 계획을 최소 3개 기록했다.
- [ ] 후보 인덱스 3개의 근거를 먼저 작성했다.
- [ ] 동일 SQL의 인덱스 전후 계획을 비교했다.
- [ ] 실행 시간뿐 아니라 Scan, actual rows, Buffers, Index Cond를 확인했다.
- [ ] `status` 단독 조건의 계획을 해석했다.
- [ ] `ORDER BY`와 `LIMIT` 계획을 확인했다.
- [ ] 인덱스 만능론 주장 2개를 반박했다.
- [ ] `07_result_validation.sql`로 최종 상태를 확인했다.
- [ ] 개인 프로젝트의 반복 조회 2개와 인덱스 후보를 작성했다.
- [ ] AI 제안을 실제 실행 계획과 비교했다.
- [ ] 핵심 캡처 3~4장만 넣었다.
- [ ] 캡처에 비밀번호·개인정보가 없다.
- [ ] GitHub 웹에서 Markdown과 이미지가 정상적으로 보인다.
- [ ] 최종 답안을 commit/push했다.

---

# 17. LMS 제출 URL

아래 형식의 **본인 GitHub 파일 URL**을 LMS에 제출합니다.

```text
https://github.com/<본인-GitHub-ID>/<본인-저장소>/blob/main/assignments/chapter10/chapter10_answer.md
```

내 제출 URL:

```text

```

> 저장소 메인 URL, 교수자 템플릿 URL, Raw URL이 아니라 **작성 완료된 본인 `chapter10_answer.md` 파일 화면 URL**을 제출합니다.
