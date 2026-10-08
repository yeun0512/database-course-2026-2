# Project 01_ Proposal

---

# 1. 프로젝트 모델

프로젝트 모델은 다음과 같습니다. 

```프로젝트 이름

도시락 정기 배송 관리

```

### 변경 SQL을 실행하기 전에 현재 DB와 실행 범위를 확인해야 하는 이유

```text
원하지 않았던 데이터가 변경되어 의도와 다른 결괏값이 도출될 수 있다. 
```

---

# 2. 비즈니스 모델

## 2-1. 실행 전 예상

```text
테이블 이름: public.students
한 행의 의미: 학생 한 명
예상 행 수: 학생 수와 동일
기본키: id
필수 열: id, email, name, major, grade
중복을 막는 열: 학생 id
자동 생성 열: timestamp
```

## 2-2. 실행 파일

```text
code/chapter04/01_create_students.sql
```

## 2-3. 실행 후 확인

```text
테이블 생성 성공 여부: 네
실제 행 수: 0
DBeaver에서 확인한 위치: Schemas_public_Tables
```

### 각 열의 역할

| 열 | 타입 | NULL 가능? | 역할 |
| --- | --- | --- | --- |
| id | integer | no | 내부 식별자 |
| name | character varying | no | 필수 문자열 |
| email | character varying | no | 필수, 중복 제한 |
| major | character varying | yes | 전공 정보 |
| grade | integer | yes | 학 정보 |
| created_at | timestamp with time zone | no | 생성 시각 |

### `id`를 학번이나 학생 수로 해석하면 안 되는 이유

```text
id는 데이터베이스 내에서 학생을 식별하도록 만든 고유한 숫자 입력 값이므로 학번과 상이할 수 있다. 이는 행을 구분하기 위한 내부 식별자일 뿐이다. 
```

### 증거 화면

권장 경로:

```text
assignments/chapter04/images/step02_table.png
```

<img width="943" height="868" alt="image" src="https://github.com/user-attachments/assets/677c03bd-b369-4ae8-8fcb-cacab85b2470" />


---

# 3. 샘플 데이터 6명 입력

## 3-1. 실행 전 예상

```text
현재 행 수: 0
실행 후 예상 행 수: 6
예상되는 NULL 포함 학생: 6
```

## 3-2. 실행 파일

```text
code/chapter04/02_insert_students.sql
```

## 3-3. 실제 결과

```text
실제 행 수:6
이준호 grade:3
박서연 존재 여부:네
윤서진 major:NULL
윤서진 grade:NULL
```

### 예상과 실제 비교

```text
예상과 실제가 일치했는가:아니요. 
다르다면 이유:처음 예상 시에는 6행의 윤서진 이외 학생들의 major 와 grade 정보가 모두 존재하는지에 대한 확신을 얻을 수 없었기 때문입니다. 
```

### `created_at` 값이 여러 행에서 같을 수 있는 이유

```text
생성된 시각이 같기 때문이다. 
```

---

# 4. SELECT 복습과 결과 검증

각 문제는 **SQL 실행 전에 예상 행 수를 먼저 작성**합니다.

