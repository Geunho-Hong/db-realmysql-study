# 쿼리 작성 및 최적화 (Insert 이후)

## 11.5 INSERT

- INSERT 문장이 동시에 실행 되는 경우 INSERT 문장 자체 보다는 테이블의 구조가 성능에 더 큰 영향을 미친다
    - 많은 경우 INSERT 와 SELECT 의 성능을 동시에 빠르게 할 수는 없다. 어느 정도 타협하여 테이블 구조를 설계해야 한다

### 11.5.1.1 INSERT IGNORE

- `INSERT IGONRE` 옵션은 PK나 UK 컬럼의 값이 이미 테이블에 중복되는 경우, 저장하는 레코드의 칼람이 테이블의 칼람과 호환되지 않는 경우 모두 무시하고 다음 레코드를 처리 한다
- INSERT 하는 테이블이 PK와 UK를 동시에 가지고 있는 경우 `INSERT IGNORE` 는 두 인덱스 중 하나라도 중복이 발생하는 레코드에 대해 INSERT 를 무시 한다
- 단순히 Key 값의 중복뿐만 아니라 데이터 타입이 일치하지 않는 경우도 컬럼의 기본 값으로 INSERT 하도록 만들 수 있다

```sql
// 사용 예제
INSERT IGONRE INTO salaries VALUES (NULL,NULL,NULL,NULL);
```

### 11.5.1.2 INSERT ... ON DUPLICATE KEY UPDATE

- `INSERT ... ON DUPLICATE KEY UPDATE` 는 PK나 UK의 중복이 발생하면 UPDATE 문장의 역할을 수행 한다
- `REPLACE` 문장은 DELETE와 INSERT의 조합으로 작동 하지만, `INSERT ... ON DUPLICATE KEY` 는 중복된 레코드가 있다면 기존 레코드를 삭제하지 않고 UPDATE 하는 방식으로 작동 한다

```sql
// 사용 예제
INSERT INTO daily_statistic (target_date, stat_name, stat_value)
VALUES( DATE(NOW()), 'VISIT', 1 )
ON DUPLICATE KEY UPDATE stat_value = stat_value + 1;
```

### 11.5.2 LOAD DATA 명령 주의 사항

- `LOAD DATA` 명령은 MySQL 엔진과 스토리지 엔진의 호출 횟수를 최소화 하고 스토리지 엔진이 직접 데이터를 적재하기 때문에 일반적인 INSERT 명령과 비교했을 때 매우 빠르다
- 하지만 `LOAD DATA` 문장은 단일 스레드로 실행되기 때문에 시간이 지날 수록 INSERT 속도는 현저하게 떨어진다
- 따라서 `LOAD DATA` 문장으로 적재할 데이터 파일을 하나 보다 여러개의 파일로 준비해 `LOAD DATA` 문장을 동시에 여러 트랜잭션으로 나눠 실행하는 것이 효율적이다
- 테이블 간 데이터 복사 작업 이라면 구지 `LOAD DATA` 문장 보다 `INSERT ... SELECT` 문장으로 WHERE 조건 절에서 데이터를 부분적으로 잘라 효율적으로 INSERT 하는 것이 좋다

## 11.5.3 성능을 위한 테이블 구조

### 11.5.3.1 대량 INSERT 기능

- 하나의 INSERT 문장으로 수백, 수천 건의 데이터를 INSERT 한 다면 PK 값 기준으로 미리 정렬해서 데이터를 적재하자
    - InnoDB 스토리지 엔진이 PK를 검색해서 레코드를 저장할 위치를 찾을 필요가 없다
- 테이블에 세컨더리 인덱스가 많을 경우 INSERT 성능은 떨어진다. 인덱스는 SELECT 효율은 높이지만 INSERT 효율이 많이 떨어지므로 서비스 사용 용도에 따라 주의해 사용하자

### 11.5.3.2 프라이머리 키 선정

