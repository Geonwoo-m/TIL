# Database 기초 개념 정리

---

## Relational Database

- 데이터를 테이블(표) 형태로 저장하는 데이터베이스
- 테이블 간의 관계를 통해 데이터를 연결
- 대표적인 RDBMS: MySQL, PostgreSQL, Oracle, MariaDB

### Relation
```
릴레이션 = 테이블
튜플     = 행 (row)
어트리뷰트 = 열 (column)
```

---

## Key 종류

| 키 | 설명 |
|------|------|
| 기본키 (Primary Key) | 튜플을 유일하게 식별하는 키, NULL 불가, 중복 불가 |
| 외래키 (Foreign Key) | 다른 테이블의 기본키를 참조하는 키, 테이블 간 관계 표현 |
| 후보키 (Candidate Key) | 기본키가 될 수 있는 키들 |
| 유니크키 (Unique Key) | 중복 불가, NULL 허용 |

---

## 데이터 타입

| 타입 | 설명 |
|------|------|
| `INT` | 정수 |
| `BIGINT` | 큰 정수 |
| `DECIMAL(p, s)` | 고정 소수점 |
| `FLOAT / DOUBLE` | 부동 소수점 |
| `CHAR(n)` | 고정 길이 문자열 |
| `VARCHAR(n)` | 가변 길이 문자열 |
| `DATE` | 날짜 (YYYY-MM-DD) |
| `DATETIME` | 날짜 + 시간 |
| `TIMESTAMP` | 날짜 + 시간 (시간대 자동 변환) |
| `BOOLEAN` | 참/거짓 |

---

## Constraints (제약 조건)

| 제약 조건 | 설명 |
|------|------|
| `NOT NULL` | NULL 값 허용 안함 |
| `UNIQUE` | 중복 값 허용 안함 |
| `PRIMARY KEY` | NOT NULL + UNIQUE |
| `FOREIGN KEY` | 다른 테이블 기본키 참조 |
| `DEFAULT` | 기본값 설정 |
| `CHECK` | 특정 조건 만족하는 값만 허용 |

---

## Foreign Key 옵션 (ON DELETE / ON UPDATE)

참조하는 테이블의 데이터가 삭제/수정될 때 어떻게 처리할지 설정

| 옵션 | 설명 |
|------|------|
| `CASCADE` | 부모 행 삭제/수정 시 자식 행도 함께 삭제/수정 |
| `SET NULL` | 부모 행 삭제/수정 시 자식 행의 FK를 NULL로 변경 |
| `RESTRICT` | 자식 행이 존재하면 부모 행 삭제/수정 불가 (기본값) |
| `SET DEFAULT` | 부모 행 삭제/수정 시 자식 행의 FK를 기본값으로 변경 |
| `NO ACTION` | RESTRICT와 동일 |
```sql
CREATE TABLE employee (
    id       INT PRIMARY KEY,
    dept_id  INT,
    FOREIGN KEY (dept_id) REFERENCES department(id)
        ON DELETE CASCADE      -- 부서 삭제 시 직원도 삭제
        ON UPDATE SET NULL     -- 부서 수정 시 dept_id를 NULL로
);
```

> CASCADE는 편리하지만 의도치 않은 대량 삭제가 발생할 수 있으니 주의

## 테이블 생성
```sql
CREATE TABLE employee (
    id        INT PRIMARY KEY,
    name      VARCHAR(50) NOT NULL,
    dept_id   INT,
    salary    DECIMAL(10, 2) DEFAULT 0,
    FOREIGN KEY (dept_id) REFERENCES department(id)
);
```

---

## INSERT / UPDATE / DELETE
```sql
-- INSERT
INSERT INTO employee (id, name, salary) VALUES (1, 'kim', 50000);

-- UPDATE
UPDATE employee SET salary = 60000 WHERE id = 1;

-- DELETE
DELETE FROM employee WHERE id = 1;
```

---

## SELECT 기본 문법
```sql
SELECT name, salary
FROM employee
WHERE salary >= 50000
ORDER BY salary DESC;
```

### 자주 쓰는 키워드
| 키워드 | 설명 |
|------|------|
| `WHERE` | 조건 필터링 |
| `ORDER BY` | 정렬 (ASC 오름차순 / DESC 내림차순) |
| `LIMIT` | 출력 행 수 제한 |
| `DISTINCT` | 중복 제거 |
| `BETWEEN` | 범위 조건 |
| `LIKE` | 패턴 매칭 (`%`, `_`) |
| `IN` | 여러 값 중 하나 |
| `IS NULL` | NULL 여부 확인 |

