## 8.2 인덱스란?

DBMS를 책으로 비유한다면, 

- 책의 마지막에 있는 찾아보기 (색인) = 인덱스 → 빠른 검색 결과 조회 (칼럼(또는 칼럼들)의 값과 해당 레코드가 저장된 주소를 키와 값의 쌍(key-value pair)으로 삼아 인덱스를 만들어 둔다.)
- 책의 내용 = 데이터 파일
- 색인으로 알아낼 수 있는 페이지 번호 = 데이터 파일에 저장된 레코드의 주소 
- 색인의 구조 = 인덱스의 구조 = 주어진 순서로 미리 **정렬**해서 보관 → 빠르게 찾기 위함 

인덱스와 데이터 파일을 자료구조에 대응시켜보면,

- SortedList = 저장되는 값을 항상 정렬된 상태로 유지 = 인덱스 (저장되는 칼럼의 값을 이용해 항상 정렬된 상태 유지) = 저장하는 과정이 복잡하고 느리지만 이미 정렬돼 있어 검색이 빠름
- ArrayList = 값을 저장되는 순서 그대로 유지 = 데이터 파일

##### 저장 성능 희생, 읽기 속도 향상

테이블의 인덱스 추가 여부는 데이터 저장 속도를 얼마나 희생하고, 읽기 속도를 얼마나 빨리 만들어야 하는지에 따라 결정해야 한다.  예컨대 SELECT  쿼리 문장의 WHERE 조건절에 사용되는 컬럼이라고 전부 인덱스로 생성하면 데이터 저장 성능이 떨어지고 인덱스 크기가 비대해져 오히려 역효과만 불러일으킬 수 있다.

> ##### 인덱스 평가 기준
>
> - 접근 유형(Access Type) : 효율적으로 지원되는 접근 유형. 접근 유형은 특정한 속성의 값을 가진 레코드나 특정한 범위에 들어가는 속성의 값을 가지는 레코드를 찾는 것을 포함
> - 접근 시간(Access Time) : 고려중인 기법을 사용해 특정한 데이터 항목이나 항목의 집합을 찾는데 걸리는 시간
> - 삽입 시간(Insertion Time) : 새로운 데이터 항목을 삽입하는데 걸리는 시간. 새로운 데이터를 삽입하기 위한 정확한 위치를 찾는데 걸리는 시간 + 인덱스 구조 갱신 시간 
> - 삭제 시간(Deletion Time) : 데이터 항목을 삭제하는데 걸리는 시간. 삭제될 항목을 찾는데 걸리는 시간 + 인덱스 구조 갱신 시간 
> - 공간 부담(Space Overhead) : 인덱스 구조가 사용하는 부가적인 공간. 부가적인 공간의 양이 적당하다면 성능 향상을 위해 인덱스가 차지하는 공간을 늘리는 것은 가치 있음

#### 인덱스의 분류

> 인덱스 ≠ 검색 키 : 한 파일에서 레코드를 찾는데 사용되는 속성이나 속성들의 집합 

##### 역할별 구분

- 프라이머리 키(Primary Key)
    - 레코드를 대표하는 컬럼의 값(컬럼의 조합)으로 만들어진 인덱스
    - 테이블에서 해당 레코드를 식별하는 기준 값이 되므로 식별자라고도 함
    - NULL값 및 중복 값을 미허용
- 보조 키 또는 세컨더리 인덱스(Secondary Key)
    - 프라이머리 키를 제외한 나머지 모든 인덱스
    - 그 중 유니크 인덱스는 프라이머리키와 성격이 비슷하고 프라이머리 키 대체 사용이 가능하여 대체 키라고도 함.
    - 모든 검색 키 값과 모든 레코드에 대한 포인터를 가지는 인덱스 엔트리로 된 밀집 인덱스여야 한다. 
        - 클러스터링 인덱스는 희소 인덱스여도 됨 (연속적인 접근으로 파일 일부분에서 중간 검색 키 값을 가지는 레코드를 찾는 것이 항상 가능하기 때문) 보조 인덱스가 몇몇 키 값을 저장하면 중간 검색 키 값을 가지는 레코드는 파일 어딘가에 있으므로 전체 파일 검색하지 않고서는 찾을 수 없다. 
    - 클러스터링 인덱스의 검색 키가 아닌 검색 키를 사용하는 질의의 성능을 향상 (보조 인덱스 갱신 비용이 있음)