- InnoDB 스토리지 엔진을 사용하는 테이블의 PK는 클러스트링 키 이다. 따라서 세컨더리 인덱스를 이용하는 쿼리 보다 쿼리의 성능이 훨씬 빠르다
- 대부분의 서비스는 INSERT 보다 SELECT 사용 빈도 수가 훨씬 더 많다. 만약 INSERT 가 매우 많이 실행되는 테이블 이라면 키를 단조 증가, 단조 감소하는 패턴의 값을 사용하자
    - 또한 인덱스의 개수를 최소화 하는 것도 좋다

> 테이블을 설계할 때 100 만건 이하의 쿼리를 많이 타지 않는 테이블 이라면 DB 모델링 시 너무 많은 시간을 소모하지 않아도 된다
> 

### 11.5.3.3 Auto-Increment 칼람

- INSERT 효율이 뛰어난 테이블을 만들기 위해서는 2가지가 필요하다
    - 단조 증가 또는 단조 감소되는 값으로 PK 설정
    - 세컨더리 인덱스 최소화
- `SELECT MAX(member_id) FROM ...` 과 같은 쿼리로 최댓값을 조회 하는 쿼리는 매우 위험하다
    - 현재 커넥션뿐만 아니라 다른 커넥션에서 증가 된 AUTO_INCREMENT 값 까지 가져올 수 있으므로 동시성이 보장되지 않아 사용하지 않는 것이 좋다
    - `LAST_INSERT_ID()` 는 다른 커넥션에서 AUTO_INCREMENT 값을 INSERT 해도 현재 커넥션에서 가장 마지막으로 INSERT 된 AUTO_INCREMENT 값 만 반환하기에 안전 하다

> 위 부분은 실무에서 궁금했던 내용이므로 유의해서 기억하자
> 

## 11.6 UPDATE와 DELETE

- 여러 테이블을 조인해서 한 개 이상 테이블의 레코드를 변경하거나 삭제 할 경우 `JOIN UPDATE` , `JOIN DELETE` 구문을 사용한다

### 11.6.1 UPDATE ... ORDER BY ... LIMIT n

- MySQL 에서는 UPDATE나 DELETE 문장에 ORDER BY 절과 LIMIT 절을 동시에 사용해 특정 칼람 으로 정렬해서 상위 몇 건만 변경 및 삭제하는 것도 가능하다
- 단 복제가 구축된 MySQL 서버에서 ORDER BY 가 포함된 UPDATE나 DELETE 문장을 사용 시 복제 소스 서버와 레플리카 서버의 값이 다를 수도 있으므로 유의 하자

```sql
// 사용 예시
DELETE FROM employees ORDER BY last_name limit 10;
```

### 11.6.2 JOIN UPDATE

- 조인된 테이블 중에서 특정 테이블의 칼람값을 다른 테이블의 칼람에 업데이트 할 때 주로 조인 업데이트를 사용 한다
- `JOIN UPDATE` 는 조인되는 모든 테이블에 대해 읽기 참조만 되는 테이블은 읽기 잠금이 걸리고, 칼람이 변경되는 테이블은 쓰기 잠금이 걸린다. 따라서 웹 서비스 같은 OLTP 환경에서는 데드락을 유발할 가능성이 높으므로 빈번하게 사용하는 것은 피하자.
    - 배치나 통계용 UPDATE 문장에서는 유용하게 쓸 수 있다
- `JOIN UPDATE` 문장에서는 GROUP BY나 ORDER BY 절을 쓸 수 없어 서브 쿼리를 이용한 파생 테이블 방법을 사용해야 한다
    - `STRAIGHT_JOIN` 키워드 왼쪽에 있는 테이블이 드라이빙 테이블, 오른쪽에 있는 테이블이 드리븐 테이블이다.

```sql
// 사용 예시
UPDATE tb_test1 t1, employees e
	SET t1.first_name = e.first_name
WHERE e.emp_no = t1.emp_no;
```

### 11.6.4 JOIN DELETE

- `JOIN DELETE` 문장에서는 DELETE 와 FROM 절 사이에 삭제 할 테이블을 명시해야 한다

```sql
// 사용 예제
DELETE e, de
FROM employees e, dept_emp de, departments d
WHERE e.emp_no = de.emp_no AND de.dept_no = d.dept_no AND d.dept_no = 'd001';
```

