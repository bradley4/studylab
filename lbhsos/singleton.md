# Singleton pattern 이란 
- Singleton pattern(싱글턴 패턴)이란 애플리케이션에서 인스턴스를 하나만 만들어 사용하기 위한 패턴이다.
- 커넥션 풀, 스레드 풀 등의 경우, 인스턴스를 여러 개 만들게 되면 자원을 낭비하게 되거나 버그를 발생시킬 수 있으므로 오직 하나만 생성하고 그 인스턴스를 사용하도록 하는 것이 이 패턴의 목적이다.

```java
public class Singleton {
	public static final Singleton INSTANCE = new Singleton();
 
	private Singleton() { }
 
	...
}
 
// 혹은 ...
public class Singleton {
	private static final Singleton INSTANCE = new Singleton();
 
	private Singleton() { }
 
	public static Singleton getInstance() {
		return INSTANCE;
	}
 
	...
}
```
## static, final
- final: 재할당을 막는다.
- static: Heap 영역이 아닌 GC 대상이 아닌 static 영역에 할당, 프로그램 종료시까지 남는다.

```java
// 생성자의 접근 범위가 private이기 때문에 생성자로는 객체를 생성할 수 없다.
// Singleton obj = new Singleton(); (X)

// INSTANCE는 final로 선언되었기 때문에 외부에서 다시 지정하는 것은 불가능하다.
// Singleton.INSTANCE = null; (X)

// 외부에서 정적 필드로 다음과 같이 접근할 수 있다.
Singleton.INSTANCE.service();

// 혹은...
        Singleton obj = Singleton.getInstance();
        obj.service();
```

## 지연 초기화(lazy initialization)
지연 초기화는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 방법이다. 값이 전혀 쓰이지 않으면 초기화도 결코 일어나지 않는다.  
아래 예시에서는 Singleton.getInstance() 메서드 호출 시점에 정적 필드가 아직 초기화되지 않았으면 객체를 생성하고, 그 후에는 전에 생성한 객체의 참조(reference)를 그대로 반환합니다.  
대부분의 경우에는 초기화 지연보다는 일반적으로 클래스를 생성하면서 초기화 하는 것이 좋다.    
초기화 비용이 크고, 내부적으로 필드 사용 빈도가 적다면 초기화 지연이 적절하다.

```java
public class Singleton {

  private static Singleton instance; // 단 하나의 인스턴스만 사용

  private Singleton() {} // private 생성자. 외부에서 인스턴스 생성못함.

  public static Singleton getInstance() { // 단하나의 인스턴스만 사용

    if (instance == null){ // 여러 스레드에서 이 곳을 동시에 실행하면 문제 발생
      instance = new Singleton();
    }
    return instance;
  }
}
```

## 장점
1. 메모리 
- 최초 한번의 new 연산자를 통해서 고정된 메모리 영역을 사용하기 때문에 추후 해당 객체에 접근할 때 메모리 낭비를 방지할 수 있다. 
- 뿐만 아니라 이미 생성된 인스턴스를 활용하니 속도 측면에서도 이점이 있다고 볼 수 있다.

2. 다른 클래스 간에 데이터 공유가 쉽다. 
- 싱글톤 인스턴스는 전역으로 사용되는 인스턴스이기 때문에 다른 클래스의 인스턴스들이 접근하여 사용할 수 있다. 
- 하지만 여러 클래스의 인스턴스에서 싱글톤 인스턴스의 데이터에 동시에 접근하게 되면 동시성 문제가 발생할 수 있으니 이점을 유의해서 설계하는 것이 좋다.

## 단점
1. 멀티 스레드 환경에서 동기화 문제가 발생할 수 있다. 
- 문제 해결을 위해 synchronized, double checked Locking, LazyHolder 방식을 사용할 수 있다.

2. private 생성자이므로 상속이 불가능하여 객체지향의 특징을 활용할 수 없다.
- 자식클래스를 만들수 없다는 점과, 내부 상태를 변경하기 어렵다는 점
- 싱글톤 패턴은 유연성이 많이 떨어지는 패턴이다.

