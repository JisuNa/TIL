# RealMySQL 8.0 1권

# 01 소개

## 1.2 왜 MySQL인가?
자기가 가장 잘 활용할 수 있는 DBMS가 가장 좋은 DBMS이다.
- 안정성
- 성능과 기능
- 커뮤니티나 인지도

# 02 설치와 설정

### 2.1 MySQL 서버 설치

해당 내용은 책 내용과 다르게 Docker에 설치하는 과정이다.

간편한 `docker-compose`를 사용하여 설치한다.

```yaml
#docker-compose.yml
version: '3'
services:
  db:
    platform: linux/amd64
    container_name: mysql-8.0.29-test1
    image: mysql:8.0.29-debian
    restart: always
    ports:
      - 3155:3306
    environment:
      MYSQL_ROOT_PASSWORD: example
      TZ: Asia/Seoul
```

`docker-compose.yml`이 작성된 경로에서 아래 명령을 실행한다.

```shell
$ docker-compose up -d
```

컨테이너가 잘 올라왔는지 확인한다.

```shell
$ docker ps

CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                               NAMES
e5594d6ff60a   mysql     "docker-entrypoint.s…"   8 minutes ago   Up 8 minutes   33060/tcp, 0.0.0.0:3155->3306/tcp   mysql-test
```

# 03 사용자 및 권한

## 계정 생성

5.7 까지는 `GRANT` 명령으로 권한의 부여와 동시에 계정 생성이 가능했다.

8.0 부터는 계정 생성은 `CREAT USER` 명령으로 권한 부여는 `GRANT` 명령으로 구분해서 실행하도록 바뀌었다.

계정 생성 시 옵션
- 계정의 인증 방식과 비밀번호
- 비밀번호 유효기간, 이력 개수, 재사용 불가 기간
- 기본 역할
- SSL 옵션
- 계정 잠금 여부

```sql
CREATE USER 'user'@'%'
IDENTIFIED WITH 'mysql_native_password' BY 'password' 
REQUIRE NONE
PASSWORD EXPIRE INTERVAL 30 DAY
ACCOUNUT UNLOCK
PASSWORD HISTORY DEFAULT 
PASSWORD REUSE INTERVAL DEFAULT
PASSWORD REQUIRE CURRENT DEFAULT;
```

### `IDENTIFIED WITH`

사용자의 인증 방식과 비밀번호 설정

`IDENTIFIED WITH` 뒤에는 반드시 인증 방식을 명시

서버의 기본 인증 방식을 사용하고자 한다면 `IDENTIFIED BY 'password'`형식으로 명시해야 한다.

기본 인증 방식은 `Caching SHA-2 Authentication`

### `REQUIRE`

MySQL 서버에 접속할 때 암호화된 SSL/TLS을 사용 여부 설정

### `PASSWORD EXPIRE`

비밀번호 유효기간을 설정

별도로 명시하지 않으면 DEFAULT 값으로 설정

- `PASSWORD EXPIRE` : 계정 생성과 동시에 비밀번호 만료 처리
- `PASSWORD EXPIRE NEVER` : 만료 기간 없음
- `PASSWORD EXPIRE DEFAULT` : default_password_lifetime 시스템 변수 값
- `PASSWORD EXPIRE INTERVAL n DAY` : 유효 기간을 오늘부터 n일자로 설정

### `PASSWORD HISTORY`

한번 사용했던 비밀번호를 재사용하지 못하게 설정

- `PASSWORD HISTORY DEFAULT` : password_history 시스템 변수 값
- `PASSWORD HISTORY n` : 비밀번호의 이력을 최근 n개 저장

### `PASSWORD REUSE INTERVAL`

한 번 사용한 비밀번호 재사용 금지 기간 설정

- `PASSWORD REUSE INTERVAL DEFAULT` : password_reuse_interval 변수 값 
- `PASSWORD REUSE INTERVAL n DAY` : n일 이후 비밀번호 재사용 가능

### `PASSWORD REQUIRE`

비밀번호 만료 후 새 비밀번호 변경할 때 기존 비밀번호 필요여부 옵션

- `PASSWORD REQUIRE CURRENT` : 비밀번호 변경 시 기존 비밀번호 필요
- `PASSWORD REQUIRE OPTIONAL` : 기존 비밀번호 불필요
- `PASSWORD REQUIRE DEFAULT` : password_require_current 시스템 변수 값