## 11.7 스키마 조작 (DDL)

### 11.7.1 온라인 DDL

- MySQL 5.5 이전 버전까지는 테이블의 구조를 변경하는 동안 다른 커넥션에서 DML을 실행 할 수 없었다
- MySQL 8.0 버전으로 업그레이드 되면서 대부분의 스키마 변경 작업은 MySQL 서버에 내장된 온라인 DDL 기능으로 처리가 가능해졌다

### 11.7.1.1 온라인 DDL 알고리즘

- MySQL 스키마 알고리즘
    - INSTANT : 테이블의 데이터는 전혀 변경하지 않고 메타데이터만 변경하고 작업을 완료한다. 테이블이 가진 레코드 건수와 무관하게 작업 시간은 매우 짧다. 스키마 변경 도중 테이블의 읽고 쓰기는 대기하게 되지만 스키마 변경 시간이 매우 짧기 때문에 다른 커넥션의 쿼리 처리에는 크게 영향을 미치지 않는다
    - INPLACE : 임시 테이블로 데이터를 복사하지 않고 스키마 변경을 실행한다. 하지만 내부적으로는 테이블의 리빌드를 실행할 수도 있다. 레코드의 복사 작업은 없지만 테이블의 모든 레코드를 리빌드해야 하기 때문에 테이블의 크기에 따라 많은 시간에 소요 될 수도 있다. 하지만 스키마 변경 중에도 테이블의 읽기와 쓰기 모두 가능하다. INPLACE 알고리즘으로 스키마가 변경되는 경우에도 최초 시작 시점과 마지막 종료 시점에는 테이블의 읽고 쓰기가 불가능하다. 하지만 이 시간은 매우 짧기 때문에 다른 커넥션의 쿼리 처리에 대한 영향도는 높지 않다.
    - COPY : 변경된 스키마를 적용한 임시 테이블을 생성하고, 테이블의 레코드를 모두 임시 테이블로 복사한 후 최종적으로 임시 테이블을 RENAME 해서 스키마 변경을 완료한다. 이 방법은 테이블 읽기만 가능하고 DML(INSERT,UPDATE,DELETE)은 실행할 수 없다
    
    > 알고리즘 순서는 INSTANT → INPLACE → COPY 순으로 탐색하며 스키마 변경에 적합한 알고리즘을 찾는다
    > 
    - `INPLACE` 나 `COPY` 알고리즘을 사용하는 경우 3가지 LOCK 기능을 사용할 수 있다
        - NONE : 아무런 잠금을 걸지 않음
        - SHARED : 읽기 잠금을 걸고 스키마 변경을 실행하기 때문에 스키만 변경 중 읽기는 가능하지만 INSERT,UPDATE,DELETE는 불가능 함
        - EXCLUSIVE : 쓰기 잠금을 걸고 스키마 변경을 실행하기 때문에 테이블의 읽고 쓰기가 불가함
    - PK를 추가하는 경우 레코드의 저장 위치가 바뀌어야 하기 때문에 테이블의 리빌드가 필요하지만 단순히 칼람의 이름만 변경하는 경우 INPLACE 알고리즘을 사용하지만 실제 테이블 레코드의 리빌드 작업은 필요치 않다
        - 테이블의 리빌드가 필요한 경우를 데이터 재구성, 테이블 리빌드라고 한다
    - 온라인 DDL이라 하더라도 MySQL 서버에 부하를 유발할 수 있고, 그로 인해 다른 커넥션의 쿼리가 느려질 수 있다. 설령 스키마 변경 작업이 직접 다른커넥션의 DML을 대기하게 만들지는 않더라도 주의해서 사용해야 한다

### 11.7.1.3 INPLACE 알고리즘

- `INPLACE` 알고리즘은 임시 테이블로 레코드를 복사하지는 않더라도 내부적으로 테이블의 모든 레코드를 리빌드 해야 하는 경우가 많다
- `INSTANT` 알고리즘을 사용 하는 경우 거의 시작과 동시에 작업이 완료되기 때문에 작업 도중 실패할 가능성은 거의 없다.
- 그러나 `INPLACE` 알고리즘으로 실행되는 경우 내부적으로 테이블 리빌드 과정이 필요하고 로그 적용 과정이 필요해 중간 과정에서 실패할 가능성이 높다
    - 실패 케이스를 찾아 보고 최대한 온라인 DDL이 실패할 가능성을 낮추는 것이 좋다

