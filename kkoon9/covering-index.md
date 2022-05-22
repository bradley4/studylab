# 커버링 인덱스란?

커버링 인덱스는 쿼리의 조건을 충족시키는데 필요한 모든 데이터들을 인덱스에서만 추출할 수 있는 인덱스를 의미합니다.

커버링 인덱스는 B-Tree 인덱스를 스캔하는 것만으로도 원하는 데이터를 가져 올 수 있으며, 컬럼을 읽기 위해 디스크에 접근하여 데이터 블록을 읽지 않아도 됩니다.

인덱스는 행 전체의 크기보다 훨씬 작으며, 인덱스의 값에 따라 정렬이 되기 때문에 Sequential Read 접근이 가능해집니다.

따라서 커버링 인덱스를 활용하면 쿼리의 성능을 비약적으로 향상시킬 수 있습니다.

예시를 위해 예전 진행했던 프로젝트 내 테이블의 DDL을 가져왔습니다.

```sql
-- auto-generated definition
create table app_push_user
(
    id                  int auto_increment
        primary key,
    send_code           varchar(50)            null,
    push_date           datetime               null,
    mobile_os_type      varchar(200)           null,
    mobile_device_token varchar(200)           null,
    send_yn             varchar(2)             null,
    title               varchar(50)            null,
    message             varchar(50)            null,
    active              varchar(3) default 'Y' not null,
    CherishId           int                    null,
    UserId              int                    null,
    createdAt           datetime               not null,
    updatedAt           datetime               not null
);

create index userId_cherishId_send_yn_send_code_idx
    on app_push_user (UserId, CherishId, send_yn, send_code);
```

app_push_user 테이블에는 UserId, CherishId, send_yn, send_code가 index로 걸려 있습니다.

```sql
EXPLAIN SELECT * FROM app_push_user;
```

위 EXPLAIN 쿼리의 결과는 다음과 같습니다.

![Untitled](./not-index.png)

```sql
EXPLAIN SELECT UserId, CherishId, send_yn, send_code FROM app_push_user;
```

위 EXPLAIN 쿼리의 결과는 다음과 같습니다.

![Untitled](index.png)
