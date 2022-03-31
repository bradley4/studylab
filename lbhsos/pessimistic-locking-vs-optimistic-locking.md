# Lock
데이터베이스에 접근해서 데이터를 수정할 때 동시에 수정이 일어나 충돌이 일어날 수 있습니다.
이러한 상황을 제어하기 위해서 트랜잭션의 격리 수준과 비즈니스 로직에 맞추어 Lock(잠금)이 필요합니다.

# Optimistic Lock
### 특징
1. 낙관적인 : 기본적으로 데이터 갱신시 충돌이 발생하지 않을 것이라고 낙관적으로 보는 락
2. 비선점적인: 데이터 갱신시 충돌이 발생하지 않을 것이라고 예상하기 때문에, 우선적으로 락을 걸지 않는다.

### 설명
- DB가 제공하는 락 기능을 사용하지 않고, Application Level(JPA 등)에서 잡아주는 락입니다.
- version 등의 컬럼을 추가하여 여러 트랜잭션 내 하나의 데이터에 중복 업데이트를 확인합니다.
- DB 트랜잭션을 걸지 않기 때문에, 여러 트랜잭션이 동일 데이터에 업데이트를 시도할 수 있습니다.
- 데이터 베이스 수준의 Roll-back이 없기에, 충돌시 대처 방안을 구현해야 합니다.

  ![Optimistic](https://i.stack.imgur.com/DEdlF.png)

### JPA에서의 Optimistic Lock
- JPA에서 낙관적 잠금을 사용하기 위해서는 Entity 내부에 @Version Annotaion이 붙은  Long, Integer, Short, Timestamp Type의 변수를 구현하여줌으로써 간단하게 구현이 가능하다고 합니다.
- 여러 업데이트가 서로 간섭하지 않도록 방지하는 version 이라는 구분 속성을 확인하여 Entity의 변경사항을 감지하는 매커니즘입니다.

#### [예시]
<pre><code>
@Entity
public class Board {
    @Id
    private String id;
    private String title;
    
    @Version
    private Integer version;
}
</code></pre>

JPA가 버전 정보를 비교하는 방법 엔티티를 수정하고 트랜잭션을 커밋할 때, 영속성 컨텍스트를 플러시하면서 UPDATE 쿼리를 실행한다.
<pre><code>
UPDATE BOARD
SET
    TITLE=?,
    VERSION=? (버전 + 1 증가)
WHERE
    ID=?
    AND VERSION=? (버전 비교)
</code></pre>
데이터베이스의 버전과 엔티티의 버전이 같으면 데이터를 수정하면서 동시에 버전도 하나 증가시킨다.

그러나 이미 데이터베이스에 수정이 이루어져 두 버전이 다르면, WHERE 문에서 조건이 달라지므로 수정할 대상이 없어진다. 이 경우 JPA가 예외를 발생시킨다.


# Pessimistic Lock
### 특징
1. 비관적인: 기본적으로 데이터 갱신시 충돌이 발생할 것이라고 비관적으로 보고 미리 잠금을 거는 락
2. 선점적인: 데이터 갱신시 충돌이 발생할 것이라고 예상하기 때문에, 우선적으로 락을 건다.

### 설명
- 트랜잭션이 시작될 때 Shared Lock 또는 Exclusive Lock을 걸고 시작하는 방법으로 DB가 제공하는 락 기능을 사용합니다.
- **DB Transaction을 이용**하여 충돌을 예방하는 것이 바로 비관적 락(Pessimistic Lock)입니다.
- 충돌이 발생하면 Transaction이 실패한 것이기 때문에 트랜잭션 전체에 자동으로 rollback이 일어난다.
  ![Pessimistic](https://i.stack.imgur.com/oybRy.png)

### Shared Lock과 Exclusive Lock
#### Exclusive Lock(배타적 잠금)
- 쓰기 잠금이라고도 불립니다.
- 어떤 트랜잭션에서 데이터를 변경하고자 할 때, 해당 트랜잭션이 완료될 때까지 해당 테이블 혹은 레코드를 다른 트랜잭션에서 읽거나 쓰지 못하게 하기 위해 Exclusive lock을 걸고 트랜잭션을 진행시키는 것입니다.
- 배타적 잠금(exclusive lock)에 걸리면 공유잠금(shared lock)을 걸 수 없습니다.
- 배타적 잠금(exclusive lock)에 걸린 테이블, 레코드 등의 자원에 다른 트랜잭션이 배타적 잠금(exclusive lock)을 걸 수 없습니다.
#### Shared Lock(공유 잠금)
- 읽기 잠금이라고도 불립니다.
- shared lock이 하나라도 걸려 있으면 exclusive lock을 걸 수 없습니다.

## [결론] Optimistic vs. Pessimistic
### 1. 낙관적 락은 어플리케이션 레벨에서 충돌 감지를 할 수 있습니다.
> 낙관적 잠금은 아래와 같은 상황에서 충돌 감지를 할 수 있습니다. 그러나, 비관적 잠금의 경우에는 아래와 같은 경우에서 1번에서 3번 사이의 트랜잭션을 유지할 수 없습니다.
1. 클라이언트가 서버에 정보를 요청
2. 서버에서는 정보를 반환
3. 클라이언트에서 이 정보를 이용하여 수정 요청
4. 서버에서는 수정 적용 ( 충돌 감지 가능 )
   ![Application-level transactions](https://i.stack.imgur.com/FCyHh.png)

### 2.낙관적 락은 트랜잭션을 필요로 하지 않기 때문에 성능적으로도 비관적 락보다 좋습니다.

### 3. 낙관적 락은 성능상 유리하지만 충돌 발생시 롤백을 개발자가 직접 처리해야 하기 때문에 트랜잭션 롤백이 발생하는 비관적 락에 비해서 불리합니다. 따라서 충돌이 많이 발생되지 않고 이를 해결하기 위한 비용이 많지 않은 곳에 사용하는 것이 유리 합니다.



