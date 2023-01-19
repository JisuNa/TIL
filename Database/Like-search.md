# Like 검색

## Overview

이 문서에서는 MySQL 검색 조건에서 사용되는 `LIKE`검색 조회 성능에 대해 알아보겠습니다.

이름에 김을 포함하는 사람은 아래와 같은 조건으로 조회합니다.

```sql
SELECT * FROM member WHERE member_name LIKE "%김%";
```

## 인덱스와 Like 검색

조회 성능을 높이기 위해 `인덱스`를 사용합니다.

그렇다면 `LIKE`검색에도 `인덱스`가 사용될까요?

### 기본 세팅

```sql
create table member(
    member_id bigint auto_increment primary key,
    member_name varchar(20) not null default ''
) ENGINE = 'innoDB';

create index member_idx01 on `member` (member_name);
```

### LIKE '김%'

```sql
explain select * from member where member_name like '김%';

+----+-------------+--------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
| id | select_type | table  | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+--------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | member | NULL       | range | member_idx01  | member_idx01 | 82      | NULL |    2 |      100 | Using index condition |
+----+-------------+--------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
```

와일드카드가 뒤에 붙은 '김%' `Like` 검색은 `인덱스`를 사용한 검색을 했습니다.

### LIKE '%김'

```sql
explain select * from member where member_name like '%김';

+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | member | NULL       | ALL  | NULL          | NULL | NULL    | NULL |   10 |    11.11 | Using where |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
```

와일드카드가 앞에 붙은 '%김' `LIKE`검색은 `인덱스`를 사용하지 않고 `Table Full Scan`이 발생했습니다.

### LIKE '%김%'

```sql
explain select * from member where member_name like '%김%';

+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table  | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | member | NULL       | ALL  | NULL          | NULL | NULL    | NULL |   10 |    11.11 | Using where |
+----+-------------+--------+------------+------+---------------+------+---------+------+------+----------+-------------+
```

와일드카드가 앞뒤로 붙은 '%김%' `LIKE`검색 또한 인덱스를 사용하지 않고 `Table Full Scan`이 발생했습니다.

## 왜 와일드카드 위치에 다른 결과가 나올까?

세 가지 케이스 결과가 동일하지 않은 이유는 `인덱스`는 `B+Tree` 구조로 되어있기 때문에 와일드카드가 뒤에 붙은 B+Tree 구조에서 '김%'은 찾을 수 있지만, '%김' 과 '%김%' 은 `인덱스`를 이용하여 찾을 수 없어 `Table Full Scan`으로 조회하게 됩니다.

## Conclusion

`LIKE` 검색에서 조건이 '%김'과 '%김%'에서 `Table Full Scan`이 발생합니다.