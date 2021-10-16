## 8.5 전문 검색 인덱스

---

> 전문 검색이란? 게시물의 내용이나 제목 등과 같이 문장이나 문서의 내용에서 키워드를 검색을 뜻한다.
> 전문 검색 인덱스란 ? 문서 내용 전체에 대한 분석과 검색을 위한 인덱싱 알고리즘을 전문 검색 인덱스라 한다.

+ 전문(Full Text) 검색에는 InnoDB 나 MyISAM 스토리지 엔진에서 기본적으로 제공하는 B-Tree 인덱스는 사용할 수 없다. 
+ 전문검색은 이름이나 별명(닉네임) 과 같은 단어에서 일부만 일치하는 사용자를 검색하는 기능으로도 사용 할 수 있다.
  + 전문 검색은 LIKE 기능과 같이 패턴 일치 검색 기능(LIKE = (%%)을 사용한)  일부만 검색하는 경우에도 인덱스를 탄다.

### 8.5.1 인덱스 알고리즘

---

> 전문 검색에서는 사용자가 검색하게 될 키워드를 분석해 내고, 빠른 검색용으로 사용할 수 있게 키워드를 통한 인덱스를 구축한다. 크게 어근 분석과 n-gram 분석 알고리즘으로 구분할 수 있다.

<br>

#### 8.5.1.1 어근 분석

---

> MySQL 서버의 전문 검색 인덱스는 두 가지 중요한 과정을 거쳐 색인 작업이 수행된다. 

- [ ] 불용어(stop word) 처리
  + <b>별 가치가 없는 단어를 모두 필터링해서 제거하는 작업을 의미한다.</b>
  + 불용어의 개수는 많지 않기 때문에 알고리즘을 구현한 코드에서 보통 모두 상수로 정의해서 사용하거나 유연성을 위해서 불용어 자체를 데이터베이스화해서 사용자가 추가,삭제,수정할 수 있게 구현하는 경우도 있다.
  + MySQL 서버에는 불용어가 소스코드에 정의돼 있지만, 이를 무시하고 사용자가 별도로 불용어를 정의할 수 있다.
  

- [ ] 어근 분석(stemming)
  + <b>검색어로 선정된 단어의 뿌리인 원형을 찾는 작업이다.</b>
  + 한글이나 일본어의 경우 영어와 같이 단어의 변형 자체는 거의 없기 때문에 어근 분석보단 문장의 형태소를 분석해서 명사와 조사를 구분하는 기능이 더 중요하다.
  + MySQL 서버는 오픈소스 형태소 분석 라이브러리인 MeCab 을 플러그인 형태로 사용할 수 있게 지원하며, 서구권 언어를 위한 형태소 분석기는 MongoDB 에서 사용되는 Snowball 이라는 오픈 소스가 존재한다.
  + 중요한 것은 각 국가의 언어가 서로 다르기 때문에 형태소 분석이나 어근 분석 또한 언어별로 방식이 모두 다르다. (한국어 같은 경우 MeCab 을 사용)
    + MeCab 이 제대로 작동하려면 단어 사전이 필요하며, 문장을 해체해서 각 단어의 품사를 식별할 수 있는 문장의 구조 인식이 필요하다.
    + 문장의 구조 인식을 위해서는 실제 언어의 샘플을 이용해 언어를 학습하는 과정이 필요한데 이 과정은 시간이 많이 필요하다.


<br>

#### 8.5.1.2 n-gram 알고리즘

--- 

> 행태소 분석은 문장을 이해하기 위한 알고리즘이기 때문에 시간과 노력이 많이 들어간다. 이런 단점을 보완하기 위한 방법으로 단순히 키워드를 검색해내기 위한 인덱싱인 n-gram 알고리즘이 도입되었다.  

+ 본문을 무조건 몇 글자씩 잘라서 인덱싱하는 방법이다.
+ 알고리즘이 단순하고 국가별 언어에 대한 이해와 준비 작업이 필요하지 않는 반면, 만들어진 인덱스의 크기는 상당히 큰 편이다.
+ n-gram 의 n 은 인덱싱할 키워드의 최소 글자 수 를 의미하는데, 일반적으로 2 글자 단위로 키워드를 쪼개서 인덱싱하는 2-gram(또는 Bi-gram) 방식이 많이 사용된다.
+ MySQL 서버에 등록된 불용어로 처리할 경우 사용자에게 더 혼란을 주기 떄문에 직접 불용어를 등록하는 방법을 권장한다.
  + my.cnf 파일의 ft_stopword_file='' 설정을 통해 기본적으로 등록된 불용어를 삭제하거나 사용자 정의 불용어를 등록할 수 있다.
  + SET GLOBAL innodb_ft_enable_stopword=OFF;

##### 2-gram 알고리즘 인덱싱 기법

---

```
to be or not to be. that is the question
```

1. 띄어쓰기 또는 . 기준으로 10개의 단어로 구분된다
   <br> to, be, or, not, to, be, That, is, the, question
2. 2 글자씩 중첩해서 토큰으로 분리된다.
   <br> that > th, ha, at
   <br> be > be
   <br> not > no, ot 