| 번호 | 조회 문제 | 예상 행 수 | 실제 행 수 | 일치? | 다르면 이유 |
| ---: | --- | ---: | ---: | --- | --- |
| 1 | 전체 학생 | 전체 학생 조회 | 6 | 6 | 일치 |
| 2 | 이름·이메일만 조회 | 전체 이름, 이메일만 조회 | 6 | 6 | 일치 |
| 3 | 특정 전공 | 컴퓨터 공학 전공 조회 | 2 | 5 | 불일치 | NULL이 아닌 행의 수를 모두 세고, 체크로 조회한 항목에 해당하는 행을 표시함.
| 4 | 특정 학년 이상 | 3학년 이상 조회 | 2 | 5 | 불일치 | NULL이 아닌 행의 수를 모두 세고, 체크로 조회한 항목에 해당하는 행을 표시함.
| 5 | 두 전공 중 하나 | 경영학, 컴퓨터 공학 전공 중 하나 조회 | 3 | 5 | 불일치 | NULL이 아닌 행의 수를 모두 세고, 체크로 조회한 항목에 해당하는 행을 표시함.
| 6 | `grade IS NULL` | 전공 입력되지 않은 학생 조회 | 1 | 6 | 학생 전체 행을 표현하고, 체크로 grade 가 NULL인 학생의 행을 표시함. |
| 7 | 전공 `DISTINCT` | 전공이 'DISTINCT'인 학생 조회 | 0 | 5 | 불일치 | 전공이 DISTINCT인 학생은 없기 때문에 조회되지 않을 것이라고 생각하였는데, 전공 항목의 데이터가 있는 경우, 조회되었다. 
| 8 | 정렬 후 상위 3명 | 정렬 후 상위 3명 조회 | 3 | 3 | 일치 |

## 4-1. 내가 직접 작성한 SQL 2개

```sql
-- SQL 1
SELECT major = '컴퓨터공학'
FROM public.students
ORDER BY id;

SELECT COUNT(major = '컴퓨터공학') AS student_count
FROM public.students;
```

```text
이 SQL의 한 행 의미: 전공을 가지는 학생 한 명
예상 행 수: 컴퓨터 공학 전공의 학생 한 명을 예상하여, 2명일 것이라고 예상하였다. 
실제 행 수: 전공을 가지는 학생 전체의 행을 보여주어, 5명이 결과로 도출되었다. 
```

```sql
-- SQL 2
SELECT grade >= 3
FROM public.students
ORDER BY id;

SELECT COUNT(grade >= 3) AS student_count
FROM public.students;

```

```text
이 SQL의 한 행 의미: 학생 한 명
예상 행 수: 2: 3학년 이상의 학생 2명이 보일 것으로 예상하였다. 
실제 행 수: 5: 학년 정보가 있는 학생 전체의 행을 보여주어, 5명이 결과로 도출되었다. 
```

## 4-2. `= NULL` 대신 `IS NULL`을 사용하는 이유

```text
= NULL을 활용하는 경우, NULL 인 행이 6개인 것으로 표시된다. 
```

## 4-3. `ORDER BY` 없이 결과 순서를 믿으면 안 되는 이유

```text
기준이 마련되어 있어야하기 때문이다. 어떠한 기준으로 정렬된 값인지를 알 수 없다. 
```

## 4-4. `DISTINCT`가 원본 데이터를 삭제하는 기능인가요?

```text
아니요. DISTINCT 는 조회 결과에서 중복된 결과를 제거하는 기능입니다. 
```

### 증거 화면

권장 경로:

```text
assignments/chapter04/images/step04_select.png
```

<img width="450" height="440" alt="image" src="https://github.com/user-attachments/assets/8f7ac2b3-cb1e-427b-ab0f-96dd73b65295" />


---

# 5. 내 가상 학생 2명 추가

실명·실제 이메일 대신 가상 데이터를 사용합니다.

## 5-1. 실행 전 계획

```text
학생 A
이름: 김김김
이메일: kimkim@example.com
전공: 경영학
학년: 2

학생 B
이름: 이이이
이메일: leelee@example.com
전공: 컴퓨터 공학
학년 또는 NULL: 2

현재 행 수: 6
추가 후 예상 행 수: 8
```

## 5-2. 내가 실행한 INSERT

```sql
INSERT INTO public.students (name, email, major, grade)
VALUES
    ('김김김', 'kimkim@example.com', '경영학', 2),
    ('이이이', 'leelee@example.com', '컴퓨터공학', 2)
RETURNING id, name, major, grade;
```

## 5-3. 실제 결과

```text
RETURNING 또는 확인 SELECT 결과: 학생 8명 결과가 나왔다. 
실제 전체 행 수: 8
예상과 일치 여부: 일치
```