### `ACCOUNT LOCK / UNLOCK`

계정 생성/변경 시 계정 잠금 여부 설정

- `ACCOUNT LOCK` : 잠금
- `ACCOUNT UNLOCK` : 잠금 해제

## 권한 Privilege

5.7 버전의 권한은 정적 권한이라고 하며 8.0 부터는 동적 권한이 추가 됐다.

![img.png](../images/정적권한-1.png)
![img.png](../images/정적권한-2.png)
![img.png](../images/동적권한.png)


```sql
# 모든 DB에 권한 부여
GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'user'@'localhost';
# 특정 스키마에 권한 부여 
GRANT SELECT, INSERT, UPDATE, DELETE ON employees.* TO 'user'@'localhost';
# 특정 스키마의 테이블에 권한 부여
GRANT SELECT, INSERT, UPDATE, DELETE ON employees.department.* TO 'user'@'localhost';
# 특정 스키마의 테이블에 권한 부여 + 수정권한은 dept_name 컬럼만
GRANT SELECT, INSERT, UPDATE(dept_name) ON employees.department.* TO 'user'@'localhost';
```

# 04 아키텍처

## MySQL 엔진 아키텍처

쿼리를 작성하고 튜닝할 때 엔진의 구조를 알아야한다.

MySQL 서버는 다른 DBMS에 비해 구조가 상당히 독특하다.

독특한 구조 때문에 다른 DBMS에서는 가질 수 없는 엄청난 혜택이 있지만 다른 DBMS에서는 문제되지 않는 것들이 MySQL에서는 문제되기도 한다.

![img.png](../images/mysql아키텍처.png)

MySQL 서버는 크게 MySQL 엔진과 스토리지 엔진으로 구분할 수 있다.

### MySQL 엔진

클라이언트가 접속 및 쿼리 요청을 처리하는 커넥션 핸들러와 SQL 파서, 전처리기, 쿼리의 최적화된 실행을 위한 옵티마이저가 있다.

### 스토리지 엔진

실제 데이터를 디스크 스토리지에 READ, WRITE 하는 것은 스토리지 엔진이 한다.

MySQL 서버에서 MySQL엔진은 하나지만 스토리지 엔진은 여러 개를 동시에 사용할 수 있다.

### 핸들러 API

MySQL 엔진의 쿼리  실행기에서 데이트럴 읽고 쓸 때 각 스토리지 엔진에 쓰기 읽기를 요청하는데 이를 핸들러 요청이라고 한다.

### MySQL 스레딩 구조

![img.png](../images/mysql스레딩구조.png)

- MySQL 서버는 스레드 기반으로 작동
- 포그라운드와 백그라운드 스레드로 구분
- 실행 중인 스레드 목록 확인이 가능
    ```sql
    SELECT thread_id, name, type, processlist_user, processlist_host, FROM performance_schema.threds ORDER BY type, thred_id;
    ```
  
### 포그라운드 스레드 (클라이언트 스레드)

- 포그라운드 스레드는 MySQL 서버에 접속된 클라이언트의 수만큼 존재
- 각 클라이언트가 요청하는 쿼리 문장을 처리
- 커넥션을 종료하면 스레드 캐시로 돌아간다.
- 데이터를 데이터 버처나 캐시에서 가져오며, 버퍼나 캐시에 없는 경우 직접 디스크의 데이터나 인덱스 파일로부터 데이터를 읽어온다.
- InnoDB 테이블은 데이터 버퍼나 캐시까지는 포그라운드 스레드가 처리하고 나머지 버퍼로부터 디스크까지 기록은 백그라운드 스레드가 처리.

### 백그라운드 스레드

MyISAM은 해당사항이 없는 부분이지만 InnoDB는 여러 가지 작업을 백그라운드 스레드가 한다.

- Insert Buffer를 병합
- 로그를 디스크로 기록 
- InnoDB 버퍼 풀의 데이터를 디스크에 기록
- 데이터를 버퍼로 읽어 옴
- 잠금이나 데드락을 모니터링

가장 중요한 것은 로그스레드와 버퍼의 데이터를 디스크로 내려쓰는 작업을 하는 쓰기 스레드

### 트랜잭션 지원 메타데이터

데이터베이스 서버에서 테이블의 구조와 스토어드 프로그램 등의 정보를 메타데이터라고 하는데 5.7 버전까지는 테이블의 구조를 FRM 파일에 저장하여 관리했다.

