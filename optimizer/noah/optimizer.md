
# 9.옵티마이저와 힌트

---

<br>

## 9.2 기본 데이터 처리

---

<br>

### 9.2.1 풀 테이블 스캔과 풀 인덱스 스캔

---

#### 옵티마이저가 풀 테이블 스캔을 선택하는 경우

1. 테이블의 레코드 건수가 너무 작은 경우 (일반적으로 테이블이 페이지 1개로 구성된 경우)
2. WHERE 절이나 ON 절에 인덱스를 이용할 수 있는 적절한 조건이 없는 경우
3. 인덱스 레인지 스캔을 사용할 수 있는 쿼리더라도 옵티마이저가 판단한 조건 일치 레코드 건수가 너무 많은 경우

<small> (P.232) 인덱스 레인지 스캔 내용 중 인덱스를 통해 읽어야 할 데이터 레코드가 20~25%를 넘으면 인덱스보다 테이블 데이터를 직접 읽는 것이 효율적인 처리 방식이다. </small>

<br>
<br>


#### MySQL 의 풀 테이블 스캔

> NyISAM 은 디스크로부터 페이지를 하나씩 읽어오지만 InnoDB 는 특정 테이블의 연속된 데이터 페이지가 읽히면 백그라운 스레드에 의해 리드 어헤드 작업이 시작된다. 

1. 풀 테이블 스캔이 실행되면 처음 몇 개의 데이터 페이지는 포그라운드 스레드가 페이지 읽기를 실행하지만 특정 시점부터는 읽기 작업을 백그라운드 스레드로 넘긴다.
2. 백그라운드 스레드가 읽기를 넘겨받는 시점부터는 한 번에 4개 또는 8개씩의 페이지를 읽으면서 계속 그 수를 증가 시킨다.
3. 한 번에 최대 64개의 데이터 페이지까지 읽어서 버퍼 풀에 저장한다.
4. 포그라운드 스레드는 버퍼 풀에 미리 준비된 데이터를 가져다 사용하기만 하면 되므로 쿼리가 빨리진다.
5. innodb_read_ahead_threshold 시스템 변수를 이용해 리드 어헤드 시작 임계값을 설정할 수 있다.

<br>

<small> 

리드 어헤드 (Read ahead) 란 ?

> 어떤 영역의 데이터가 앞으로 필요해지리라는 것을 예측해서 요청이 오기 전에 미리 디스크에 읽어 InnoDB 의 버퍼 풀에 가져다 두는 것을 의미한다. ( 케시 데이터 지역성 중 공간 지역성과 비슷해 보인다. )

버퍼 풀이란 ?

> 테이블(디스크의 데이터 파일) 및 인덱스 데이터를 캐시하는 주 메모리 영역을 버퍼 풀이라 한다. 버퍼 풀은 변경된 데이터를 모아서 처리하기 떄문에 랜덤 I/O 횟수를 줄일 수 있다.

![img.png](img.png)


MySQL 스레딩 구조 P.80

> MySQL 서버는 스레드 기반으로 작동하며 크게 포그라운드, 백그라운드 스레드로 구분된다.

포그라운드 스레드란 (Foreground thread)? P.82

> 포그라운드 스레드는 최소한 MySQL 서버에 접속된 클라이언트의 수만큼 존재하며, 주로 각 클라이언트 사용자가 요청하는 쿼리 문장을 처리한다.
> 
> 포그라운드 스레드는 데이터를 MySQL 의 데이터 버퍼나 캐시로부터 가져오며, 버퍼나 캐시에 없는 경우에는 직접 디스크의 데이터나 인덱스 파일로부터 데이터를 읽어와서 작업을 처리한다.


백그라운드 스레드란 (Background thread)? P.83
+ 인서트 버퍼를 병합하는 스레드
+ <b>로그를 디스크로 기록하는 스레드</b>
+ <b>InnoDB 버퍼 풀의 데이터를 디스크에 기록하는 스레드</b>
+ 데이터를 버퍼로 읽어오는 스레드
+ 잠금이나 데드락을 모니터링하는 스레드

</small>

#### MySQL 의 풀 인덱스 스캔
+ 리드 어헤드는 풀 테이블 스캔과 동일하게 사용된다.

