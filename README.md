# db-realmysql-study

## 스터디 소개

    Real MySQL의 핵심 내용을 읽고 정리한 스터디 입니다.
     * 참여자: m3252,SujungRim,YunNote,GuenhoHong
     * 기간: 2021.09.12 ~


### 트랜잭션과 잠금
<details>
    <summary>5.1 트랜잭션</summary>
    <div markdown="1">
        - 5.1.1 MySQL 에서의 트랜잭션 <br/>
        - 5.1.2 주의 사항 <br/>
    </div>
</details>
<details>
    <summary>5.2 MySQL 엔진의 잠금</summary>
    <div markdown="1">
        - 5.2.1 글로벌 락 <br/>
        - 5.2.2 테이블 락 <br/>
        - 5.2.3 네임드 락 <br/>
        - 5.2.4 메타데이터 락 <br/>
    </div>
</details>
<details>
    <summary>5.3 InnoDB 스토리지 엔진 잠금</summary>
    <div markdown="1">
        - 5.3.1 InnoDB 스토리지 엔진의 잠금 <br/>
        - 5.3.2 인덱스와 잠금 <br/>
        - 5.3.3 레코드 수준의 잠금 확인 및 해제 <br/>
    </div>
</details>
<details>
    <summary>5.4 MySQL의 격리 수준</summary>
    <div markdown="1">
        - 5.4.1 READ UNCOMMITTED <br/>
        - 5.4.2 READ COMMITTED <br/>
        - 5.4.3 REPEATABLE READ <br/>
        - 5.4.4 SERIALIZABLE <br/>
    </div>
</details>


### 인덱스

<details>
    <summary>8.1 디스크 읽기 방식</summary>
    <div markdown="1">
        - 8.1.1 하드 디스크 드라이버(HDD)와 솔리드 스테이트 드라이버(SSD) <br/>
        - 8.1.2 랜덤 I/O와 순차 I/O <br/>
    </div>
</details>
<details>
    <summary>8.2 인덱스란?</summary>
    <div markdown="1">
        - Summary : 
    </div>
</details>
<details>
    <summary>8.3 B-Tree 인덱스</summary>
    <div markdown="1">
        - 8.3.1 구조 및 특성 <br/>
        - 8.3.2 B-Tree 인덱스 키 추가 및 삭제 <br/>
        - 8.3.3 B-Tree 인덱스 사용에 영향을 미치는 순서 <br/>
        - 8.3.4 B-Tree 인덱스를 통한 데이터 읽기 <br/>
        - 8.3.5 다중 컬럼(Multi-column 인덱스) <br/>
        - 8.3.6 B-Tree 인덱스의 정렬 및 스캔 방향 <br/>
        - 8.3.7 B-Tree 인덱스의 가용성과 효율성 <br/>
    </div>
</details>
<details>
    <summary>8.4 R-Tree 인덱스</summary>
    <div markdown="1">
        - 8.4.1 구조 및 특성<br/>
        - 8.4.2 R-Tree 인덱스의 용도<br/>
    </div>
</details>
<details>
    <summary>8.5 전문 검색 인덱스</summary>
    <div markdown="1">
        - 8.5.1 인덱스 알고리즘 <br/>
        - 8.5.2 전문 검색 인덱스의 가용성<br/>
    </div>
</details>
<details>
    <summary>8.6 함수 기반 인덱스</summary>
    <div markdown="1">
        - 8.6.1 가상 칼람을 이용한 인덱스<br/>
        - 8.6.2 함수를 이용한 인덱스 <br/>
    </div>
</details>
<details>
    <summary>8.7 멀티 밸류 인덱스</summary>
</details>
<details>
    <summary>8.8 클러스터링 인덱스</summary>
    <div markdown="1">
        - 8.8.1 클러스터링 인덱스 <br/>
        - 8.8.2 세컨더리 인덱스에 미치는 영향 <br/>
        - 8.8.3 클러스터링 인덱스의 장점과 단점 <br/>
        - 8.8.4 클러스터링 테이블 사용 시 주의사항 <br/>
    </div>
</details>
<details>
    <summary>8.9 유니크 인덱스</summary>
    <div markdown="1">
        - 8.9.1 유니크 인덱스와 일반 세컨더리 인덱스의 비교 <br/>
        - 8.9.2 유니크 인덱스 사용 시 주의사항 <br/>
    </div>
</details>
<details>
    <summary>8.10 외래키</summary>
    <div markdown="1">
        - 8.10.1 자식 테이블의 변경이 대기하는 경우 <br/>
        - 8.10.2 부모 테이블의 변경 작업이 대기하는 경우 <br/>
    </div>
</details>

### 옵티마이저와 힌트
<details>
    <summary>9.1 개요</summary>
    <div markdown="1">
        - 9.1.1 쿼리 실행 절차 <br/>
        - 9.1.2 옵티마이저의 종류 <br/>
    </div>
</details>
<details>
    <summary>9.2 기본 데이터 처리</summary>
    <div markdown="1">
        - 9.2.1 풀 테이블 스캔과 풀 인덱스 스캔 <br/>
        - 9.2.2 병렬 처리 <br/>
        - 9.2.3 ORDER BY 처리 (Using filesort) <br/>
        - 9.2.4 GROUP BY 처리 <br/>
        - 9.2.5 DISTINCT 처리 <br/>
        - 9.2.6 내부 임시 테이블 활용 <br/>
    </div>
