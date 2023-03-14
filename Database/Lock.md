# Lock

## Lock 이란

멀티 트랜잭션 환경에서 데이터베이스의 **일관성**과 **무결성**을 유지하기 위해 보장할 수 있는 장치

## Lock의 설정 범위 (Level)

### Database

- 데이터베이스 전체 Lock
- 1개의 세션만 DB의 데이터 접근이 가능
- 일반적으로 사용하지 않지만 DB의 버전업과 같은 경우에 사용

### File

- 파일이란 테이블, row 등과 같은 실제 데이터가 쓰여지는 물리적인 저장소 (잘 사용하지 않음)

### Table

- 테이블 수준의 Lock
- 테이블의 모든 row을 업데이트 처럼 전체 테이블에 영향을 주는 변경을 수행할 때 사용
- DDL (CREATE, ALTER, DROP) 구문 사용시 Lock 됨

### Page & Block

- 파일의 일부인 페이지와 블록을 기준으로 Lock (잘 사용하지 않음)

### Column

- 컬럼 기준으로 Lock
- 설정 및 해제의 리소스가 많이 들어 일반적으로 사용하지 않고, 지원하는 DBMS도 많지 않음

### Row

- 행 기준으로 Lock
- DML(SELECT, INSERT, UPDATE, DELETE) 구문 사용시 Lock 됨

## Shared Lock

`Shared Lock`은 S Lock, Read Lock 이라고도 한다.

S Lock이 걸린 데이터는 `SELECT`가 가능하며, `INSERT` `UPDATE` `DELETE`는 불가능하다.

S Lock이 걸린 데이터에 대해서 다른 트랜잭션도 S Lock을 획득할 수 있으나, X Lock은 획득할 수 없다.

S Lock을 사용하면 조회한 데이터가 트랜잭션 내내 변경되지 않음을 보장한다.

```sql
-- 8.0
select * from table_name where id = 1 for share;

-- 5.8 이하
select * from table_name where id = 1 lock in share mode;
```

## Exclusive Lock

Exclusive Lock은 Write Lock 또는 X Lock 이라고도 한다.

X Lock을 획득한 트랜잭션은 `SELECT` `INSERT` `UPDATE` `DELETE`을 사용할 수 있다.

하지만 다른 트랜잭션은 X Lock이 걸린 데이터에 대해 S Lock 획득, 쓰기 연산을 할 수 없다.

다른 트랜잭션에서 S Lock, X Lock을 획득할 수 없다.

```sql
select * from table_name where id = 1 for update;
```

## Intent Lock

내재 락은 사용자가 요청한 범위에 대한 락을 걸 수 있는지 빠르게 파악하기 위해 사용되는 락이다.

IS, IX, SIX 등이 있다.

## Blocking

블로킹은 Lock 경합이 발생하여 특성 세션이 작업을 진행하지 못하고 멈춰 선 상태

S Lock 간에는 블로킹이 발생하지 않지만 S 락과 X 락, X과 X락 간에는 블로킹이 발생한다. 

## 주의사항

- 트랜잭션의 길이가 너무 길면 경합 확률이 높아진다.
- 트랜잭션 격리 레벨을 불필요하게 상향 조정하지 않는다.
- 쿼리 시간이 길어지지 않게 적절한 튜닝이 필요.

## in JPA..

### PESSIMISTIC_READ

```java
public interface UserRepository extends JpaRepository<User, Long> {
    @Lock(LockModeType.PESSIMISTIC_READ)
    Optional<User> findWithPessimisticLockById(Long id);
}
```

```sql
select
    ...
from
    user user0_ 
where
    user0_.id=? lock in share mode
```

### PESSIMISTIC_WRITE

```java
public interface UserRepository extends JpaRepository<User, Long> {
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    Optional<User> findWithPessimisticLockById(Long id);
}
```

```sql
select
    ...
from
    user user0_ 
where
    user0_.id=? for update
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

# Conclusion