### 내가 일부 값을 NULL로 둔 이유 또는 NULL을 사용하지 않은 이유

```text
입력 가능한 정보는 최대한 많이 알고 있는 것으로 가정하고 싶었기 때문이다. 
```

---

# 6. 안전한 UPDATE

내가 추가한 가상 학생 한 명만 수정합니다.

## 6-1. 먼저 대상 확인 SELECT

```sql
SELECT *
FROM public.students
WHERE email = 'kimkim@example.com';
```

```text
예상 대상 행 수: 1
실제 대상 행 수: 1
```

## 6-2. UPDATE

```sql
UPDATE public.students
SET grade = 3
WHERE email = 'kimkim@example.com'
returning id, name, email, grade;
```

```text
예상 영향 행 수: 1
실제 영향 행 수: 1
RETURNING 결과: 1명 (김김김) 의 행만 조회되었다. 
```

## 6-3. UPDATE 후 재조회

```sql
SELECT id, name, email, major, grade, created_at
FROM public.students
ORDER BY id ASC;
```

### `WHERE` 없는 UPDATE를 실행하면 위험한 이유

```text
변경하려고 하지 않았던 데이터까지 변경되어버릴 우려가 있기 때문이다.
```

### 증거 화면

권장 경로:

```text
assignments/chapter04/images/step06_update.png
```

<img width="1002" height="593" alt="image" src="https://github.com/user-attachments/assets/6b396c0d-10bc-4dc9-90d6-c009f00616a5" />


---

# 7. 안전한 DELETE

내가 추가한 가상 학생 한 명을 삭제합니다.

## 7-1. 삭제 전 확인

```sql
SELECT *
FROM public.students
WHERE email = 'kimkim@example.com';
```

```text
예상 대상 행 수: 1
실제 대상 행 수: 1
```

## 7-2. DELETE

```sql
DELETE FROM public.students
WHERE email = 'kimkim@example.com'
RETURNING id, name, email;
```

```text
예상 영향 행 수:1
실제 영향 행 수:1
RETURNING 결과: 7, 김김김, kimkim@example.com인 행이 보였다. 
```

## 7-3. 삭제 후 재조회

```sql
SELECT id, name, email, major, grade, created_at
FROM public.students
ORDER BY id ASC;
```

```text
삭제 후 같은 조건의 SELECT 결과 행 수: 7개 
```

### `DELETE` 성공 메시지만 보고 끝내지 않고 다시 SELECT해야 하는 이유

```text
실제로 삭제된 데이터의 행이 제대로 삭제된 것인지, 다른 행에는 영향이 없는지 확인해야한다. 
```
---

# 8. 본문 기준 UPDATE·DELETE 상태 검증

`04_update_delete_students.sql`을 본문 시작 상태에서 실행했다면 다음을 확인합니다.

```text
최종 학생 수: 6
이준호 grade: 4
박서연 존재 여부: 존재하지 않는다. 0행 
```

본문 기준 기대 상태와 비교합니다.

```text
학생 수 = 5
이준호 grade = 4
박서연 = 0행
```

### 내 실제 결과가 기준과 다르다면 원인

```text
만들어낸 학생 1명의 데이터를 삭제하지 않아서, 5명의 기대와 달랐기 때문에 계속해서 기대와 다르다는 오류가 발생하였다. 확인하였을 때, 박서연 삭제와 이준호 학년 수정은 잘 이루어졌으나, 추가했던 학생이 6행으로 남아있어서 오류가 발생하는 것을 알 수 있었다.
```

---

# 9. 의도한 실패 2개 관찰

> 실패 테스트는 데이터베이스 규칙이 실제로 데이터를 보호하는지 확인하는 실험입니다.

## 9-1. 중복 이메일 `UNIQUE` 오류

내가 사용한 SQL:

