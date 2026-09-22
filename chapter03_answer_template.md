# Chapter 03 확장 실습 답안 템플릿

> **과제:** PostgreSQL과 DBeaver로 실습 환경 검증하기  
> **사용 방법:** 이 파일을 내려받아 본인의 GitHub 저장소에 `chapter03_answer.md`라는 이름으로 저장한 뒤 실습하면서 바로 작성합니다.  
> **제출 방법:** LMS에는 파일을 직접 업로드하지 않고, **본인 GitHub 저장소의 `chapter03_answer.md` 파일 URL**을 제출합니다.

---

## 제출 전 보안 주의

이 과제 파일과 캡처 화면에는 다음 정보를 올리지 않습니다.

```text
실제 PostgreSQL 비밀번호
전체 DB 접속 URL
API Key / Token
개인정보
공개할 필요가 없는 사내 서버 주소
```

LMS에서 제출자를 확인할 수 있으므로 공개 저장소의 답안 파일에 학번이나 실명을 반드시 적을 필요는 없습니다.

```text
GitHub 계정 또는 별칭: yeun0512
과제 작성일: 2026.09.22
사용한 AI 도구: GPT
```

---

# 1. PostgreSQL과 DBeaver 환경 확인

## 1-1. 내 환경

| 항목 | 작성 내용 |
| --- | --- |
| 운영체제 | 윈도우 |
| PostgreSQL 버전 | 18 |
| DBeaver 버전 | DBeaver 26.2.0 |
| Host | localhost |
| Port | 5432 |
| Database | PostgresSQL |
| Username | Postgres |

> 비밀번호는 기록하지 않습니다.

## 1-2. PostgreSQL과 DBeaver 역할 설명

```text
PostgreSQL은: 데이터베이스이다. 

DBeaver는:클라이언트 서버로, 데이터베이스 서버에 연결하기 위해 필요하다.

두 프로그램의 차이는: DBeaver은 쿼리를 작성하기 위한 것이고, DBeaver에 입력하면 PostgresSQL에 전달되어 실행된다. 다시 결과가 DBeaver에 표시된다. 
```

---

# 2. 연결 테스트와 첫 SQL

## 2-1. DBeaver 연결 결과

- [0] PostgreSQL 연결 유형 선택
- [0] Host 확인
- [0] Port 확인
- [0] Database 확인
- [0] Username 확인
- [0] Test Connection 성공

### 연결 성공 화면

권장 이미지 경로:

```text
assignments/chapter03/images/step02_connection.png
```

<img width="952" height="635" alt="image" src="https://github.com/user-attachments/assets/05397b38-6de2-47f2-985c-d8f1ecf0bc64" />


## 2-2. 첫 SQL 실행

```sql
SELECT 1 + 1 AS result;
```

실행 전 예상:

```text
2
```

실제 결과:

```text
2
```

이 결과가 의미하는 것:

```text
1+1=2
```

---

# 3. 현재 연결 위치를 SQL로 검증

다음 SQL을 실행합니다.

```sql
SELECT version();
SELECT current_database();
SELECT current_user;
SELECT current_schema();
SHOW search_path;
SHOW transaction_read_only;
SHOW TimeZone;
```

## 3-1. 결과 기록

| 확인 항목 | 실제 결과 | 내가 이해한 의미 |
| --- | --- | --- |
| `version()` | PostgreSQL 18.6 on x86_64-windows, compiled by msvc-19.44.35228, 64-bit | postgres의 버전 |
| `current_database()` | postgres | 현재 사용중인 데이터베이스가 postgres임 |
| `current_user` | postgres | 유저 이름 |
| `current_schema()` | public | 스키마 |
| `search_path` | public, "$user" | public의 user로 연결됨 |
| `transaction_read_only` | off | 읽기모드 아님 |
| `TimeZone` | Asia/Seoul | 아시아, 서울에서의 접속 |

## 3-2. 반드시 설명할 것