## 11.7.2 데이터베이스 변경

- 데이터베이스 단위로 변경하거나 설정하는 DDL 명령은 많지 않다.  DB에 설정할 수 있는 옵션은 기본 문자 집합이나 콜레이션을 설정하는 정도이다

### 11.7.2.1 데이터베이스 생성

- `CREATE DATABASE [IF NOT EXISTS] employees;`
- `IF NOT EXISTS` 키워드를 사용하면 DB가 없는 경우에만 생성하고 이미 있다면 DDL은 무시 된다

### 11.7.2.2 데이터베이스 목록

- `SHOW DATABASES`
    - MySQL 서버가 가지고 있는 데이터베이스 목록을 나열한다
    - `SHOW DATABASES LIKE '%emp%';` 와 같이 사용하여 문자열 검색이 가능하다

### 11.7.2.3 데이터베이스 선택

- `USE employees`
    - 별도로 명시하지 않으면 현재 커넥션의 테이블이나 프로시저를 검색 한다

### 11.7.2.4 데이터베이스 속성 변경

- `ALTER DATABASE employees CHARATER SET=euckr COLLAT=euckr_korean_ci;`
    - 지정한 문자 집합이나 콜레이션을 변경 한다

### 11.7.2.5 데이터베이스 삭제

- `DROP DATABASE [IF EXISTS] employees`
    - 지정한 이름의 데이터 베이스 목록을 삭제한다
    - IF EXISTS 키워드를 사용하면 해당 DB가 존재할 때만 삭제하고 아니라면 명령을 실행하지 않는다

## 11.7.4 테이블변경

### 11.7.4.1 테이블 생성

- `TEMPORARY` 키워드를 사용하면 해당 DB 커넥션에서만 사용 가능한 임시 테이블을 생성 한다

### 11.7.4.2 테이블 구조 조회

- `SHOW CREATE TABLE` , `DESC` 명령으로 MySQL 테이블 구조를 확인할 수 있다
- `SHOW CREATE TABLE` 명령은 칼럼의 목록과 인덱스 , 외래키 정보를 모두 보여주기 때문에 SQL 튜닝 및 테이블 구조를 확인할 때 주로 이 명령을 사용한다

### 11.7.4.3 테이블 구조 변경

- 테이블의 구조를 변경하려면 `ALTER TABLE` 명령을 사용한다
- `ALTER TABLE` 명령은 테이블 자체의 속성을 변경할 수 있을 뿐 아니라 인덱스의 추가 삭제나 칼람을 추가/삭제 하는 용도로도 사용된다

### 11.7.4.4 테이블 명 변경

- MySQL 서버에서 테이블명을 변경하려면 `RENAME TABLE` 명령을 이용하면 된다
- `RENAME TABLE` 명령은 단순히 테이블의 이름 변경 뿐만 아니라 다른 DB로 테이블을 이동할 때도 사용할 수 있다

```sql
// 사용 예제
RENAME TABLE table1 TO table2
RENAME TABLE db1.table1 TO db2.table2;
```

- 배치 테이블 예제

```sql
// 새로운 테이블 및 데이터 생성
CREATE TABLE batch_new (...);
INSERT INTO batch_new SELECT ...;

// 기존 테이블과 교체
RENAME TABLE batch TO batch_old;
RENAME TABLE batch_new TO batch;
```

- 위와 같이 배치 프로그램이 작성되면 기존 테이블과 신규 테이블을 교체하는 동안 일시적으로 batch 테이블이 없어지는 시점이 발생한다
- 이 같은 문제점을 막기 위해 RENAME 명령어를 하나로 합쳐 사용할 수 있다
    - `RENAME TABLE batch TO batch_old , batch_new TO batch`
- RENAME 명령을 하나의 문장으로 묶으면 RENAME TABLE에 명시된 모든 테이블에 대해 잠금을 걸고 이름 변경 작업을 수행하므로 잠깐의 잠금 대기가 발생할 뿐 에러가 발생하지는 않는다

