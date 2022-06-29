## 왜 primitive type은 제네릭(Generic)으로 사용할 수 없을까?

### 제네릭(Generic) 이란?

- 클래스나 메서드에서 사용할 내부 데이터 타입을 외부에서 지정하는 기법으로 Java 5에서 추가된 기능
- 컴파일 시점에 객체의 타입을 체크하여 타입 안정성을 높이고 형변환을 하지 않아도 되는 특징이 있음
- 로 타입(raw type)과 매개변수화 타입(parameterized type)으로 정의됨
```java
// java.util.List 코드
public interface List<E> extends Collection<E> {...}
// List - 로 타입 (raw type)
// E - 매개변수화 타입 (parameterized type)

    List<String> list = new ArrayList<>(); // String 이 매개변수화 타입
```
---
#### 제네릭의 장점 예시
```java
/* 제네릭이 추가되기 전 컬렉션 사용법 */
List list = new ArrayList();
list.add(1);
list.add("2");

Integer aa = (Integer) list.get(0); // 명시적 형변환 필요
Integer bb = (Integer) list.get(1); // 런타임 에러 발생!!!
```

```java
/* 제네릭 적용 후 */
List<Integer> list = new ArrayList();
list.add(1);
list.add("2"); // 컴파일 에러 발생!!!

Integer aa = (Integer) list.get(0);
Integer bb = (Integer) list.get(1);
```
---
### 타입 이레이저 (Type Erasure)

- 제네릭 도입 이전 버전의 호환을 위해 도입된 프로세스
- 컴파일 시점에 제네릭 타입을 제거하여 컴파일된 소스는 1.4 JVM 에서도 실행가능하도록 함
- 타입의 안정성은 컴파일 시점에 확인하였기 때문에 런타임 시점엔 타입을 고려하지 않아도 됨

```java
// 작성된 소스코드
public class Store<T> {
    private List<T> list = new ArrayList<>();

    public void add(T t) {
        list.add(t);
    }
    public T get(int i) {
        return list.get(i);
    }
}
```

```java
// <>이 사라지고 T는 최상위 클래스인 Object 타입으로 대체
public class Store {
    private List list = new ArrayList();
    
    public void add(Object t) {
        list.add(t);
    }
    public Object get(int i) {
        return list.get(i);
    }
}
```
---

### 결론
이러한 내부 변환 과정에서 제네릭 타입은 상위 한정제한을 두지 않는다면 최상위 클래스인 Object로 형변환이 되어야하는데
기본 타입(primitive type)은 Object를 상속받지 않아 변환이 불가능하여 제네릭 타입으로 사용이 불가능하다.

기본 타입으로 제네릭 클래스 또는 인터페이스를 사용하고 싶은 경우,
각 기본 타입의 래퍼 클래스(wrapper class)를 사용하면 제네릭 타입을 사용할 수 있다. 

---

### 로 타입 사용 지양

이펙티브 자바 아이템 26 에서 **로 타입은 사용하지 말라** 는 내용이 있습니다.

매개변수 타입을 명시적으로 지정하지 않고 
**로 타입(raw type)을 사용하면 제네릭이 안겨주는 안정성과 표현력을 모두 잃게 된다**고 합니다.

따라서, 제네릭 타입을 사용하는 경우엔 매개변수 타입을 명시적으로 지정하여 사용하는 것을 권장!

### 제네릭 변성

- 제네릭에서의 상위타입 / 하위타입에 관련된 내용
- 링크 참고

---
### 참고
- [[Java] 제네릭 (1) - 타입이레이저, 변성 ( + 제네릭 메서드는 왜 오버로딩이 안될까?)](https://ttl-blog.tistory.com/282#--%--%EC%A-%-C%EB%--%A-%EB%A-%AD%--%ED%--%--%EC%-E%--%EC%-D%--%--%EA%B-%BD%EA%B-%---bound-%EB%A-%BC%--%EC%A-%-C%EA%B-%B-%ED%--%A-%EB%-B%--%EB%-B%A--)
- [[이펙티브 자바 3판] 아이템 26. 로 타입은 사용하지 말라](https://madplay.github.io/post/dont-use-raw-types)