# Chapter 02 확장 실습 답안 템플릿

> **과제:** 데이터와 DBMS의 기본 개념  
> **사용 방법:** 이 파일을 내려받아 본인의 GitHub 저장소에 `chapter02_answer.md`라는 이름으로 저장한 뒤 실습하면서 바로 작성합니다.  
> **제출 방법:** LMS에는 파일을 직접 업로드하지 않고, **본인 GitHub 저장소의 `chapter02_answer.md` 파일 URL**을 제출합니다.

---

## 제출 전 개인정보 주의

LMS에서 제출자를 확인할 수 있으므로 이 공개 Markdown 파일에 학번이나 실명을 반드시 적을 필요는 없습니다.

```text
GitHub 계정 또는 별칭: yeun0512
과제 작성일: 2026.09.14
사용한 AI 도구: GPT
```

> 실제 비밀번호, API Key, 전체 DB 접속 URL, 개인정보가 포함된 화면은 올리지 않습니다.

---

# 1. PostgreSQL에서 현재 위치 확인

## 1-1. 실행한 SQL

```sql
SELECT version();
SELECT current_database();
SELECT current_user;
SELECT current_schema();
SHOW search_path;
```

## 1-2. 실행 결과 기록

```text
PostgreSQL 버전:PostgreSQL 18.6 on x86_64-windows, compiled by msvc-19.44.35228, 64-bit
현재 데이터베이스:postgres
현재 사용자:postgres
현재 스키마:public
search_path:public, "$user"
```

## 1-3. 구조를 내 말로 설명

```text
PostgreSQL은: 데이터를 저장하고, SQL을 실행하는 데이터베이스 관리 시스템 (소프트웨어)이다. 

현재 접속한 데이터베이스는: postgres이다.

스키마는: 데이터베이스 안에서 객체를 구분하고 관리하는 공간이다. 

DBeaver 또는 psql 같은 도구는: 사용자에게 데이터베이스의 결과를 보여주는 클라이언트이다. 
```

## 1-4. 계층 구조 완성

```text
사용자
→ DEeaver에서 SQL을 작성한다. 
→ PostgreSQL DBMS
→ Postgres가 SQL 실행
→ PostgresSQL이 결과 또는 오류를 반환한다. 
→ Dbeaver가 결과를 화면에 표시한다. 
→ 행 / 열
```

## 1-5. 증거 화면

권장 경로:

```text
assignments/chapter02/images/step01_environment.png
```

```markdown
![PostgreSQL 현재 위치 확인](./images/step01_environment.png)
```
<img width="817" height="698" alt="image" src="https://github.com/user-attachments/assets/69870ea9-4072-4655-959a-fc2ed23313f4" />

---

# 2. 데이터베이스 안의 스키마와 테이블 관찰

## 2-1. 스키마 조회 결과

실행한 SQL:

```sql
SELECT schema_name
FROM information_schema.schemata
ORDER BY schema_name;
```

관찰한 스키마 이름 중 3개 이내를 적습니다.

```text
1. pg_catalog
2. pg_temp_4
3. pg_toast
```

### `public`은 무엇인가요?

```text
나의 설명: postgres, schemas의 하위 항목으로 만든 스키마 입니다. 
```

### 데이터베이스와 스키마는 같은 것인가요?

```text
나의 설명: 아니요. 스키마는 데이터베이스 안에 속하는 하위 규모의 공간입니다. 데이터베이스는 업무 공간 전체를 일컫는 말이고, 스키마는 데이터베이스 안에서 테이블 등 객체를 구분하여 관리하는 공간입니다. 
```

## 2-2. 현재 보이는 테이블 조회

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_type = 'BASE TABLE'
  AND table_schema NOT IN ('pg_catalog', 'information_schema')
ORDER BY table_schema, table_name;
```

```text
조회된 사용자 테이블 수 또는 눈에 띈 테이블: 2개, table_schema의 practice와 table_name의 members

