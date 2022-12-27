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

### 2.4 서버 설정