```sql
SELECT id, name, email
FROM public.students
ORDER BY id;


INSERT INTO public.students (name, email, major, grade)
VALUES ('중복테스트', 'minji@example.com', '테스트전공', 1);
```

```text
오류 메시지 핵심 단서: 고유 제약 조건 위반 
왜 실패해야 맞는가: email은 고유하도록 제약 조건을 설정하였는데, insert로 추가한 정보의 이메일이 기존의 자료와 일치하였기 때문이다. 
어떤 규칙이 작동했는가:email은 고유하도록 한다. 
실패 후 기존 데이터가 어떻게 유지되었는가: 기존데이터는 변화 없이, insert가 진행되지 않았다. 
```

## 9-2. 이름 `NULL` 입력 `NOT NULL` 오류

내가 사용한 SQL:

```sql
INSERT INTO public.students (name, email, major, grade)
VALUES (NULL, 'null_name_test@example.com', '테스트전공', 1);
```

```text
오류 메시지 핵심 단서: not null 제약조건을 위반했습니다. 
왜 실패해야 맞는가: name 칼럼은 null 상태일 수 없다는 제약 조건을 부여하였는데, 입력한 데이터에 이름 정보가 없기 때문이다. 
어떤 규칙이 작동했는가:name은 null일 수 없음. 
```

### 실패한 INSERT 뒤 자동 생성 `id` 번호에 빈 구간이 생길 수 있어도 문제라고 단정할 수 없는 이유

```text

id는 행 식별용 내부 번호일 뿐, 학생 수가 아니기 때문에 빈 구간이 생길 수 있습니다. 
```

### 증거 화면

권장 경로:

```text
assignments/chapter04/images/step09_constraint_error.png
```

`<img width="634" height="512" alt="image" src="https://github.com/user-attachments/assets/3f515a93-32c8-426b-8375-63320a584f76" />`

---

# 10. `verify_students.sql`로 최종 상태 확인

실행 파일:

```text
code/chapter04/verify_students.sql
```

```text
현재 전체 학생 수:6
NULL 개수:2개
이준호 grade:4
박서연 존재 여부:없음
현재 데이터 상태에서 예상과 다른 부분:추가했던 학생 2명 중 한명에 대한 정보가 여전히 남아있다. 
```

### 검증 SQL을 따로 두면 좋은 이유

```text
데이터 변경 없이 현재 상태가 어떤지 확인할 수 있기 때문이다. 오류가 없는 상태이더라도, 데이터가 예상과 다를 수 있는데, 이를 직접 확인할 수 있어서 좋다. 
```

---

# 11. AI를 SQL 작성자가 아니라 검토자로 활용

먼저 본인이 SQL을 작성한 뒤 AI에게 검토를 요청합니다.

## 11-1. 내가 작성한 SQL

```sql
SELECT id, name, email, major
FROM public.students
WHERE email = 'leelee@example.com';

DELETE FROM public.students
WHERE email = 'leelee@example.com'
RETURNING id, name, email;
```

## 11-2. AI에게 전달한 핵심 요청

```text

나는 PostgreSQL 초보자입니다.
아래 SQL을 바로 다시 작성하지 말고 먼저 안전성을 검토해 주세요.
다음 순서로 답해 주세요.

1. 이 SQL이 영향을 줄 것으로 예상되는 행
2. WHERE 조건이 너무 넓거나 모호하지 않은지
3. NULL 처리에서 주의할 점
4. 실행 전에 같은 조건으로 확인할 SELECT
5. 실행 후 결과를 확인할 SELECT
6. 내가 놓친 위험이 있다면 질문 형태로 제시

[SELECT id, name, email, major
FROM public.students
WHERE email = 'leelee@example.com';
DELETE FROM public.students
WHERE email = 'leelee@example.com'
RETURNING id, name, email;]

```

## 11-3. AI 검토 결과