3. 의존 관계상 클라이언트가 구체 클래스에 의존
- new 키워드를 직접 사용하여 클래스 안에서 객체를 생성하고 있으므로, 이는 SOLID 원칙 중 DIP를 위반하게 된다.

4. private 생성자로 싱글톤 객체 생성에 제한적이기 때문에 인터페이스를 구현한 싱글톤 객체가 아니라면 mock객체 (가짜 객체)를 만들 수 없어 이를 사용해 테스트하기 어렵다.

# 싱글턴 코드는 테스트하기가 어렵다
- 정적 필드는 한번 할당되면 보통은 프로그램이 종료되기 전까지 계속 살아있게 된다. 
- 각 테스트는 독립적, 즉 다른 테스트에 영향을 미치지 않아야 하는데 한 테스트에서 싱글턴 객체가 만들어지면 그 이후의 다른 테스트에서도 이를 확인할 수 있다.
- 일반적으로 인터페이스가 아닌 클래스를 통해 구현되는 싱글턴은 목(mock)으로 대체할 수 없으므로 단위 테스트를 매우 까다롭게 만듭니다.

# 싱글톤에서 주의할 점
- 멀티 스레드 환경에서 여러 스레드가 싱글톤에 동시접근 할 수 있기 때문에 상태 관리를 주의해야한다.
- 대부분은 stateless하게 만들어져야 한다. 내부 상태값의 동시수정이 이루어지는 경우 매우 위험하기 때문이다.

# 싱글턴이 좋지 않다는데 왜 스프링 프레임워크 같은 녀석들은 별다른 규칙이 없을 때 기본으로 Singleton bean 을 만들까요?
  애플리케이션 컨텍스트가 싱글톤으로 빈을 관리하는 이유는 대규모 트래픽을 처리할 수 있도록 하기 위함이다.  
  스프링은 최초에 설계될 때 부터 대규모의 엔터프라이즈 환경에서 요청을 처리할 수 있도록 고안되었다.   
  그리고 그에 따라 계층적으로 처리 구조(Controller, Service, Repository 등) 가 나뉘어지게 되었다.  
  그런데 매번 클라이언트에서 요청이 올 때마다 각 로직을 처리하는 빈을 새로 만들어서 사용한다고 생각해보자.   
  요청 1번에 5개의 객체가 만들어진다고 하고, 1초에 500번 요청이 온다고 하면 초당 2500개의 새로운 객체가 생성된다.   
  아무리 GC의 성능이 좋아졌다 하더라도 부하가 걸리면 감당이 힘들 것이다.   
  그래서 이러한 문제를 해결하고자 빈을 싱글톤 스코프로 관리하여 1개의 요청이 왔을 때 여러 쓰레드가 빈을 공유해 처리하도록 하였다.   

  그래서 스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공하는데, 그것이 바로 __싱글톤 레지스트리(Singleton Registry)__ 이다.  
  스프링 컨테이너는 싱글톤을 생성하고, 관리하고 공급하는 컨테이너이기도 하다.  

#  싱글톤 레지스트리(Singleton Registry)
- 스프링 컨테이너가 싱글톤 레지스트리의 역할을 하여 빈을 싱글톤으로 관리
- private 생성자로 객체의 생성을 막는 방법이 아니라 일반 자바 클래스를 싱글톤으로 활용할 수 있도록 지원한다.
- public 생성자를 사용할 수 있기 때문에 필요하다면 새로운 오브젝트를 생성하고 Mock 오브젝트로 대체하는 등의 작업을 할 수 있다.
- 객체지향적 설계와 디자인 패턴 적용이 가능하다.
- 스프링은 Service, Repository, Controller같은 경우 대부분 상태 변수가 없이 무상태성으로 클래스를 설계하여 동시성 문제를 막을 수 있다. 변경이 일어날 수 있는 정보들에 대해서는 파라미터, 지역변수, 리턴값등을 활용하면 된다.

## 싱글톤에서 필요한 정보는 상태없이 어떻게 관리할까?
- 파라미터, 로컬 변수, 리턴 값 등을 이용한다.
  - 위 변수들은 스택 영역에 독립적으로 저장이 되기 때문에 스레드마다 분리되어 있다.