아직 테이블이 거의 없어도 괜찮은 이유: 아직 테이블에 정보를 입력하지 않았기 때문입니다. 
```

## 2-3. 관찰 정리

```text
PostgreSQL 서버 안에는 여러 데이터베이스가 있을 수 있다.
한 데이터베이스 안에는 여러 스키마가 있을 수 있다.
스키마 안에는 테이블과 같은 데이터베이스 객체가 존재한다.
```

---

# 3. TEMP TABLE로 테이블·행·열·키 직접 확인

## 3-1. 임시 테이블 생성 완료 확인

- [ ] `ch02_students` 생성
- [ ] `ch02_courses` 생성
- [ ] `ch02_enrollments` 생성

각 테이블의 **한 행 의미**를 적습니다.

| 테이블 | 한 행의 의미 |
| --- | --- |
| `ch02_students` | 학생 한 명 |
| `ch02_courses` | 강의 한 개 |
| `ch02_enrollments` | 강의 등록 사건 한 개 |

## 3-2. 열의 의미 확인

### `ch02_students`

| 열 | 값의 의미 | 내부 식별자 / 업무 식별자 / 일반 속성 |
| --- | --- | --- |
| `id` | DB 내 학생 행 구분 식별자 | 내부 식별자 |
| `student_number` | 학번 | 업무 식별자 |
| `name` | 학생 이름 | 일반 속성 |
| `major` | 전공 | 일반 속성 |

### `ch02_enrollments`

| 열 | 값의 의미 | PK / FK / 일반 속성 |
| --- | --- | --- |
| `id` | DB 내 학생 행 구분 식별자 | students의 내부 식별자 |
| `student_id` | 학번 | 외부 식별자 |
| `course_id` | 강의 번호 | 외부 식별자 |
| `status` | 등록 상태 | 내부 식별자 |

## 3-3. 입력된 행 수

```text
students 행 수: 3
courses 행 수: 0
enrollments 행 수: 0
```

## 3-4. 내부 식별자와 업무 식별자

```text
students.id가 필요한 이유: 내부 식별자이다. 이는 DB 내에서 행을 안정적으로 구분하기 위한 것이다. 

student_number가 필요한 이유: 외부 식별자이다. 현실 업무에서 대상을 쉽게 식별할 수 있게 한다. 

둘을 항상 같은 값으로 사용하지 않아도 되는 이유: students.id인 primary key는 중복되지 않아야 하고, 비어있지 않도록 자동으로 관리된다. 따라서, 행을 언정적으로 구분할 수 있다. 
```

## 3-5. 숫자처럼 보이는 학번을 문자열로 저장한 이유

```text
나의 설명: '00123' 과 같이 첫 자리에 0이 포함되는 경우가 있을 수 있다. 숫자로 저장하는 경우, 0이 생략되어 오류가 발생할 수 있기 때문에 문자열로 저장한다. 
```

---

# 4. 테이블과 조회 결과는 다르다

## 4-1. 원본 테이블 행 수

```text
ch02_students 전체 행 수: 3
```

## 4-2. 일부 열만 조회

실행 SQL:

```sql
SELECT name, major
FROM ch02_students
ORDER BY id;
```

```text
원본 테이블의 열 수와 조회 결과의 열 수가 다른 이유: 조건을 부여해서 이름과 전공 열만 보이게 하였기 때문이다.
```

## 4-3. 조건을 적용한 조회

실행 SQL:

```sql
SELECT id, student_number, name, major
FROM ch02_students
WHERE major = '컴퓨터공학'
ORDER BY id;
```

```text
원본 테이블 행 수: 3
조회 결과 행 수: 2
원본 테이블의 데이터가 삭제된 것인가?: 아니요. 
그렇게 판단한 이유: 원본 데이터를 삭제한 것이 아니라, select 와 where 이용해서 컴퓨터공학이 전공인 학생만을 표시하도록 하였기 때문입니다. 
```

## 4-4. 정렬 결과 비교

```sql
SELECT id, name
FROM ch02_students
ORDER BY name ASC;

SELECT id, name
FROM ch02_students
ORDER BY name DESC;
```

```text
ASC 결과의 첫 학생: 김민지
DESC 결과의 첫 학생: 이준호

