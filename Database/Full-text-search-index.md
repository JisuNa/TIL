# Full Text Search Index (FSTI) 전문 검색 인덱스

## Overview

와일드카드가 앞에 붙은 `LIKE` '김%' 검색은 `인덱스`를 사용하여 조회를 합니다.

하지만 `LIKE` '%김'과 '%김%'은 `인덱스`를 사용하지 않고 `Table Full Scan`으로 검색해서 데이터가 많으면 많을수록 성능이 느려집니다.

이러한 `LIKE` 검색의 성능 개선을 위해 MySQL 5.6 이상 부터 지원하는 `Full Text Search Index`에 대해 알아보겠습니다.

(한글은 5.7이상부터 지원..)

## 인덱스 알고리즘

### 어근 분석 알고리즘

MySQL8.0 `Full Text Search Index`는 다음과 같은 두 가지 과정을 거쳐 색인 작업이 수행됩니다.

- Stop Word(불용어) 처리
- 어근 분석

**Stop Word**

불용어 처리는 검색에서 가치가 없는 단어를 모두 필터링해서 제거하는 작업입니다.

**어근 분석**

어근 분석은 검색어로 선정된 단어의 뿌리인 원형을 찾는 작업입니다.

MeCab은 일본어를 위한 형태소 분석 플러그인으로 한글에 맞게 사용하려면 단어 사전 등록과 실제 언어 샘플을 이용한 학습 과정이 필요합니다.

### n-gram 알고리즘

n-gram은 본문을 무조건적으로 몇 글자씩 잘라 인덱싱하는 방법입니다.

n-gram에서 n은 인덱싱할 키워드의 최소 글자 수를 의미하는데, MySQL은 기본 2글자 단위로 키워드를 쪼개서 인덱싱하는 2-gram(Bi-gram)을 사용합니다.

n-gram parser를 쓰기 위해서 `WITH PARSER ngram`을 테이블 생성, 변경으로 명시할 수 있습니다.

```sql
CREATE TABLE member
(
    member_id   BIGINT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
    member_name VARCHAR(20) NOT NULL,
    description TEXT NULL,
    FULLTEXT    INDEX ngram_idx(description) WITH PARSER `ngram`
) Engine=InnoDB CHARACTER SET utf8mb4;

ALTER TABLE member ADD FULLTEXT INDEX fti_description(description) WITH PARSER ngram;
```

전문 조회는 다음과 같습니다.

```sql
SELECT * FROM member WHERE MATCH(description) AGAINST('진상' IN BOOLEAN MODE);
```

### `Natural Language Mode` vs `Boolean Mode`

n-gram parser를 이용한 전문 검색은 `Natural Language Mode` 또는 `Boolean Mode`를 사용할 수 있는데, 둘의 결과 값이 차이가 있습니다.

**`Natural Language Mode`**

`Natural Language Mode`는 2글자 단위로 쪼개진 키워드를 모두 조회하기 때문에 원하지 않은 결과 값이 추가될 가능성이 존재합니다.

예를들어 `제주특별자치도` 내용이 있는 기사를 검색 했을 때 `제주특별자치도`는 `제주` `주특` `특별` `별자` `자치` `치도`로 쪼개진 키워드가 하나라도 포함된 데이터를 조회하게 됩니다. 

조회 결과에 `제주특별자치도` 데이터도 포함되지만 `제주공항` `주특기` `서울특별시` 등의 데이터도 함께 나오게 됩니다.

결과는 정확도에 따라 정렬됩니다.

**`Boolean Mode`**

`Boolean Mode`는 `스터디 그룹`과 같이 띄어쓰기를 포함한 구문 검색이 가능하고 정확도가 높습니다.

필수(+), 예외(-) 연산자를 사용할 수 있습니다.

```sql
SELECT * FROM member WHERE MATCH(description) AGAINST('+서울 -제주' IN BOOLEAN MODE);
```

### 성능

100만건의 데이터 조회의 성능을 비교해보았습니다.

```sql
> select * from starlucks.member where `description` like '%kpuhw%'
107 rows retrieved starting from 1 in 4 s 483 ms (execution: 4 s 470 ms, fetching: 13 ms)

select *, match(description) AGAINST('hu' IN NATURAL LANGUAGE MODE) as score from starlucks.member where match(description) AGAINST('kpuhw' IN NATURAL LANGUAGE MODE)
53159 rows retrieved starting from 1 in 562 ms (execution: 529 ms, fetching: 33 ms)

select *, match(description) AGAINST('kpuhw' IN BOOLEAN MODE) as score from starlucks.member where match(description) AGAINST('kpuhw' IN BOOLEAN MODE)
107 rows retrieved starting from 1 in 831 ms (execution: 820 ms, fetching: 11 ms)
```

`LIKE`는 결과는 정확하지만 4초 이상으로 가장 오래 걸렸고, FTS `NATURAL LANGUAGE MODE`가 가장 빨랐지만 정확도가 아주 낮았습니다.

결과적으로 FTS `BOOLEAN MODE`가 가장 빨르며 정확한 결과를 얻을 수 있었습니다.

## Conclusion

실무에서 `LIKE`로 조회를 사용하는 테이블의 데이터가 크다면 `Full Text Search Index`를 적용하여 성능 향상 성과를 낼 수 있습니다. 