```SQL  
1. 풀 인덱스 스캔을 하는 경우 : MySQL 서버는 단순히 레코드의 건수만 필요로 하는 쿼리라면 용량이 작은 인덱스를 선택하여 I/O 횟수를 줄일 확률이 크다.
SELECT COUNT(*) FROM employees; 


2. 풀 테이블 스캔을 하는 경우 : 레코드에만 있는 칼럼이 필요한 경우에는 풀 테이블 스캔을 한다.
SELECT * FROM employees; 
```

<br>

### 9.2.2 병렬 처리

---

> innodb_parallel_read_threads 시스템 변수를 이용해 하나의 쿼리에 최대 몇 개의 스레드를 이용해서 처리할지를 변경할 수 있다.

+ WHERE 조건 없이 단순히 테이블의 전체 건수를 가져오는 쿼리만 병렬처리 가능하다. (순차 I/O)
+ 스레드의 개수는 CPU 코어의 개수를 넘지 않도록 한다. 

```SQL
SET SESSION innodb_parallel_read_threads = 2;
SELECT COUNT(*) FROM tb;

SET SESSION innodb_parallel_read_threads = 4;
SELECT COUNT(*) FROM tb;

```

<br>

### 9.2.3 ORDER BY 처리 (using filesort)

---


#### 인덱스를 활용한 정렬

##### 장점
     INSERT, UPDATE, DELETE 쿼리가 실행될 때 이미 정렬된 인덱스를 순서대로 읽기만 하면 되므로 매우 빠르다.
##### 단점
     INSERT, UPDATE, DELETE 작업 시 부가적인 인덱스 추가/삭제 작업이 필요하다.
     인덱스 때문에 디스크 공간이 더 필요하다.
     인덱스의 개수가 늘어날수록 InnoDB 버퍼 풀을 위한 메모리가 더 필요하다.

<br> 

#### Filesort 를 이용한 정렬

##### 장점
     인덱스를 생성하지 않아도 되므로 인덱스를 활용한 정렬에 단점이 장점으로 바뀐다.
     정렬해야 할 레코드가 많지 않으면 메모리에서 Filesort 가 처리되므로 빠르다.
##### 단점
     정렬 작업이 쿼리 실행 시 처리되므로 레코드 대상 건수가 많아질수록 쿼리의 응답 속도가 느리다.

#### 사례
+ 정렬 기준이 너무 많아서 요건별로 모두 인덱스를 생성하는 것이 불가능한 경우
+ GROUP BY 의 결과 또는 DISTINCT 같은 처리의 결과를 정렬해야 하는 경우
+ UNION 의 결과와 같이 임시 테이블의 결과를 다시 정렬해야 하는 경우
+ 랜덤하게 결과 레코드를 가져와야 하는 경우

#### 적용확인
+ 실행 계획의 Extra 칼럼에 "Using filesort" 메세지가 표시되는지 여부
    
<br>

 
#### 2.2.3.1 소트 버퍼

---

> MySQL 은 정렬을 수행하기 위해 별도의 메모리 공간을 할당 받는데, 이 메모리 공간을 <b> 소트 버퍼(Sort Buffer)</b> 라 부른다.
>
> 소트 버퍼는 정렬이 필요한 경우에만 할당되며, 버퍼의 크기는 정렬해야 할 레코드의 크기에 따라 가변적으로 증가하고, 최대 사용 가능한 소트 버퍼 공간은 sort_buffer_size 를 통해 설정할 수 있다.


##### 레코드 건수가 소트 버퍼로 할당된 공간보다 큰 경우

![](SortBuffer1.jpg)

+ 수행된 멀티 횟수는 Sort_merge_passes 상태변수에 누적해서 집계된다.
+ 소트 버퍼를 레코드 크기만큼 크게 설정 한다고 하여도 큰 차이가 없다.
+ 소프 버퍼의 크기가 256KB 에서 8MB 사이에서 최적의 성능을 보인다.
    ![img_1.png](img_1.png)
  
+ 정렬을 위해 할당하는 소트 버퍼는 세션 메모리 영역에 속한다. 즉, 여러 클라이언트가 공유해서 사용할 수 있는 영역이 아니다.
+ 소트 버퍼의 크기를 너무 크게 설정하면 (10MB 이상) 운영체재는 메모리 부족 현상을 겪을 수 있다.

<br>

#### 2.2.3.2 정렬 알고리즘

---

