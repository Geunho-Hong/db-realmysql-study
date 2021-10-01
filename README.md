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