### DBeaver 연결 이름과 `current_database()`는 왜 같은 개념이 아닌가요?

```text
연결 이름은 임의로 지정할 수 있지만, 명령어를 사용하여 확인한 것은 실제 연결된 데이터베이스의 이름이다. 
```

### `current_schema()`와 `search_path`는 어떤 관계가 있나요?

```text
current schema는 검색 경로에서 사용할 수 있는 첫 번째 스키마로, search path는 검색 경로 전체이나 current schema 는 그 중 일부라는 관계가 있습니다. 
```

### `transaction_read_only = off`라는 결과만으로 모든 테이블을 만들 권한이 있다고 단정할 수 있나요?

```text
아니요. 읽기모드가 아니라는 것을 나타낼 뿐입니다. 
```

## 3-3. 증거 화면

권장 경로:

```text
assignments/chapter03/images/step03_location_check.png
```
<img width="1015" height="465" alt="image" src="https://github.com/user-attachments/assets/ae4bcfce-7642-4f78-8c52-b9e2bc882eeb" />

---

# 4. `ai_database_book` 데이터베이스 확인

## 4-1. 현재 데이터베이스

```sql
SELECT current_database();
```

실제 결과:

```text
ai_database_book
```

- [0] 결과가 `ai_database_book`이다.
- [0] 다른 DB라면 올바른 연결로 전환했다.

## 4-2. 연결을 바꾼 뒤 다시 검증

```text
전환 전 데이터베이스:Postgres
전환 후 데이터베이스:ai_database_book
전환 여부를 판단한 근거:select current_database() 를 통해 확인했다. 
```

### 화면에서 보이는 연결 이름만 믿지 않고 SQL을 다시 실행해야 하는 이유

```text
데이터베이스를 생성한다고 바로 연결되는 것이 아니다. 
```

---

# 5. SQL 실행 범위 실험

SQL Editor에 다음 세 문장을 입력합니다.

```sql
SELECT 'A' AS step;
SELECT 'B' AS step;
SELECT 'C' AS step;
```

## 5-1. 한 문장 실행

```text
내가 실행한 문장: 마지막 문장
실제 결과: C
```

## 5-2. 선택 영역 실행

```text
선택한 문장: 1, 2 문장
실제 결과: result 1, result 2 탭이 2개로, 각각의 결과 A, B 가 나왔다.  
```

## 5-3. 전체 스크립트 실행

```text
실제 결과: result 결과 탭 3개로, 각각 A, B, C가 나왔다. 
결과 탭 또는 실행 순서에서 관찰한 점: 선택한 영역에 따라 실행 결과 개수가 달라진다. 
```

## 5-4. 결과 해석

```text
한 문장 실행과 전체 스크립트 실행의 차이: 한 문장 실행은 한 문장을 범위로 하지만, 전체 스크립트 실행은 전체 문장을 범위로 한다. 

변경 SQL에서 실행 범위를 잘못 선택하면 위험한 이유: select 문장과 update 나 delete 등이 섞여 있을 때, 실행 범위가 잘못 설정되면 데이터의 변경이 발생할 수 있기 때문이다. 
```

### 증거 화면

권장 경로:

```text
assignments/chapter03/images/step05_execution_scope.png
```

<img width="782" height="327" alt="image" src="https://github.com/user-attachments/assets/94bc407f-48e2-4883-96e1-d313e79ac92f" />

---

# 6. 제공된 환경 확인 SQL 실행

Public 저장소의 Chapter 03 파일을 사용합니다.

```text
code/chapter03/setup_check.sql
code/chapter03/setup_validate_local.sql
```

## 6-1. `setup_check.sql`

실행 결과에서 확인한 항목:

```text
PostgreSQL 버전: PostgreSQL 18.6 on x86_64-windows, compiled by msvc-19.44.35228, 64-bit
현재 DB:ai_database_book
현재 사용자:postgres
현재 스키마:public
search_path:public, "$user"
읽기 전용 여부:off
TimeZone:Asia/Seoul
1 + 1 결과:2
public 스키마 존재 여부:0
public USAGE 권한:0
public CREATE 권한:0
```