이 실험을 통해 ORDER BY에 대해 알게 된 점: 학생이 정렬되는 순서를 다르게 한 것 입니다. 
```

## 4-5. 증거 화면

권장 경로:

```text
assignments/chapter02/images/step04_result_set.png
```

`여기에 STEP 4 핵심 증거 화면을 삽입하세요.`
<img width="673" height="525" alt="image" src="https://github.com/user-attachments/assets/1e2bd691-985c-4247-b18f-2dc64c644eb0" />


---

# 5. PK와 FK를 실제로 관찰

## 5-1. 정상 데이터의 관계 읽기

다음 SQL 결과를 보고 작성합니다.

```sql
SELECT
    e.id AS enrollment_id,
    s.name AS student_name,
    c.title AS course_title,
    e.status
FROM ch02_enrollments AS e
JOIN ch02_students AS s
    ON s.id = e.student_id
JOIN ch02_courses AS c
    ON c.id = e.course_id
ORDER BY e.id;
```

```text
한 행이 의미하는 것: enrollment (강의 등록) 사건 한 개 

같은 student_id가 여러 enrollment 행에서 반복될 수 있는 이유: 동일한 학생이 각각의 강의에서 등록 여부 값을 가질 수 있기 때문이다. 학생 1명이 여러 수강신청을 가질 수 있기 때문이다. 

같은 course_id가 여러 enrollment 행에서 반복될 수 있는 이유: 동일한 강의가 각각의 학생에 따라 등록 여부 값을 가질 수 있기 때문이다. 강의 하나가 여러 수강신청을 가질 수 있기 때문이다. 
```

## 5-2. 기본키 중복 오류 관찰

중복 PK 입력을 시도한 결과:

```text
실행 성공 / 실패: 실패
오류 메시지에서 확인한 핵심 단어: 중복된 키 값, 고유 제약 조건 위반
왜 실패했다고 생각하는가: id의 값은 중복되면 안되는 것이 고유 조건인데 새 학생이 기존에 있는 김민지 행의 '1'의 ID를 가지도록 입력되었기 때문이다. 
```

## 5-3. 존재하지 않는 학생을 참조하는 FK 오류 관찰

존재하지 않는 `student_id`를 사용한 수강신청 입력 결과:

```text
실행 성공 / 실패:실패
오류 메시지에서 확인한 핵심 단어: 참조키, 제약 조건 위반, 테이블에 없습니다. 
왜 실패했다고 생각하는가:student id 999가 테이블에 없기 때문에 참조되지 못했기 때문이다. 
```

## 5-4. PK와 FK의 차이 정리

```text
PK는 같은 테이블에서 각 행을 고유하게 구분하기 위한 키이다.

FK는 존재하지 않는 행을 참조하지 않도록 방지하기 위한 키이다.

FK 값이 여러 행에서 반복될 수 있는 이유는
여러 번의 같은 값이 반복되는 것은 정상적인 현상이기 때문이다.
```

## 5-5. 증거 화면

권장 경로:

```text
assignments/chapter02/images/step05_pk_fk.png
```

> 오류 메시지는 전체 화면이 아니라 테이블명·constraint·참조 오류가 보이는 정도만 캡처합니다.

`여기에 STEP 5 핵심 증거 화면을 삽입하세요.`
<img width="1244" height="594" alt="image" src="https://github.com/user-attachments/assets/88679266-f299-4fee-9abf-c40187019a18" />

---

# 6. 관계와 카디널리티를 자연어로 설명

현재 임시 데이터 기준으로 작성합니다.

```text
학생 한 명은 여러 수강신청을 가질 수 있는가?: 네
 
강의 한 개는 여러 수강신청을 가질 수 있는가?: 네

수강신청 한 건은 학생 몇 명을 참조하는가?: 한 명