---

## 서브쿼리

쿼리 안에 또 다른 쿼리를 사용하는 것
```sql
-- IN: 서브쿼리 결과 목록 중 하나와 일치하면 true
SELECT name FROM employee
WHERE dept_id IN (SELECT id FROM department WHERE name = 'IT');

-- EXISTS: 서브쿼리 결과가 존재하면 true
SELECT name FROM employee e
WHERE EXISTS (SELECT 1 FROM project WHERE emp_id = e.id);

-- ANY: 서브쿼리 결과 중 하나라도 조건 만족하면 true
SELECT name FROM employee
WHERE salary > ANY (SELECT salary FROM employee WHERE dept_id = 2);

-- ALL: 서브쿼리 결과 모두 조건 만족해야 true
SELECT name FROM employee
WHERE salary > ALL (SELECT salary FROM employee WHERE dept_id = 2);
```

---

## NULL과 Three-Valued Logic

- NULL = 값이 없음 / 알 수 없음
- NULL과의 비교 연산은 항상 UNKNOWN 반환
- SQL은 TRUE / FALSE / UNKNOWN 세 가지 값으로 동작
```sql
-- NULL 비교는 = 가 아닌 IS NULL 사용
SELECT * FROM employee WHERE salary IS NULL;
SELECT * FROM employee WHERE salary IS NOT NULL;
```

| 연산 | 결과 |
|------|------|
| NULL = NULL | UNKNOWN |
| NULL IS NULL | TRUE |
| NULL + 1 | NULL |
| WHERE 조건이 UNKNOWN | 해당 행 제외 |

---

## JOIN

두 개 이상의 테이블을 연결해서 데이터를 조회하는 것
```sql
-- INNER JOIN: 두 테이블 모두 일치하는 행만
SELECT e.name, d.name
FROM employee e
INNER JOIN department d ON e.dept_id = d.id;

-- LEFT JOIN: 왼쪽 테이블 전체 + 오른쪽 일치하는 행
SELECT e.name, d.name
FROM employee e
LEFT JOIN department d ON e.dept_id = d.id;

-- RIGHT JOIN: 오른쪽 테이블 전체 + 왼쪽 일치하는 행
SELECT e.name, d.name
FROM employee e
RIGHT JOIN department d ON e.dept_id = d.id;

-- FULL OUTER JOIN: 양쪽 테이블 전체
SELECT e.name, d.name
FROM employee e
FULL OUTER JOIN department d ON e.dept_id = d.id;

-- CROSS JOIN: 두 테이블의 모든 조합
SELECT e.name, d.name
FROM employee e
CROSS JOIN department d;
```

| JOIN 종류 | 설명 |
|------|------|
| INNER JOIN | 두 테이블 모두 일치하는 행만 |
| LEFT JOIN | 왼쪽 전체 + 오른쪽 일치 행 |
| RIGHT JOIN | 오른쪽 전체 + 왼쪽 일치 행 |
| FULL OUTER JOIN | 양쪽 전체 |
| CROSS JOIN | 모든 조합 (카테시안 곱) |

---

## GROUP BY / HAVING / Aggregate Function
```sql
SELECT dept_id, COUNT(*), AVG(salary)
FROM employee
GROUP BY dept_id
HAVING AVG(salary) >= 50000;
```

> WHERE는 그룹화 전 필터링, HAVING은 그룹화 후 필터링

### Aggregate Function (집계 함수)
| 함수 | 설명 |
|------|------|
| `COUNT(*)` | 행 개수 |
| `SUM(col)` | 합계 |
| `AVG(col)` | 평균 |
| `MAX(col)` | 최댓값 |
| `MIN(col)` | 최솟값 |

---

## 느낀 점
- 처음엔 잘 알고 있는 것 같았지만, 복습을 하면서 특히 JOIN이나 서브쿼리 부분을 들어가면서부터 제대로 이해하지 못하고 있었다는 걸 깨달았다.
- NULL과 Three-Valued Logic은 단순히 값이 없다는 개념이 아니라 UNKNOWN이라는 세 번째 상태가 존재한다는 게 인상깊었다. WHERE 조건에서 UNKNOWN이면 해당 행이 제외된다는 것도 놓치기 쉬운 포인트였다.
- JOIN은 종류가 많아서 헷갈렸는데, 결국 어느 테이블을 기준으로 데이터를 합치느냐의 차이라는 걸 이해하고 나니 구분이 됐다.
- CASCADE나 SET NULL 같은 FK 옵션은 실무에서 테이블 설계할 때 중요한 부분이므로 잘 기억해둬야겠다.