### 이 파일을 여러 번 실행해도 비교적 안전한 이유

```text
데이터를 변형하는 drop, delete 등 문장이 없기 때문이다. 
```

## 6-2. `setup_validate_local.sql`

```text
실행 결과:Chapter 03 recommended local environment validation passed
PASS / FAIL: Pass
```

실패했다면 실패 항목:

```text
없다. 
```

그 실패가 실제 문제인지 환경 차이인지 판단한 근거:

```text
없다. 
```

---

# 7. 안전한 오류 진단 실습

실제 오류가 있었다면 그 오류를 사용합니다. 오류가 없었다면 **데이터를 삭제하거나 서버를 강제로 중지하지 말고**, 안전한 SQL 문법 오류를 하나 만들어 관찰합니다.

예:

```sql
SELEC 1;
```

> 오류를 확인한 뒤 올바른 `SELECT 1;`로 복구합니다.

## 7-1. 오류 기록

```text
오류 메시지 핵심 문장:
구문 오류, "SELEC" 부근

Error position: line: 1

내가 먼저 생각한 원인 1: 1번 줄에서 문제가 생김. 

내가 먼저 생각한 원인 2: 구문 오류인 바, 오타가 있을 것임. 

실제로 확인한 방법: 실제로 1번 줄을 확인하였음. 

실제 원인: 1번 줄에서 SELECT 대신 SELEC 라고 잘못 입력했기 때문이다. 

수정한 내용: SELECT 1;로 다시 입력하였다. 
```

## 7-2. 수정 후 재검증

```sql
SELECT 1;
SELECT current_database();
```

```text
재검증 결과:ai_database_book
```

## 7-3. 오류를 유형으로 분류

- [ ] 서버 실행 문제
- [ ] Host 문제
- [ ] Port 문제
- [ ] Database 문제
- [ ] Username/인증 문제
- [0] SQL 문법 문제
- [ ] 권한 문제
- [ ] 기타

선택 이유:

```text
SQL 문법에서 SELECT; 입력 중 오타가 발생하여 오류가 생긴 것이기 때문이다.
```

---

# 8. AI를 오류 분석 보조 도구로 사용

## 8-1. AI에게 전달한 프롬프트

비밀번호·개인정보·전체 접속 URL은 제거하고 기록합니다.

```text
오류 화면 사진을 첨부하고, 아래 프롬프트 입력하였다. 
나는 PostgreSQL과 DBeaver를 처음 배우는 학생입니다.
아래 오류를 바로 하나의 원인으로 단정하지 말고,
초보자가 안전하게 확인할 순서대로 분석해 주세요.
다음 형식으로 설명해 주세요.

1. 오류 메시지에서 확인되는 사실
2. 가능한 원인 후보
3. 각 원인을 확인하는 안전한 방법
4. 확인 결과에 따라 다음에 할 행동
5. 실행하면 위험할 수 있어 피해야 할 명령
실제 비밀번호나 개인정보는 포함하지 않았습니다.
[오류 메시지 붙여넣기]
```

## 8-2. AI 답변 검토

| AI가 제안한 확인 방법 | 실제로 확인했는가? | 결과 | 수용 / 수정 / 거절 |
| --- | --- | --- | --- |
| 아래처럼 첫 줄의 철자를 고칩니다. | 네. | 오류가 발생하지 않았다. | 수용 |

### AI가 오류 원인을 너무 빨리 단정한 부분이 있었나요?

```text
아니요. 오류를 고치는 방법으로 한 가지를 제안하고, 고쳐지지 않았을 경우의 대안들을 제시하였습니다. 
```

### 오류 메시지와 실제 환경 중 무엇을 확인해서 최종 판단했나요?