수강신청 한 건은 강의 몇 개를 참조하는가?: 한 개
```

아래 구조를 완성합니다.

```text
students 1 ── N enrollments N ── 1 courses
```

### 학생과 강의가 N:M 관계라고 볼 수 있는 이유

```text
나의 설명: 학생 1명은 N개의 수강신청을 한다. 강의 1개는 M개의 수강신청을 갖는다. 수강신청 1건은 학생 1명, 강의 1개를 참조한다.  
```

> 아직 0개 허용 여부, 필수 관계, 삭제 정책까지 확정하지 않습니다. 그런 규칙은 Chapter 05~06에서 다룹니다.

---

# 7. AI가 만든 테이블 구조 직접 검토

## 7-1. AI에게 묻기 전에 내가 먼저 찾은 문제

다음 구조를 보고 최소 4개를 적습니다.

```sql
CREATE TABLE student_courses (
    student_name VARCHAR(50),
    student_email VARCHAR(100),
    course_title VARCHAR(100),
    instructor_name VARCHAR(50)
);
```

```text
문제 1. students_course 에 대한 테이블이다. 
문제 2. 테이블의 한 행은 student_course 에 대한 정보를 갖는다. 
문제 3. 각 행을 안정적으로 구분하는 pk 가 무엇인지 분명하지 않다.
문제 4. 필수값, 중복, 참조 규칙이 무엇인지 분명하지 않다. 
```

## 7-2. AI 검토 요청 프롬프트

사용한 핵심 프롬프트를 기록합니다.

```text
나는 PostgreSQL과 데이터베이스를 처음 배우는 학생입니다. 아직 정규화와 ERD를 정식으로 배우기 전입니다. 다음 테이블 구조를 검토해 주세요. CREATE TABLE student_courses ( student_name VARCHAR(50), student_email VARCHAR(100), course_title VARCHAR(100), instructor_name VARCHAR(50) ); 완성된 정답 설계를 바로 만들어 주지 말고 다음 질문 중심으로 설명해 주세요. 1. 한 행의 의미가 명확한가? 2. PK 후보가 필요한가? 3. 내부 식별자와 업무 식별자를 구분할 필요가 있는가? 4. FK로 표현해야 할 관계 후보는 무엇인가? 5. 중복 저장 위험이 있는가? 6. 현재 요구사항만으로 결정할 수 없는 정책은 무엇인가? 확정되지 않은 업무 규칙은 임의로 결정하지 마세요.
```

## 7-3. AI 제안과 나의 판단

| AI의 지적 또는 제안 | 동의 / 수정 / 보류 | 나의 근거 |
| --- | --- | --- |
| 한 행의 의미가 완전히 명확하지 않다 | 동의 | 학생이 같은 강좌 다시 수강할 수 있는지, 같은 제목의 강좌 여러개인지 등 AI 가 지적한 정보들에 대한 답이 분명하지 않다. |
| pk 후보가 필요하다 | 동의 | students_course에서 student가 중심인지 course가 중심인지 스스로 명확하게 정리되지 않아서 이를 분명히 해야할 것 같다. |
| 내부 식별자와 업무 식별자를 구분할 필요가 있다. 다만 지금 당장 어떤 열을 추가해야한다고 단정할 단계는 아니다. | 보류 | 담당 교수와의 관계가 아직 명확하지 않음을 지적했는데, 관련하여 결정된 바가 없으므로 보류하였다. |
| 학생, 강좌, 담당자의 관계가 있다. 수강을 나타내는 행에서 학생과 강좌를 가리키는 열이 FK 후보가 된다. | 동의 | 학생을 기준으로 관리하는 것이 타당해보인다. |
| 중복 저장의 위험이 있다. | 동의 | 한 학생이 여러 강좌를 수강하는 경우 이메일과 이름이 매 행 반복되기 때문이다. 어떤 특정 값이 변경되면 관련된 모든 행의 값을 고쳐야한다는 지적이 타당해보인다. |

## 7-4. 본문과 대조한 항목

AI 설명 중 최소 하나를 `chapter02.md`와 비교합니다.

```text
AI가 설명한 내용: PK 후보가 필요하다. student_name은 동명이인이 있을 수 있으므로 일반적으로 적절하지 않다.

본문에서 확인한 내용:두 학생의 이름은 같지만 id가 다르므로 서로 다른 행으로 구분할 수 있습니다.
[출처] Chapter 02. 데이터와 DBMS의 기본 개념|작성자 아토믹데브

일치 / 부분 일치 / 수정 필요: 일치