| AI 제안 | 수용 / 수정 / 거절 | 실제 검증 결과 | 나의 이유 |
| --- | --- | --- | --- |
| email과 일치하는 행 삭제하려고 하는데, 고유 제약 조건 만족되어 있는 상태인지 현재 데이터베이스의 제약 조건은 별도로 확인해야 합니다. | 수용 | 실제 데이터를 확인하여 email이 일치하는 사람이 한 명 뿐임을 확인하였다. | 데이터를 수정하려는 경우, 초기 데이터를 직접 확인하는 것이 안전하기 때문이다. |
| 이미 적어 주신 첫 번째 SELECT가 삭제 조건과 같아서 미리 확인하는 용도로 적절합니다. 실행 결과에서 대상 행의 이름과 이메일이 의도한 대상인지 확인하세요. | 수용 | 확인하였다. | 초기 데이터를 직접 확인하는 것이 안전하기 때문이다. |
| 다른 작업이나 사용자가 동시에 같은 행을 수정할 가능성이 있나요? | 수용 | 없음을 확인하였다. | 동시 작업이 가능했다면, 데이터가 잘못 수정될 우려가 있기 때문에 검토하는 것이 필요하다고 생각하였다. |

### AI가 예상한 영향 행 수와 실제 결과가 같았나요?

```text
네 
```

### AI 답변을 실행 전에 검토해야 하는 이유

```text
AI 답변이더라도, 실제로 원하는 방식으로 데이터가 수정된 것인지, 수정 이후에는 이러한 의도가 잘 반영되도록 데이터가 변형되었는지 직접 확인해야하기 때문입니다.  
```

---

# 12. 내 서비스 테이블 하나 확장 설계

Chapter 01~03에서 정한 개인 서비스에서 **테이블 하나**를 선택합니다.

```text
서비스 이름: 도시락 서비스 회원 관리 
테이블 이름: members
한 행의 의미: 회원 한 명
```

| 열 이름 | 저장할 값 | 타입 후보 | NULL 가능? | UNIQUE 후보? | 이유 |
| --- | --- | --- | --- | --- | --- |
| id | 고유 번호 | 숫자 | no | no | 관리를 위해 부여하는 임의의 값으로, unique해야한다. |
| name | 이름 | 문자열 | no | no | 이름은 동일한 사람이 있을 수 있기 때문이다. |
| email | 이메일 | 문자열 | yes | yes | 이메일 정보는 개인마다 하나씩 서로 다르게 가지기 때문이다. |
| age | 나이 | 숫자 | yes | no | 나이가 동일한 사람이 있을 수 있기 때문이다. |
| address | 주소 | 문자열 | no | no | 하나의 주소에 여러 명이 거주하는 경우, 동일한 주소를 가질 수 있기 때문이다. |

```text
PK 후보: id
업무 식별자 후보: member
아직 미확정인 규칙: 도시락의 종류와 관련된 테이블을 어떤 방식으로 만들고, 연결할지 확정하지 못하였다.
```

## 선택: CREATE TABLE 초안

> 아직 확정되지 않은 업무 규칙은 억지로 제약조건으로 만들지 않습니다.

```sql
SELECT current_database();
SELECT current_user;
SELECT current_schema();
SHOW search_path;

DO $$
BEGIN
    IF current_database() <> 'ai_database_book' THEN
        RAISE EXCEPTION
            '생성 중단: 현재 데이터베이스는 %입니다. ai_database_book 연결을 선택하세요.',
            current_database();
    END IF;

    IF to_regnamespace('public') IS NULL THEN
        RAISE EXCEPTION
            '생성 중단: public 스키마가 존재하지 않습니다.';
    END IF;

    IF NOT has_schema_privilege(current_user, 'public', 'USAGE') THEN
        RAISE EXCEPTION
            '생성 중단: 사용자 %에게 public 스키마 USAGE 권한이 없습니다.',
            current_user;
    END IF;

    IF NOT has_schema_privilege(current_user, 'public', 'CREATE') THEN
        RAISE EXCEPTION
            '생성 중단: 사용자 %에게 public 스키마 CREATE 권한이 없습니다.',
            current_user;
    END IF;

    IF current_setting('transaction_read_only')::boolean THEN
        RAISE EXCEPTION
            '생성 중단: 현재 연결이 읽기 전용입니다.';
    END IF;
END
$$;

CREATE TABLE public.students (
    id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    age INTEGER,
    address VARCHAR(100) UNIQE NOT NULL
);


```