# 스프링의 빈의 scope는 모두 싱글톤일까
## 우선, scope 란?
scope는 범위라는 뜻의 용어이고 여기서는 스프링의 빈이 1) 언제 생성되고 2) 언제까지 존재하며 3) 어디까지 적용되는지를 말한다.
기본적인 스프링 빈의 scope는 싱글톤이며 빈의 생명주기는 스프링 컨테이너와 함께한다.
## 이외의 scope도 존재한다!
- 프로토타입 Prototype scope : 컨테이너에 빈을 요청할 때마다 매번 새로운 오브젝트를 생성
- 요청 Request scope : 새로운 HTTP 요청이 생길 때마다 생성
- 세션 Session scope : 웹의 세션과 유사하게 생성

# 멀티스레드 동시성 해결방법
## synchronized 사용
```java
public class Singleton {

  private static Singleton instance;

  private Singleton() {}

  public static synchronized Singleton getInstance() {  // synchronized 추가
    if (instance == null){
      instance = new Singleton();
    }
    return instance;
  }
}
```
문제점 : 성능의 문제가 있다. 
여러 클라이언트 요청이 있을 때 getInstance() 메서드에 static synchronized를 적용했기 때문에 Singleton 클래스 자체를 락을 걸었다. 
클래스를 사용할 때 단 하나의 스레드만 사용하게 되는 것이다. 
그럼 다른 스레드는 락이 풀릴 때까지 기다려야 한다.

## Double checked Locking의 문제
```java
public class Singleton {
  private static Singleton instance; // (1)

  private Singleton() { }

  // 코드가 다소 장황하지만 동기화 오버헤드를 피할 수 있으므로 성능 이점이 있다.
  public static Singleton getInstance() {
    if (instance == null) {
      // 동기화 블록은 한번에 하나의 스레드만 접근할 수 있다.
      synchronized (Singleton.class) {
        if (instance == null) {
          instance = new Singleton();
        }
      }
    }
    return instance;
  }
 
	...
}
```
하지만 double checked 방식은 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시 volatile로 선언해서 가시성을 확보해야 한다. 
특히 멀티 코어 환경에서 동작한다면 스레드 별 cpu cache와 메인 메모리 간에 동기가 이뤄지지 않을 수 있다.

예를 들어

1. 첫번째 스레드가 instance를 생성하고 synchronized를 벗어남.
2. 두번째 스레드가 synchronized 블록에 들어와서 null 체크를 하는 시점에서,
3. 첫번째 스레드에서 생성한 instance가 CPU cache가 있는 working memory에만 존재하고 main memory에는 존재하지 않을 경우
4. 또는, main memory에 존재하지만 두번째 스레드의 working memory에 존재하지 않을 경우
5. 즉, 메모리 간 동기화가 완벽히 이루어지지 않은 상태라면 두번째 스레드는 인스턴스를 또 생성하게 된다.
따라서 volatile을 붙여야 thread safe하게 사용할 수 있다.
```java
public class Singleton {
  private static volatile Singleton instance;

  private Singleton() { }

  public static Singleton getInstance() {
    if (instance == null) {
      synchronized (Singleton.class) {
        if (instance == null) {
          instance = new Singleton();
        }
      }
    }
    return instance;
  }
 
	...
}
```
## 요청 시 초기화 홀더 패턴 INITIALIZATION-ON-DEMAND HOLDER PATTERN
```java
public class Singleton {

  private Singleton() { }

  private static final class Holder {
    private static final Singleton INSTANCE = new Singleton();
  }

  public static Singleton getInstance() {
    return Holder.INSTANCE;
  }
 
	...
}
```
Holder 클래스에 선언된 정적 필드인 INSTANCE가 사용될 때 Holder 클래스의 초기화가 일어난다.   
즉, 위의 예시에서는 런타임에 Singleton.getInstance()를 호출하여 Holder.INSTANCE을 사용하기 전에 클래스로더를 통해 Holder 클래스의 초기화가 일어나게 된다.   
그와 동시에 Holder 클래스의 초기화 단계에서 정적 필드 INSTANCE의 초기화가 단 한 번만 일어난다.