내가 최종적으로 이해한 내용: 학생 이름 등 겹칠 수 있는 정보가 아니라, 완전히 구별되는 PK를 지정하여, 각 행을 고유하게 구분하는 것이 필요하다. 
```

## 7-5. 증거 화면

권장 경로:

```text
assignments/chapter02/images/step07_ai_review.png
```

`여기에 AI 검토 과정의 핵심 화면을 삽입하세요.`
<img width="997" height="1235" alt="image" src="https://github.com/user-attachments/assets/6f862e4f-58b1-4dff-8bb4-06208fd92018" />

---

# 8. Chapter 01의 개인 서비스 아이디어를 DB 용어로 다시 표현

Chapter 01에서 정한 개인 서비스 주제를 그대로 사용하거나 새 주제를 정해도 됩니다.

## 8-1. 서비스 기본 정보

```text
서비스 이름:운동 관리
서비스 목적:헬스장 회원 관리
```

## 8-2. PostgreSQL 구조 후보

```text
데이터베이스 이름 후보:workout
스키마 이름 후보:member
```

> 아직 실제 데이터베이스나 스키마를 생성하지 않아도 됩니다.

## 8-3. 테이블 후보와 한 행 의미

최소 3개를 작성합니다.

| 테이블 후보 | 한 행의 의미 | 내부 ID 후보 | 업무 식별자 후보 |
| --- | --- | --- | --- |
| member | 회원 한 명 | member | 문자열 |
| trainer | 강사 한 명 | trainer | 문자열 |
| exercise | 운동 참여 횟수 한 건 | exercise | 문자열 |

## 8-4. FK 후보

```text
1. member.trainer → mem.trainer
   이유:회원 기준의 연결 필요

2. member.exercise → mem.exercise
   이유: 회원 기준의 연결 필요
```

## 8-5. 자연어 관계 문장

```text
1.하나의 회원은 강사 데이터를 가질수도, 가지지 않을 수도 있다
2.하나의 회원은 여러 운동 참여 횟수를 가질 수 있다
3.하나의 강사는 여러 운동 참여 횟수와 연결될 수 있다
```

## 8-6. 아직 확정하지 않을 정책

```text
Q1.하나의 회원이 여러 강사 데이터와 연결될 수 있는가
Q2.이용 현황이 1회도 없는 회원을 어떻게 관리할 것인가
Q3.운동 참여 횟수 이외에 시간도 관리할 것인가
```

---

# 9. AI를 개인 구조의 검토자로 사용

## 9-1. 사용한 프롬프트

```text
나는 데이터베이스를 처음 배우는 학생입니다. 아직 ERD나 정규화를 배우기 전입니다. 내가 생각한 서비스는 다음과 같습니다. 서비스: 헬스장 회원 관 목적: 이 서비스는헬스장 회원과 운영을 관리하기 위한 서비스이다.    
| member | 회원 한 명 | member | 문자열 |
| trainer | 강사 한 명 | trainer | 문자열 |
| exercise | 운동 참여 횟수 한 건 | exercise | 문자열 |
## 8-4. FK 후보
```text
1. member.trainer → mem.trainer
   이유:회원 기준의 연결 필요
2. member.exercise → mem.exercise
   이유: 회원 기준의 연결 필요
```
## 9-2. AI가 질문한 내용 중 유용했던 것

```text
1.exercise는 “운동 종류”가 아니라 회원의 운동 참여 1건을 뜻하므로, 이름을 exercise_record 또는 exercise_participation처럼 바꾼다.
2.문자열은 ID 자체라기보다 자료형에 가깝다. 보통 member_id, trainer_id처럼 이름을 붙인다.
3.member.exercise처럼 회원 테이블에 운동 기록을 직접 넣기보다, exercise_record가 회원을 가리키게 한다.
```

## 9-3. AI가 너무 빨리 결정한 내용 또는 내가 보류한 내용

```text
1.현재 관계 문장에 맞는 FK 후보 추천
```

## 9-4. 검토 후 수정한 구조

| 수정 전 | 수정 후 | 수정 이유 |
| --- | --- | --- |
| 자연어 문장 수정:  | 운동 참여 기록 한 건은 한 회원에 속한다. | 운동 참여 기록과 회원 정보 연결 필요 |
| 테이블에 운동 기록을 직접 넣기 | exercise_record가 회원을 가리키게 한다. | 회원 당 여러 운동 데이터 필요 |
| excercise 테이블 | exercise_record 테이블 | 운동 기록이라는 점에서 목적에 더 적합 |

---

# 10. 최종 개념 정리

아래 문장을 본인의 말로 완성합니다.