```text
실제 환경
```

### AI 활용에서 가장 유용했던 점

```text
오류가 어디에서 발생했는지만을 알려주는 것이 아니라, 어떻게 다시 입력해야하는지까지 알려준다는 점이 유용했다. 
```

### AI 답변을 그대로 실행하지 않고 확인해야 하는 이유

```text
실제 환경에서 오류 없이 실행되는지를 검토해야하기 때문이다. 
```

---

# 9. Chapter 01~02 개인 서비스와 연결

앞에서 선택한 개인 서비스가 PostgreSQL을 사용한다고 가정합니다.

```text
서비스 이름: 헬스장 회원관리

사용할 데이터베이스 이름 후보: workout

사용할 스키마 이름 후보: member

앞으로 만들고 싶은 테이블 후보 3개:
1.
2.
3.
```

### 아직 SQL을 만들지 않고 이름과 역할만 정하는 이유

```text

```

### Chapter 02에서 정리했던 한 행의 의미 중 수정할 부분이 있나요?

```text

```

---

# 10. 초보자용 연결 가이드 작성

친구가 자신의 PC에서 같은 실습을 시작한다고 가정합니다. 아래 순서를 자신의 말로 작성합니다.

```text
1. PostgreSQL 서버가 실행되는지 확인하는 방법:

2. DBeaver에서 PostgreSQL 연결을 만드는 방법:

3. Host / Port / Database / Username의 의미:

4. ai_database_book에 연결되었는지 확인하는 방법:

5. 현재 위치를 확인하는 SQL:

6. 한 문장과 전체 스크립트 실행을 구분해야 하는 이유:

7. 비밀번호를 GitHub나 AI 프롬프트에 넣으면 안 되는 이유:
```

---

# 11. 최종 성찰

아래 문장은 반드시 본인의 말로 작성합니다.

```text
1. DBeaver와 PostgreSQL의 가장 중요한 차이는
   ____________________________________________________________ 이다.

2. 내가 지금 어느 데이터베이스에 연결되어 있는지 확인할 때
   화면 이름만 보지 않고 ______________________________________ 해야 한다.

3. PostgreSQL 오류가 발생했을 때 가장 먼저 해야 할 일은
   ____________________________________________________________ 이다.

4. AI를 오류 해결에 사용할 때 가장 중요한 것은
   ____________________________________________________________ 이다.
```

---

# 12. 제출 체크리스트

- [ ] `chapter03_answer.md`의 빈 필수 항목을 작성했다.
- [ ] PostgreSQL과 DBeaver의 역할 차이를 설명했다.
- [ ] `current_database/current_user/current_schema/search_path`를 실제로 확인했다.
- [ ] `ai_database_book` 연결 여부를 SQL로 검증했다.
- [ ] SQL 실행 범위 세 가지를 비교했다.
- [ ] `setup_check.sql`을 실행했다.
- [ ] `setup_validate_local.sql` 결과를 확인했다.
- [ ] 오류 원인을 먼저 스스로 추정한 뒤 AI를 사용했다.
- [ ] AI 제안을 실제 환경에서 검증했다.
- [ ] 핵심 캡처 3~4장만 골라 넣었다.
- [ ] 캡처에 비밀번호·개인정보·전체 접속 URL이 없다.
- [ ] Markdown 이미지가 GitHub 웹 화면에서 실제로 보인다.
- [ ] 최종 답안 파일을 commit/push했다.

---

# 13. LMS 제출 URL

아래 형식의 **본인 GitHub 파일 URL**을 LMS에 제출합니다.

```text
https://github.com/<본인-GitHub-ID>/<본인-저장소>/blob/main/assignments/chapter03/chapter03_answer.md
```

내 제출 URL:

```text

```

> 저장소 메인 URL, 교수자 템플릿 URL, Raw URL이 아니라 **작성 완료된 본인 `chapter03_answer.md` 파일 화면 URL**을 제출합니다.
