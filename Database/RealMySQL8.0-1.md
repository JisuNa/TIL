# RealMySQL 8.0 1권

## 01 소개

### 1.2 왜 MySQL인가?
자기가 가장 잘 활용할 수 있는 DBMS가 가장 좋은 DBMS이다.
- 안정성
- 성능과 기능
- 커뮤니티나 인지도

## 02 설치와 설정

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

## 03 사용자 및 권한

### 계정 생성

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