```text
PostgreSQL은 데이터를 저장하고 SQL을 실행하는 DBMS이다.

DBeaver 또는 psql은 PostgresSQL에 접속해, SQL을 보내고 결과를 보여주는 클라이언트이다.

데이터베이스와 스키마의 차이는 데이터베이스는 관련 데이터가 저장되는 논리적 공간 전체, 스키마는 그 중 객체를 이름으로 구분해 관리하는 이름 공간이라는 것이다.

테이블 한 행은 하나의 객체 (ex. 대상 학생 1명)을 나타내는 것이다.

조회 결과가 원본 테이블과 다른 이유는 일부만을 조회하도록 하는 조건을 추가하였기 때문이다.

내부 식별자와 업무 식별자의 차이는 PK는 한 테이블에서 고유하게 구분하는 것이고, FK는 참조 대상의 키와 연결되는 것이라는 점이다.

PK는 기본키로, 한 테이블 안에서 각 행을 고유하게 구분하는 하나 이상의 열이다.

FK는 외래키로, 다른 테이블 또는 같은 테이블의 참조 대상 키와 연결되는 하나 이상의 열이다.
```

---

# 11. 이번 Chapter에서 새롭게 알게 된 점

최소 3개를 작성합니다.

```text
1. 각각의 테이블마나 PK가 있고, 이는 고유하게 구분된다.
2. 테이블의 내용은 FK를 통해 연결된다. 
3. 여러 개의 대상을 연결할 수 있는데, 학생과 강의가 수강신청 여부를 통해 연결되는 것처럼 양쪽의 행이 서로 연결 되는 경우에는 N:M의 관계가 나타날 수 있다. 
```

## 아직 헷갈리는 내용

```text
1. 어떤 조건을 결정해서 부여해야하는지 분명하게 와닿지 않는다. 
2. 그런 조건들을 부여하기로 결정했다면, 조건의 내용을 어떤식으로 알려줄 수 있는지 궁금하다. 
```

## AI에게 다시 질문하고 싶은 내용

```text

```
앞선 프롬프트 입력하였을 때, PK 후보가 필요하다고 했는데 어떤 것이 적절할지 예시를 들어 설명해달라고 질문하고 싶다. 
---

# 12. 제출 전 자기 점검

- [ ] PostgreSQL에서 현재 database / schema / search_path를 확인했다.
- [ ] DBMS, database, schema, table을 구분해서 설명할 수 있다.
- [ ] TEMP TABLE 3개를 생성하고 직접 데이터를 조회했다.
- [ ] 각 테이블의 한 행 의미를 작성했다.
- [ ] 테이블과 조회 결과가 다르다는 것을 실제 SQL로 확인했다.
- [ ] `ORDER BY`를 사용하지 않으면 업무 순서를 가정하면 안 된다는 점을 이해했다.
- [ ] 내부 식별자와 업무 식별자의 차이를 설명할 수 있다.
- [ ] PK 중복 입력 실패를 직접 확인했다.
- [ ] 존재하지 않는 FK 참조 실패를 직접 확인했다.
- [ ] FK 값이 반복될 수 있는 이유를 설명할 수 있다.
- [ ] AI가 만든 테이블을 내가 먼저 검토했다.
- [ ] AI 설명 중 최소 하나를 본문과 대조했다.
- [ ] 개인 서비스의 테이블 후보를 3개 이상 작성했다.
- [ ] 개인 서비스의 FK 후보와 미확정 정책을 기록했다.
- [ ] 실제 비밀번호·API Key·민감한 접속 정보가 포함되지 않았는지 확인했다.
- [ ] 이미지 링크가 GitHub에서 정상적으로 보이는지 확인했다.

---

# 13. GitHub 제출 정보

답안 파일 권장 위치:

```text
assignments/chapter02/chapter02_answer.md
```

이미지 권장 위치:

```text
assignments/chapter02/images/
```

LMS 제출 URL 형식:

```text
https://github.com/<본인-GitHub-ID>/<본인-저장소>/blob/main/assignments/chapter02/chapter02_answer.md
```

## 최종 확인

- [ ] 위 URL을 로그아웃 상태 또는 다른 브라우저에서 열어도 확인 가능하다.
- [ ] Markdown이 정상 렌더링된다.
- [ ] 이미지가 깨지지 않는다.
- [ ] LMS에 교수자 템플릿 URL이 아니라 **내 답안 파일 URL**을 제출했다.