3. 분리된 토큰을 인덱스에 저장하면 된다. 이때 중복된 토큰은 하나의 인덱스 엔트리로 병합되어 저장한다.
4. 불용어를 걸러내는 작업을 수행하는데, 이때 불용어와 동일하거나 불용어를 포함하는 경우 걸러서 버린다.

``` SQL
SELECT * FROM information_schema.INNODB_FT_DEFAULT_STOPWORD;

+-------+
| value |
+-------+
| a     |
| about |
| an    |
| are   |
| as    |
| at    |
| be    |
| by    |
| com   |
| de    |
| en    |
| for   |
| from  |
| how   |
| i     |
| in    |
| is    |
| it    |
| la    |
| of    |
| on    |
| or    |
| that  |
| the   |
| this  |
| to    |
| was   |
| what  |
| when  |
| where |
| who   |
| will  |
| with  |
| und   |
| the   |
| www   |
+-------+
```

5. 최종 인덱스를 B-Tree 인덱스에 저장한다.
   <br> et, he, no, ot, qu, st, Th, th, ue
<br>


#### 사용자 정의 불용어 처리

---

1. my.cnf 의 불용어 처리 파일 생성하는 방법

```
ft_stopword_file = '/data/my_custom_stopword.txt'
```

2. 불용어 목록을 테이블로 저장하는 방법 (InnoDB 만 사용 가능)
```SQL
CREATE TABLE my_stopword(value VARCHAR(30)) ENGINE = INNODB;
INSERT INTO my_stopword(value) VALUES ('MySQL');

SET GLOBAL innodb_ft_server_stopword_table ='mydb/my_stopword';
ALTER TABLE tb_bi_gram ADD FULLTEXT INDEX fx_title_body(title, body) WITH PARSER ngram;

-- 이떄 불용어 목록을 변경한 이후 전문 검색 인덱스가 생성돼야만 변경된 불용어가 적용된다.

```
3. innodb_ft_stopword_table 시스템 변수를 이용하는 방법은 innodb_ft_server_stopword_table 시스템 변수와 사용법은 동일하다.
  + 단, 여러 전문 검색 인덱스가 서로 다른 불용어를 사용해야 하는 경우 innodb_ft_user_stopword_table 시스템 변수를 이용한다.

<br>

### 8.5.2 전문 검색 인덱스의 가용성

---

> 전문 검색 인덱스를 사용하려면 아래 두 가지 조건을 갖춰야 한다.

+ 쿼리 문장이 전문 검색을 위한 문법(MATCH ... AGAINST ...)을 사용
+ 테이블이 전문 검색 대상 컬럼에 대해서 전문 인덱스 보유

```SQL
CREATE TABLE tb_test (
    doc_id INT,
    doc_body TEXT,
    PRIMARY KEY(doc_id),
    FULLTEXT KEY fx_docbody (doc_body) WITH PARSER ngram
) ENGINE = InnoDB;



SELECT * FROM tb_test WHERE doc_body LIKE '%애플%'; (X)

SELECT * FROM tb_test WHERE MATCH(doc_body) AGAINST ('애플' IN BOOLEAN MODE); (O)


```

<br>

  
## 8.6 함수 기반 인덱스

> 컬럼의 값을 변형해서 만들어진 값에 대한 인덱스를 구축해야 할 때 함수 기반 인덱스를 활용한다.
> MySQL 8. 버전부터 지원하며 가상 컬럼을 이용한 인덱스와 함수를 이용한 인덱스가 있다.

+ 함수 기반 인덱스는 인덱싱할 값을 계산하는 과정의 차이만 있을 뿐, 실제 인덱스의 내부적인 구조는 B-Tree 인덱스와 동일하다.

### 8.6.1 가상 컬럼을 이용한 인덱스

```SQL
CREATE TABLE 유저 (
  이름 VARCHAR(10)
  성 VARCHAR(10)
)

ALTER TABLE 유저 
    ADD full_name VARCHAR(30) AS (CONCAT(이름,'',성)) VIRTUAL,
    ADD INDEX ix_full_name (full_name)
```

+ WHERE full_name = '문승찬'
+ 가상 컬럼은 테이블에 새로운 컬럼을 추가하는 것과 같은 효과를 내기 떄문에 실제 테이블의 구조가 변경된다는 단점이 있다.

### 8.6.2 함수를 이용한 인덱스

+ 테이블 구조를 변경하지 않고, 계산된 결과값의 검색을 빠르게 만들어준다.
+ 조건절에 함수 기반 인덱스에 명시된 표현식이 그대로 사용돼야 한다.
  + WHERE CONCAT(이름, '', 성) = '문승찬'
  

## 8.7 멀티 밸류 인덱스

> MySQL 5. 버전부터 json 타입 컬럼을 지원하였지만 이에 대한 인덱스를 지원하지 않았다. 8. 버전부턴 이런 컬럼들에 대한 멀티 밸류 인덱스를 지원한다.

+ 멀티 밸뷰 인데스를 활용하기 위해서는 일반적인 조건 방식을 사용하면 안되고, 반드시 다음 함수들을 이용해야 한다.
  + MEMBER OF()
  + JSON_CONTAINS()
  + JSON_OVERLAPS()
  