</details>
<details>
    <summary>9.3 고급 최적화</summary>
    <div markdown="1">
        - 9.3.1 옵티마이저 스위치 옵션 <br/>
        - 9.3.2 조인 최적화 알고리즘 <br/>
    </div>
</details>
<details>
    <summary>9.4 쿼리 힌트</summary>
    <div markdown="1">
        - 9.4.1 인덱스 힌트 <br/>
        - 9.4.2 옵티마이저 힌트 <br/>
    </div>
</details>

### 실행 계획
<details>
    <summary>10.1 통계 정보</summary>
    <div markdown="1">
        - 10.1.1 테이블 및 인덱스 통계 정보 <br/>
        - 10.1.2 히스토그램 <br/>
        - 10.1.3 코스트 모델 (Cost Model) <br/>
    </div>
</details>
<details>
    <summary>10.2 실행 계획 확인</summary>
    <div markdown="1">
        - 10.2.1 실행 계획 출력 포맷 <br/>
        - 10.2.2 쿼리의 실행 시간 확인 <br/>
    </div>
</details>
<details>
    <summary>10.3 실행 계획 분석</summary>
    <div markdown="1">
        - 10.3.1 id 칼럼 <br/>
        - 10.3.2 select_type 칼럼 <br/>
        - 10.3.3 table 칼람 <br/>
        - 10.3.4 partitions 칼람 <br/>
        - 10.3.5 type 칼람 <br/>
        - 10.3.6 possible_keys 칼람 <br/>
        - 10.3.7 key 칼람 <br/>
        - 10.3.8 key_len 칼람 <br/>
        - 10.3.9 ref 칼람 <br/>
        - 10.3.10 rows 칼람 <br/>
        - 10.3.11 filtered 칼람 <br/>
        - 10.3.12 Extra 칼람 <br/>
    </div>
</details>

### 쿼리 작성 및 최적화
<details>
    <summary>11.1 쿼리 작성과 연관된 시스템 변수</summary>
    <div markdown="1">
        - 11.1.1 SQL 모드 <br/>
        - 11.1.2 영문 대소문자 구분 <br/>
        - 11.1.3 MySQL 예약어 <br/>
    </div>
</details>
<details>
    <summary>11.2 매뉴얼의 SQL 문법 표기를 읽는 방법</summary>
</details>
<details>
    <summary>11.3 MySQL 연산자와 내장 함수</summary>
    <div markdown="1">
        - 11.3.1 리터럴 표기법 문자열 <br/>
        - 11.3.2 MySQL 연산자 <br/>
        - 11.3.3 MySQL 내장 함수 <br/>
    </div>
</details>
<details>
    <summary>11.4 SELECT</summary>
    <div markdown="1">
        - 11.4.1 SELECT 절의 처리 순서 <br/>
        - 11.4.2 WHERE 절과 GROUP BY 절, ORDER BY 절의 인덱스 사용 <br/>
        - 11.4.3 WHERE 절의 비교 조건 사용 시 주의 사항 <br/>
        - 11.4.4 DISTINCT <br/>
        - 11.4.5 LIMIT n <br/>
        - 11.4.6 COUNT() <br/>
        - 11.4.7 JOIN <br/>
        - 11.4.8 GROUP BY <br/>
        - 11.4.9 ORDER BY <br/>
        - 11.4.10 서브 쿼리 <br/>
        - 11.4.11 CTE (Common Table Expression) <br/>
        - 11.4.12 윈도우 함수 (Window Function) <br/>
        - 11.4.13 잠금을 사용하는 SELECT <br/>
    </div>
</details>
<details>
    <summary>11.5 INSERT</summary>
    <div markdown="1">
        - 11.5.1 고급 옵션 <br/>
        - 11.5.2 LOAD DATA 명령 주의 사항 <br/>
        - 11.5.3 성능을 위한 테이블 구조 <br/>
    </div>
</details>
<details>
    <summary>11.6 UPDATE와 DELETE</summary>
    <div markdown="1">
        - 11.6.1 UPDATE ... ORDER BY ... LIMIT n <br/>
        - 11.6.2 JOIN UPDATE <br/>
        - 11.6.3 여러 레코드 UPDATE <br/>
        - 11.6.4 JOIN DELETE <br/>
    </div>
</details>
<details>
    <summary>11.7 스키마 조작 (DDL)</summary>
    <div markdown="1">
        - 11.7.1 온라인 DDL <br/>
        - 11.7.2 데이터베이스 변경 <br/>
        - 11.7.3 테이블 스페이스 변경 <br/>
        - 11.7.4 테이블 변경 <br/>
        - 11.7.5 칼람 변경 <br/>
        - 11.7.6 인덱스 변경 <br/>
        - 11.7.7 테이블 변경 묶음 실행 <br/>
        - 11.7.8 프로세스 조회 및 강제 종료 <br/>
        - 11.7.9 활성 트랜잭션 조회 <br/>
    </div>
</details>
<details>
    <summary>11.8 쿼리 성능 테스트</summary>
    <div markdown="1">
        - 11.8.1 쿼리의 성능에 영향을 미치는 요소
    </div>
</details>