파일 기반의 메타데이터는 생성 및 변경 작업이 트랜잭션을 지원하지 않기 때문에 테이블의 생성 또는 변경 중에 서버가 비정상적으로 종료되면 일관성이 깨지는 경우가 있었다.

8.0 버전부터는 이런 문제점을 해결하기 위해 테이블 구조 정보나 스토어드 프로그램을 InnoDB의 테이블에 저장하도록 개선했다.

### InnoDB 스토리지 엔진 아키텍처

InnoDB는 MySQL에서 사용할 수 있는 스토리지 엔진 중 유일하게 레코드 기반의 잠금을 제공하여 높은 동시성 처리가 가능하고 안정적이며 성능이 좋다.

![img.png](../images/InnoDB구조.png)

### 프라이머리 키에 의한 클러스터링

- InnoDB의 모든 테이블은 기본적으로 pk를 기준으로 클러스터링되어 저장.
- pk값의 순서대로 디스크에 저장
- 모든 세컨더리 인덱스는 레코드의 주소 대신 pk의 값을 논리적인 주소로 사용

### 외래 키 지원

- 외리 키는 InnoDB 스토리지 엔진 레벨에서 지원하는 기능
- MyISAM이나 MEMORY 테이블에서는 사용 불가
- 운영 환경에서는 외래 키를 권장하지 않는다.
- 외래 키 체크를 끄고 킬 수 있다.
  ```sql
  SET foreign_key_checks=OFF;
  SET foreign_key_checks=ON;
  ```

### MVCC (Multi Version Concurrency Control)

MVCC의 가장 큰 목적은 잠금을 사용하지 않고 일관된 읽기를 제공

트랜잭션 격리 레벨에 따라 동작이 달라진다.

![img.png](../images/InnoDB의버퍼풀과데이터파일의상태.png)

```sql
UPDATE member SET m_area='경기 WHERE' m_id=12;
```

![img.png](../images/update후InnoDB버퍼풀과데이터파일의언두영역의변화.png)

```sql
SELECT * FROM member WHERE m_id=12;
```

격리 수준이 `READ_UNCOMMITTED`인 경우 커밋이 됐거나 안됐거나 InnoDB 버퍼풀이나 데이터 파일로부터 변경되지 않은 데이터를 읽어서 반환한다.
`READ_COMMITTED` 이상의 격리 수준인 경우에는 커밋되기 전의 내용을 보관하고 있는 언두영역의 데이터를 반환한다.

### 잠금 없는 일관된 읽기 (Non-Locking Consistent READ)

InnoDB에서 읽기 작업은 다른 트랙잭션이 가지고 있는 잠금을 기다리지 않고 읽기 작업이 가능

격리 수준이 `SERIALIZABLE`이 아니고 `INSERT`와 연결되지 않은 순수한 읽기 작업은 다른 트랜잭션과 관계없이 바로 실행된다.

### 자동 데드락 감지

InnoDB 스토리지 엔진은 내부적으로 잠금이 데드락이 걸렸는지 잠금 대기 목록을 그래프 형태로 관리하여 체크한다.

InnoDB 스토리지 엔진은 데드락 감지 스레드를 가지고 있고 주기적으로 검사해 데드락이 걸린 트랜잭션들을 찾아서 하나를 강제 종료한다.

언두로그 양이 적은 트랜잭션을 종료시키는데 트랜잭션 강제 롤백으로 인한 부하가 적기 때문

### InnoDB 버퍼 풀

- InnoDB 스토리지 엔진에서 가장 핵심적인 부분이다. 
- 디스크의 데이터 파일이나 인덱스 정보를 메모리에 캐시하는 공간 
- 쓰기 작업을 지연시켜 일괄 작업으로 처리하는 버퍼 역할도 한다.
- 데이터를 변경하는 쿼리를 버퍼 풀에 모아서 처리하면 디스크 작업 횟수도 줄어든다.

### 버퍼 풀의 크기 설정

- 5.7 부터 풀 크기 조절 가능
- 운영체제 메모리 공간이 8GB 미만이면 50% 정도 설정
- 50GB 이상이면 15~30GB 설정
- 풀을 늘리는 작업은 시스템 영향도가 크지 않지만, 줄이는 작업은 매우 크다.
- 128MB 단위로 단위로 설정

~~중략..~~