### AI에게 검토받은 뒤 수정한 부분

```text
정의하신 서비스는 회원 정보를 관리하므로 테이블 이름은 students가 아니라 members가 적절합니다.
또 주소는 여러 회원이 공유할 수 있다고 하셨으므로 UNIQUE를 두지 않아야 합니다.

CREATE TABLE public.members (
    id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    age INTEGER,
    address VARCHAR(100) NOT NULL
);
```

---

# 13. 최종 성찰

아래 문장은 본인의 말로 작성합니다.

```text
1. SQL 실행 성공과 올바른 대상 선택이 다른 이유는
   단순한 실행의 성공은 의도한 데이터를 대상으로 했는지를 보장하는 것이 아니기 때문이다.

2. UPDATE와 DELETE 전에 SELECT를 먼저 해야 하는 이유는
   데이터에 있어 변경을 일으키는 것이기 때문에, 영향을 미치는 범위를 분명히 하는 것이 안전하기 때문이다.

3. 영향받은 행 수를 확인해야 하는 이유는
   원하는 데이터만 변경된 것인지, 다른 데이터에는 영향이 없는지를 직접 검토하기 위함이다.

4. UNIQUE 또는 NOT NULL 오류를 '보호 장치가 정상 동작한 결과'라고 볼 수 있는 이유는
   초기 부여한 제약 조건을 만족하지 않은 경우, 오류로 인식하여 데이터를 수정하지 않은 결과이기 때문이다.

5. AI가 SQL을 만들어 주더라도 내가 반드시 확인해야 하는 것은
   내가 계획한 목표 달성에 적절한 문장인지, 다른 데이터의 변형은 없는지 등 이다.
```

---

# 14. 제출 체크리스트

- [o] `chapter04_answer.md`를 본인 저장소에 만들었다.
- [o] 현재 DB와 실행 환경을 확인했다.
- [o] `public.students`를 생성했다.
- [o] 샘플 6명 입력 결과를 검증했다.
- [o] SELECT 문제에서 실행 전 예상 행 수를 작성했다.
- [o] 가상 학생 2명을 추가했다.
- [o] UPDATE 전후를 SELECT로 확인했다.
- [o] DELETE 전후를 SELECT로 확인했다.
- [o] UNIQUE 오류를 관찰했다.
- [o] NOT NULL 오류를 관찰했다.
- [o] `verify_students.sql`로 상태를 확인했다.
- [o] AI 제안을 실제 SQL 결과와 비교했다.
- [o] 개인 서비스 테이블 하나를 확장 설계했다.
- [o] 핵심 캡처는 3~4장 정도로 제한했다.
- [o] 비밀번호·개인정보가 캡처에 없다.
- [o] Markdown 이미지가 GitHub 웹 화면에서 정상 표시된다.
- [o] commit/push를 완료했다.

---

# 15. LMS 제출 URL

아래 형식의 **본인 GitHub 파일 URL**을 LMS에 제출합니다.

```text
https://github.com/<본인-GitHub-ID>/<본인-저장소>/blob/main/assignments/chapter04/chapter04_answer.md
```

내 제출 URL:

```text
https://github.com/yeun0512/database-course-2026-2/edit/main/chapter04/chapter04_answer.md
```

> 교수자 템플릿 URL이나 저장소 메인 URL이 아니라 **작성 완료된 본인 `chapter04_answer.md` 파일 화면 URL**을 제출합니다.