### 11.7.4.5 테이블 상태 조회

- `SHOW TABLE STATUS` 를 통해 테이블 상태를 조회 할 수 있다
- `information_schema` 데이터베이스의 테이블들은 MySQL 서버가 가진 테이블에 대한 다양한 정보를 제공한다

### 11.7.4.7 테이블 삭제

- MySQL 8.0 버전에서는 특정 테이블을 삭제하는 작업이 다른 테이블의 DML이나 쿼리를 직접 방해하지는 않는다
- 테이블이 크다면 직간접적으로 영향을 미칠 수 있기 때문에 서비스 도중에 (DROP TABLE) 은 수행하지 않는 것이 좋다

### 11.7.5.1 칼람 추가

```sql
// 테이블의 제일 마지막에 새로운 칼람을 추가
ALTER TABLE employees ADD COLUMN emp_telno VARHCAR(20), ALGORITHSM=INSTANT;

// 테이블의 중간에 새로운 칼람을 추가
ALTER TABLE employees ADD COLUMN emp_telno VARCHAR(20) AFTER emp_no, ALGORITHSM=INPLACE, 
LOCK=NONE;
```

- 테이블의 마지막에 새로운 칼람을 추가하는 것은 INSTANT 알고리즘으로 즉시 추가가 가능하다
- 그러나 칼람 중간에 값을 추가하는 것은 테이블 리빌드가 필요해 INPLACE 알고리즘으로 처리해야 하므로 테이블이 큰 경우 칼람을 테이블의 마지막 칼람으로 추가하는 것을 권장한다

## 11.7.6 인덱스 변경

- `ALTER TABLE ADD INDEX` 키워드를 통해 인덱스 추가가 가능하다
- `SHOW INDEXS` , `SHOW CREATE TABLE` 명령어를 통해 인덱스 조회가 가능하다
- `ALTER TABLE [테이블명] RENAME INDEX [인덱스1] TO [인덱스2]` 를 통해 인덱스 변경이 가능하다
- `ALTER TABLE ... DROP INDEX ...` 명령어로 인덱스 삭제가 가능하다
    - 주요 키워드만 정리하고 각각의 특징에 대해서는 검색 및 서적을 참고하자

### 11.7.8 프로세스 조회 및 강제 종료

- `SHOW PROCESSLIST` 명령어를 통해 서버에 접속된 사용자 목록이나 각 클라이언트 사용자가 현재 어떤 쿼리를 실행하고 있는지 확인할 수 있다
    - `KILL QUERY [ID]` : 쿼리는 강제 종료시키지만 커넥션은 유지한다
    - `KILL [ID]` : 쿼리뿐만 아니라 해당 커넥션까지 강제 종료한다

## 11.8 쿼리 성능 테스트

- 쿼리가 얼마나 효율적이고 더 개선할 부분이 있는지 확인하려면
    - 1) 실행 계획을 살펴보고 문제될 만한 부분이 있는지 검토한다
    - 2) 특별히 문제될 부분이 없다면 쿼리를 직접 실행한다
- InnoDB 스토리지 엔진은 일반적으로 파일 시스템의 캐시나 버퍼를 거치지 않는 `Direct I/O` 를 사용하므로 운영체제의 캐시가 그다지 큰 영향을 미치지 않는다. 그러나 보다 정확한 성능 테스트를 위해서는 캐시를 제거하고 테스트하자
- 쿼리를 한 번 실행해서 나오는 결과를 그대로 신뢰해서는 안 된다.  6~7번 정도 실행 후 처음 한 두번의 결과는 버리고 나머지 결과의 평균값을 기준으로 비교하는 것이 좋다

> 실제 서비스용 MySQL 서버에서는 현재 테스트 중인 쿼리만 실행되는 것이 아니라 동시에 여러개의 쿼리가 실행 중 이다. 따라서 각 쿼리가 자원을 점유하기 위한 경합 등이 발생하므로 항상 테스트보다는 느린 처리 성능을 보이는 것이 일반적이다
>