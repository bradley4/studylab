
# 영속성(Persistence)
데이터를 생성한 프로그램이 종료되더라도 사라지지 않는 데이터의 특성을 말한다.  
영속성을 갖지 않는 데이터는 단지 메모리에서만 존재하기 때문에 프로그램을 종료하면 모두 잃어버리게 된다.   
때문에 파일 시스템, 관계형 테이터베이스 혹은 객체 데이터베이스 등을 활용하여 데이터를 영구하게 저장하여 영속성 부여한다.  

# Persistence Layer
프로그램의 아키텍처에서, 데이터에 영속성을 부여해주는 계층을 말한다.  
JDBC를 이용하여 직접 구현할 수 있지만 Persistence framework를 이용한 개발이 많이 이루어진다.

# 계층
프레젠테이션 계층 (Presentation layer) - UI 계층 (UI layer) 이라고도 한다.  
애플리케이션 계층 (Application layer) - 서비스 계층 (Service layer) 이라고도 한다.  
비즈니스 논리 계층 (Business logic layer) - 도메인 계층 (Domain layer) 이라고도 한다.  
데이터 접근 계층 (Data access layer) - 영속 계층 (Persistence layer) 이라고도 한다.  

# DAO 란 무엇일까?
## 배경
Dao(Data Access object)는 스프링의 아버지라 부를 수 있는 J2EE에서 등장한 개념이다.

애플리케이션을 사용하다보면 영구저장소 매커니즘이 필요할 때가 매우 많아진다. 이때 영구저장소의 구현체는 정말 무수히 많다. (Oracle, Mysql, MongoDB)

그렇다면 어플리케이션에서 영구저장소에 접근하기 위해서는 각 영구 저장소 벤더에서 제공하는 API를 통해서 접근하면 된다.

그러나 영구저장소 API 를 사용하게 되면 아래와 같은 문제가 생긴다.

## 문제점
1. 구현체와 로직이 너무 강한 결합을 가지게 된다.  
예를 들어, MariaDB 를 사용하다가 Oracle을 사용하게 된다면 MariaDB의 API를 사용한 모든 구현을 변경해야 한다.

2. 레이어가 깨진다.  
MariaDB API를 서비스 로직에서 사용하기 위해 new 하여 생성하였다고 가정해보면, Service 로직을 담당하는 객체와 db와 관련된 api가 강한 결합을 갖게 되며 영속성과 관련된 로직이 서비스 로직에 생성되어 계층이 섞인다.

3. 개발자 러닝커브가 증가한다.  
벤더마다 api가 달라서 개발자는 사용할 때마다 학습해야한다.

## 등장
이러한 문제점을 해결하기 위해 나온 패턴이 DAO패턴이다. DAO 벤더들의 API와 로직 사이의 어댑터와 같은 역할을 한다. 

벤더 구현체를 DAO 객체로 한 번 더 감쌈으로써 데이터 소스가 변경되더라도 그 로직에는 변화가 없다.  
DAO가 어댑터의 역할을 수행함으로써 벤더들 사이의 구현 차이점을 극복할 수 있다.

![다이어그램](https://www.oracle.com/ocom/groups/public/@otn/documents/digitalasset/145996.jpg)

## 정리
비즈니스 로직과 퍼시스턴스 로직의 명확한 분리를 위해 퍼시스턴스 로직을 캡슐화하고 도메인 레이어에 객체-지향적인 인터페이스를 제공하는 객체를 DAO(Data Access Object) 라고 한다.  

# Repository란 무엇인가?
- Eric Evans의 Domain-Driven Design에 따르면, “Repository는 객체의 Collection을 저장, 검색 하는 등의 동작을 캡슐화 한다” 고 한다.  
- Repository는 객체의 상태를 관리하는 저장소이다. 
- Repository는 도메인 레이어에 속한다. 

일반적으로 우리는 Repository를 영구저장소처럼 사용하는데, 이는 Repository의 구현체가 영구저장소 API를 이용해 구현하기 때문이다.   
(클라이언트는 Repository의 인터페이스만을 사용하고, 그 내부 구현은 모르기 때문에 계층이 섞이는 것이 아니다.)  

우리가 일반적으로 사용하는 관점에서 Repository의 Interface는 도메인 레이어,  Repository의 구현체는 영속성 레이어에 속한다  

고수준모듈에서 저수준 모듈에 의존하게 된다. 따라서 도메인 레이어에 속해있는 Repository를 Application 레이어에서 사용하는건 전혀 어색하지 않다. 따라서 서비스로직에서 Repository의 인터페이스를 사용할 수 있게 된다.  

# 예시
## 1. DAO
```java
public interface UserDao {
    void create(User user);
    User read(Long id);
    void update(User user);
    void delete(String userName);
}
```
```java
public class UserDaoImpl implements UserDao {
    private final EntityManager entityManager;
    
    @Override
    public void create(User user) {
        entityManager.persist(user);
    }

    @Override
    public User read(long id) {
        return entityManager.find(User.class, id);
    }

    // ...
}
```

- DAO의 인터페이스는 데이터베이스의 CRUD 쿼리와 1:1 매칭되는 세밀한 단위의 오퍼레이션을 제공한다.
- DAO는 데이터 매핑/접근 계층으로 작동하여 쿼리를 숨긴다.

## 2. Repository
```java
public interface UserRepository {
    User get(Long id);
    void add(User user);
    void update(User user);
    void remove(User user);
}
```
```java
public class UserRepositoryImpl implements UserRepository {
    private UserDaoImpl userDaoImpl;
    
    @Override
    public User get(Long id) {
        User user = userDaoImpl.read(id);
        return user;
    }

    @Override
    public void add(User user) {
        userDaoImpl.create(user);
    }

    // ...
}
```
- 리포지토리는 도메인과 데이터 액세스 레이어 사이의 레이어로 데이터를 조합하고 도메인 객체를 준비하는 복잡성을 숨긴다.  
- repository는 여러 DAO를 사용해 구현될 수 있지만, 그 반대의 경우는 불가능하다.  
- Repository는 보통 한정된 인터페이스다. 단순히 Get(id), Find(Specification), Add(Entity)와 같은 객체들의 collection이다.  

### Repository 예시 2
(위 예시가 너무 체감이 안와서 찾아본 예시..)  
예를 들어 PetType Table(name: 고양이, type: 포유류) Pet Table(name: 냐옹이, breed: 페르시안 고양이)과 같은 모델이 있다고 가정했을 때, PetTypeDao, PetDao가 있을 것이다. 

||PetType|
|---|---|
|name|	고양이|
|type|	포유류|


||Pet|
|---|---|
|name|	야옹이|
|breed|	페르시안 고양이|

만약 PetType에 없는 종류의 Pet을 DB에 추가하고 싶을 경우(예를 들면, 푸들), 
Dao 상위에 Repository interface에 메서드를 생성해 두 개의 DAO를 묶을 수 있다고 한다.


## 궁금한점
1. 우리가 현재 쓰는 Repository 는 그러면 인터페이스는 Repository 이고, 그 내부는 DAO 패턴으로 이해하는게 맞을까?
2. Repository 패턴을 컬렉션으로 이해한다면, 조회 및 연산에 관련한 메서드만이 존재하는게 맞을까? (update 쪽은 DAO 쪽으로 이동?)
