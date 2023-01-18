# Shared Lock & Exclusive Lock

해당 문서에서 설명하는 Lock 메커니즘은 MySQL InnoDB를 기준으로 작성되었습니다.

## Lock 이란

멀티 트랜잭션 환경에서 데이터베이스의 일관성과 무결성을 유지하기 위해 보장할 수 있는 장치

# Shared Lock

`Shared Lock`은 `S Lock` `Read Lock`이라고도 한다.

`S Lock`이 걸린 데이터는 `SELECT`가 가능하며, `INSERT` `UPDATE` `DELETE`는 불가능하다.

`S Lock`이 걸린 데이터에 대해서 다른 트랜잭션도 `S Lock`을 획득할 수 있으나, `X Lock`은 획득할 수 없다.

`S Lock`을 사용하면 조회한 데이터가 트랜잭션 내내 변경되지 않음을 보장한다.


```sql
select * from table_name where id = 1 for share;
```

# Exclusive Lock

`Exclusive Lock`은 `Write Lock` 또는 `X Lock` 이라고도 한다.

`X Lock`을 획득한 트랜잭션은 `SELECT` `INSERT` `UPDATE` `DELETE`을 사용할 수 있다.

하지만 다른 트랜잭션은 `X Lock`이 걸린 데이터에 대해 `S Lock` 획득, 쓰기 연산을 할 수 없다.

다른 트랜잭션에서 `S Lock`, `X Lock`을 획득할 수 없다.

```sql
select * from table_name where id = 1 for update;
```

# Lock Test

## Expect

> 1. `S락`을 걸었을 경우 다른 트랜잭션에서 `S락` 획득 성공
> 2. `S락`을 걸었을 경우 다른 트랜잭션에서 `X락` 획득 실패
> 3. `X락`을 걸었을 경우 다른 트랜잭션에서 `S락` 획득 실패
> 4. `X락`을 걸었을 경우 다른 트랜잭션에서 `X락` 획득 실패

## 기본 세팅

```sql
create table tb_order
(
    order_id   bigint auto_increment primary key,
    user_id    bigint               default 0,
    order_name varchar(30) not null,
    create_dtm datetime    not null default CURRENT_TIMESTAMP,
    update_dtm datetime    not null default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP
);

insert into tb_order(user_id, order_name) values ('1', '오늘의 커피');
```

## S Lock Test 

### Expect.1 `S락`을 걸었을 경우 다른 트랜잭션에서 `S락` 획득 성공

**트랜잭션A에서 `SELECT ... FOR SHARE`으로 `S락` 획득**

```sql
> start transaction
completed in 10 ms
> select * from tb_order where order_id = 1 for share;
1 row retrieved starting from 1 in 26 ms (execution: 12 ms, fetching: 14 ms)
# 커밋하지 않음
```

**트랜잭션B에서 `SELECT ... FROM SHARE`으로 `S락` 획득 시도**

```sql
> start transaction
completed in 11 ms
> select * from tb_order where order_id = 1 for share;
1 row retrieved starting from 1 in 27 ms (execution: 15 ms, fetching: 12 ms)
> commit;
completed in 7 ms
```

### Expect.2 `S락`을 걸었을 경우 다른 트랜잭션에서 `X락` 획득 실패

```sql
> update tb_order set user_id = 3 where order_id = 1;
[40001][1205] Lock wait timeout exceeded; try restarting transaction
```

## X Lock Test

### Expect.3 `X락`을 걸었을 경우 다른 트랜잭션에서 `S락` 획득 실패

**트랜잭션A에서 `SELECT ... FOR UPDATE`으로 `X락` 획득**

```sql
> start transaction
completed in 3 ms
> select * from tb_order where order_id = 1 for update
1 row retrieved starting from 1 in 27 ms (execution: 8 ms, fetching: 19 ms)
```

**트랜잭션 B에서 `SELECT ... FOR UPDATE`으로 `X락` 획득 시도**

```sql
> start transaction
completed in 3 ms
> select * from tb_order where order_id = 1 for share
[40001][1205] Lock wait timeout exceeded; try restarting transaction
```

### Expect.4 `X락`을 걸었을 경우 다른 트랜잭션에서 `X락` 획득 실패

```sql
update tb_order set user_id = 3 where order_id = 1;
[40001][1205] Lock wait timeout exceeded; try restarting transaction
```