> 레코드를 정렬할 때 레코드 전체를 소트 버퍼에 담을지 또는 정렬 기준 컬럼만 소트 버퍼에 담을지에 따라 "싱글 패스" 와 "투 패스" 2가지 정렬 모드로 나눌 수 있다.

```SQL
-- 옵티마이저 트레이스 활성화
SET OPTIMIZER_TRACE = "enabled=on", END_MARKERS_IN_JSON = on;
SET OPTIMIZER_TRACE_MAX_MEM_SIZE = 1000000;

-- 쿼리 실행
SELECT * FROM employees ORDER BY last_name LIMIT 100000, 1;

-- 트레이스 내용 확인
SELECT * FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE \G 
... 


'{
  "steps": [
    {
      "join_preparation": {
        "select#": 1,
        "steps": [
          {
            "expanded_query": "/* select#1 */ select `hospital`.`id` AS `id`,`hospital`.`name` AS `name` from `hospital` order by `hospital`.`name` limit 100000,1"
          }
        ] /* steps */
      } /* join_preparation */
    },
    {
      "join_optimization": {
        "select#": 1,
        "steps": [
          {
            "substitute_generated_columns": {
            } /* substitute_generated_columns */
          },
          {
            "table_dependencies": [
              {
                "table": "`hospital`",
                "row_may_be_null": false,
                "map_bit": 0,
                "depends_on_map_bits": [
                ] /* depends_on_map_bits */
              }
            ] /* table_dependencies */
          },
          {
            "rows_estimation": [
              {
                "table": "`hospital`",
                "table_scan": {
                  "rows": 32,
                  "cost": 1
                } /* table_scan */
              }
            ] /* rows_estimation */
          },
          {
            "considered_execution_plans": [
              {
                "plan_prefix": [
                ] /* plan_prefix */,
                "table": "`hospital`",
                "best_access_path": {
                  "considered_access_paths": [
                    {
                      "rows_to_scan": 32,
                      "access_type": "scan",
                      "resulting_rows": 32,
                      "cost": 7.4,
                      "chosen": true,
                      "use_tmp_table": true
                    }
                  ] /* considered_access_paths */
                } /* best_access_path */,
                "condition_filtering_pct": 100,
                "rows_for_plan": 32,
                "cost_for_plan": 7.4,
                "sort_cost": 32,
                "new_cost_for_plan": 39.4,
                "chosen": true
              }
            ] /* considered_execution_plans */
          },
          {
            "attaching_conditions_to_tables": {
              "original_condition": null,
              "attached_conditions_computation": [
              ] /* attached_conditions_computation */,
              "attached_conditions_summary": [
                {
                  "table": "`hospital`",
                  "attached": null
                }
              ] /* attached_conditions_summary */
            } /* attaching_conditions_to_tables */
          },
          {
            "clause_processing": {
              "clause": "ORDER BY",
              "original_clause": "`hospital`.`name`",
              "items": [
                {
                  "item": "`hospital`.`name`"
                }
              ] /* items */,
              "resulting_clause_is_simple": true,
              "resulting_clause": "`hospital`.`name`"
            } /* clause_processing */
          },
          {
            "refine_plan": [
              {
                "table": "`hospital`"
              }
            ] /* refine_plan */
          }
        ] /* steps */
      } /* join_optimization */
    },
    {
      "join_execution": {
        "select#": 1,
        "steps": [
          {
            "filesort_information": [
              {
                "direction": "asc",
                "table": "`hospital`",
                "field": "name"
              }
            ] /* filesort_information */,
            "filesort_priority_queue_optimization": {
              "limit": 100001,
              "rows_estimate": 1092,
              "row_size": 520,
              "memory_available": 262144
            } /* filesort_priority_queue_optimization */,
            "filesort_execution": [
            ] /* filesort_execution */,
            "filesort_summary": {
              "rows": 32,
              "examined_rows": 32,
              "number_of_tmp_files": 0,
              "sort_buffer_size": 261888,
              "sort_mode": "<sort_key, rowid>"
            } /* filesort_summary */
          }
        ] /* steps */
      } /* join_execution */
    }
  ] /* steps */
}'

```
+ 출력 내용에서 filesort_summary 섹션의 sort_algorithm 필드에 정렬 알고리즘이 표시된다.
+ sort_mod 필드에는 "<fixed"
<br>