> #### 순서 인덱스 (검색 키의 값을 정렬 저장, 이를 레코드와 연계)
>
> 기본 인덱스(클러스터링) : 레코드, 파일이 연속적인 순서로 정의한 속성을 검색 키로 사용
>
> 보조 인덱스(논클러스터링) : 파일의 연속적인 순서와 다른 순서로 구성되는 검색 키의 인덱스
>
> 인덱스 순차 파일 : 검색 키에 대해 기본 인덱스를 가지는 파일
>
> ##### 인덱스와 엔트리의 관계
>
> - 밀집 인덱스
>     - 밀집 인덱스의 인덱스 엔트리는 파일에 있는 모든 검색 키 값에 대해 나타남.
>     - 밀집 클러스터링 인덱스에서 인덱스 레코드는 검색 키 값과 그 검색 키 값의 첫번째 데이터 레코드에 대한 포인터 포함
>     - 똑같은 검색 키 값을 가진 나머지 레코드는 첫 번째 레코드 이후부터 연속적 저장 (클러스터링 인덱스로 동일한 검색 키로 정렬되기 때문)
>     - 밀집 비클러스터링 인덱스에서 인덱스는 똑같은 검색 키를 가진 모든 레코드에 대한 포인터 목록을 저장해야 함
>
> - 희소 인덱스 (책의 목차)
>     - 희소 인덱스의 인덱스 엔트리는 검색 키 값에 대해 몇 개만 나타남 
>     - 희소 인덱스는 오직 릴레이션이 검색 키로 정렬되어 저장될 때, 즉 인덱스가 클러스터링 인덱스인 경우 사용될 수 있음 
>     - 각 인덱스 엔트리는 검색 키 값과 그 검색 키 값의 첫 번째 데이터 레코드에 대한 포인터 포함
>     - 레코드를 위치시키기 위해서는 찾고자 하는 검색 키 값보다 작거나 동일한 것 중 가장 큰 검색 키 값을 가지는 인덱스 엔트리를 찾고, 그 파일에서 원하는 레코드를 찾을 때까지 포인터를 따라간다.
>
> #### 다계층 인덱스
>
> 메인 메모리에 인덱스를 다 넣을 수 없음. 인덱스 블록을 디스크로부터 메인 메모리에 읽은 시간 多. 여러 단계의 인덱스 생성(희소 외부 인덱스 - 내부 인덱스 블록). 다계층 인덱스를 통해 이진 검색으로 레코드를 찾는 것보다 상당히 적은 입출력 연산 소모 
>
> → 더 효율적인 트리구조 ?!
>
> #### 해시 인덱스(Hash Index) : 버킷의 범위 안에서 값이 일정하게 분배됨. 값이 할당되는 버킷은 해시 함수에 의해 결정됨

##### 데이터 저장 방식(알고리즘)별 구분

- B-Tree 인덱스
    - 칼럼의 값을 변형하지 않고 원래의 값을 이용해 인덱싱하는 알고리즘
    - MySQL의 위치 기반 검색을 지원하는 R-Tree 인덱스 알고리즘도 B-Tree의 응용
- Hash 인덱스
    - 칼럼의 값으로 해시값을 계산해서 인덱싱하는 알고리즘
    - 매우 빠른 검색 지원
    - 값을 변형해 인덱싱하므로 전방(Prefix) 일치와 같이 값의 일부만 검색하거나 범위 검색할 때는 사용 불가
    - 주로 메모리 기반 데이터베이스에서 많이 사용됨 
- Fractal-Tree 인덱스
- 로그 기반의 Merge-Tree 인덱스



