## 프록시란?
프록시(Proxy)는 대리자라는 뜻으로, 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장하여 클라이언트의 요청을 받아주는 역할을 합니다.  
자바에서 프록시는 **타겟의 기능을 확장**하거나 **타깃에 대한 접근을 제어**하기 위한 목적으로 사용하는 클래스를 말합니다.

*참고) 원래 요청하려는 대상, 즉 최종적으로 요청을 위임받아 처리하는 실제 오브젝트를 타깃이라고 합니다.

### 프록시의 조건
프록시가 되려면 클라이언트는 요청을 보낸 대상이 타깃인지 프록시인지 구분을 할 수 없어야 합니다.  
_즉, 타깃과 프록시는 같은 인터페이스를 확장해야 합니다._

### 프록시 디자인 패턴
1. 데코레이터(Decorator) 패턴
- 타깃에 부가적인 기능을 런타임 시 다이나믹하게 부여해주기 위해 사용하는 프록시 패턴

2. 프록시(Proxy) 패턴
- 타깃에 대한 접근 방법을 제어하려는 목적을 가지고 프록시를 사용하는 패턴을 말합니다.
- 특정 상황에서 타깃에 대한 접근권한을 제어할 수 있다.
- 예를 들어, 특정 조건이 만족되면 타깃의 핵심 로직을 호출하기 전에 예외를 던져서 접근을 불가능하게 만들 수 있습니다.
- 타깃으로부터 응답으로 받은 데이터가 메모리에 존재할 때, 프록시는 타깃으로 요청을 보내지 않고, 기존 응답의 데이터를 클라이언트에게 전달할 수 있습니다.

### 프록시 데코레이터 패턴 예시
Hello 라는 클래스의 메소드가 대문자로 변환된 문자열을 리턴하고 싶어할 때, 프록시를 이용하여 Hello 메소드를 변경하지 않은 채로 부가기능을 추가할 수 있습니다.
- Hello.java
<pre><code>public interface Hello {
    String sayHello(String name);
    String sayHi(String name);
}
</code></pre>

- HelloTarget.java
<pre><code>public class HelloTarget implements Hello {
    @Override
    public String sayHello(String name) {
        return "Hello " + name;
    }

    @Override
    public String sayHi(String name){
        return "Hi " + name;
    }
}
</code></pre>

- HelloUppercase.java
<pre><code>public class HelloUppercase implements Hello {
    HelloTarget helloTarget;
    @Override
    public String sayHello(String name) {
        return helloTarget.sayHello(name).toUpperCase();
    }

    @Override
    public String sayHi(String name){
        return helloTarget.sayHi(name).toUpperCase();
    }
}
</code></pre>
HelloUppercase는 HelloTarget 타입의 객체를 가지고 있으며 HelloTarget에게 위임하여 원래 일을 한 뒤,   
toUpperCase()라는 메소드를 이용하여 '대문자 변환'이라는 부가기능을 덧붙여 리턴합니다.

처음부터 클라이언트에게 Hello라는 타입을 제공하고 이를 사용하게 했다면, 클라이언트는 자신이 타겟을 직접 사용하고 있는지, 프록시를 사용하고 있는지 알 수도 알 필요도 없게 됩니다.

위 예시처럼 타겟 코드의 수정 없이 타겟의 기능을 확장하거나 부가 기능을 추가할 수 있습니다.

## 다이내믹 프록시
다이내믹 프록시는 리플렉션 기능을 이용하여 프록시를 동적으로 생성해줍니다.

### 분류
다이내믹 프록시는 크게 **인터페이스 유무**에 따라 두 가지로 분류할 수 있습니다.
- 인터페이스가 존재하는 경우: JDK 동적 프록시
- 인터페이스가 없는 경우: CGLIB

_Spring에서 사용하는 Dynamic Proxy와 CGLib을 이용한 proxy가 있습니다._

JDK Dynamic Proxy는 Reflection을 통해 동적으로 proxy 객체를 생성합니다.
- Reflection을 통해 동적으로 proxy 객체 생성
- interface 기준으로 proxy 생성

### 기존 프록시 구현의 문제점
1. 인터페이스를 직접 구현해야한다.
- 위 예시에서 Hello라는 인터페이스를 구현하려면 Hello의 모든 메소드를 구현해야한다.
- 만약 sayHi()메소드만 기능을 확장하고 싶다고 하더라도 sayHello()를 모두 override해야한다.

2. 프록시 클래스 내 중복이 발생한다.
- 위 예시에서 부가기능이 '대문자 변환'일 경우, 모든 메소드에 toUpperCase()가 붙게되고 이는 중복으로 보여집니다.

-> _다이나믹 프록시는  위 두 문제를 해결합니다._

## JDK Dynamic Proxy  
### 리플렉션(Reflection)
- 리플렉션이란 객체를 통해 클래스의 정보를 분석해 내는 프로그램 기법을 말합니다. 
- 구체적인 클래스 타입을 알지 못해도 그 클래스의 정보(메서드, 타입, 변수 등등)에 접근할 수 있게 해주는 자바 API입니다.

#### 리플렉션이 가능한 이유 
자바에서는 JVM이 실행되면 사용자가 작성한 자바 코드가 컴파일러를 거쳐 바이트 코드로 변환되어 static 영역에 저장됩니다.  
Reflection API는 이 정보를 활용합니다. 그래서 클래스 이름만 알고 있다면 언제든 static 영역을 뒤져서 정보를 가져올 수 있는 것입니다.

#### 리플렉션 단점
- 성능 오버헤드
: 컴파일 타임이 아닌 런타임에 동적으로 타입을 분석하고 정보를 가져오므로 JVM을 최적화할 수 없기 때문입니다. 
- 추상화 깨짐 
: 직접 접근할 수 없는 private 인스턴스 변수, 메서드에 접근하기 때문에 내부를 노출하면서 추상화가 깨집니다. 

### Java.lang.reflect.Proxy 의 newProxyInstance()
![Proxy docs](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FJSesa%2FbtqLGYRdhho%2FbTF6AEGyjT1vPvWZmva7o0%2Fimg.png)
첫 번째 인자: 프록시를 만들 클래스 로더  
두 번째 인자: 어떤 인터페이스에 대해 프록시를 만들 것인지 명시  
세 번째 인자: InvocationHandler 인터페이스의 구현체  
리턴 값: 동적으로 만든 프록시 객체  

-> 일일이 구현해야 한다는 문제는 reflectionAPI가 해결해주고, 중복은 InvocationHandler가 해결해줍니다.

위 예시에서 다이나믹 프록시를 적용한다면 아래와 같습니다.
<pre><code>Hello hello = (Hello) Proxy.newProxyInstance(
    Main.class.getClassLoader(),
    new Class[]{Hello.class},
    new UppercaseHandler(new HelloTarget())
);
</code></pre>

### InvocationHandler
InvocationHandler는 invoke()라는 메소드 하나만 가지고 있는 인터페이스입니다.  
invoke() 메소드는 다이나믹하게 생성될 프록시의 어떤 메소드든 호출됐을 때 호출되는 메소드입니다.  
따라서, **여기서 어떤 메소드에 기능을 확장할지 결정할 수 있고, 확장된 기능을 구현할 수도 있습니다.**
<pre><code>
public class UppercaseHandler implements InvocationHandler {
    Object target;
    public UppercaseHandler(Object target) {
        this.target = target;
    }       
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object ret = method.invoke(target, args);
        if (ret instanceof String && method.gatName().startWith("say")) {
            return ((String) ret).toUpperCase();
        } else {
            return ret;
        }
    }
}
</code></pre>

첫 번째 인자: 프록시 객체  
두 번째 인자: 메소드 객체(클라이언트가 호출한 메소드)  
세 번째 인자: 메소드의 인자(클라이언트가 메소드에 전달한 인자)

### [정리] 다이나믹 프록시 호출 과정
![다이나믹 프록시 호출 과정](https://blog.kakaocdn.net/dn/cBNw54/btqLFTpclqB/GlBiJpf5s3GRMFWK9SWw5K/img.png)

1. 클라이언트는 Hello 타입의 객체에게 sayHi()를 요청한다. (클라이언트는 프록시인지 타겟인지 모른다.)
2. 다이나믹하게 만들어진 프록시 객체는 사용자가 요청한 메소드의 정보와 메소드의 인자를 InvocationHandler 구현체의 invoke() 메소드의 인자로 전달하여 호출한다.
3. InvocationHandler의 invoke() 메소드에서 타겟의 어떤 메소드에 적용할지 검사하고, 적용해야 하는 메소드라면 타겟의 원래 메소드를 호출하여 결과를 담아둔 뒤 추가 기능을 적용하여 리턴한다. 적용하지 말아야 하는 메소드라면 타겟의 원래 메소드의 리턴을 그대로 리턴한다.

## CGLib
CGLIB는 JDK동적 프록시와 달리 바이트 코드를 조작해서 동적으로 클래스를 생성해주는 라이브러리 입니다.   
또한 인터페이스 없이 구체 클래스만 존재하더라도 프록시를 생성할 수 있습니다.
### MethodInterceptor
CGLIB는 MethodInterceptor 를 통해 구현합니다.
(JDK Dynamic Proxy 의 InvocationHandler 와 비슷합니다.)
<pre><code>public interface MethodInterceptor extends Callback{
    Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable;
}
</code></pre>
MethodInterceptor는 단일 메서드 인터페이스로 JDK 동적 프록시의 InvocationHandler와 굉장히 유사한 구조와 동작 방식을 가지고 있습니다.
- obj : CGLIB가 적용된 객체입니다.
- method : 호출된 메서드입니다.
- args : 호출된 메서드의 매개변수입니다.
- proxy : 실제 타깃의 메서드를 invoke 할 때 사용됩니다. method를 이용하여 실행이 가능하지만 성능상 proxy.invoke()를 권장합니다.

### CGLib vs. JDK Dynamic Proxy
- 이 방식은 JDK Dynamic Proxy와는 다르게 Reflection을 사용하지 않고, Extends(상속) 방식을 이용해서 Proxy화 할 메서드를 오버라이딩 하는 방식입니다.
- CGlib은 기본적으로 Byte 코드를 조작해서, 바이너리가 만들어지기 때문에 JDK Dynamic Proxy보다 성능적으로 더 우세합니다
- JDK Dynamic Proxy는 Invocation Handler의 실체를 구현하게 되는데, 특정 Object에 대해 Reflection을 사용하기 때문에 성능이 조금 떨어집니다.

## Spring AOP  
Spring AOP에서는 JDK Dynamic Proxy 와 CGlib 을 통해 Proxy화 합니다.  

AOP는 관점지향 프로그래밍이라는으로 "기능을 핵심 비즈니스 기능과 공통기능으로 '구분’하고,모든 비즈니스 로직에 들어가는 공통기능의 코드를 개발자의 코드 밖에서 필요한 시점에 적용하는 프로그래밍 방식입니다.

### AOP 용어 
1. 조인포인트(Joinpoint): 클라이언트가 호출하는 모든 비즈니스 메소드, 조인포인트 중에서 포인트컷이 되기 때문에 포인트컷의 후보라고 할 수 있습니다.  
   - 스프링은 프록시를 이용해서 AOP를 구현하기 때문에 메서드 호출에 대한 Join point만 지원합니다.
2. 포인트컷(Pointcut): 특정 조건에 의해 필터링 된 조인포인트, 수많은 조인포인트 중에 특정 메소드에서만 공통기능을 수행시키기 위해 사용됩니다. 
3. 어드바이스(Advice): 공통기능의 코드, 독립된 클래스의 메소드로 작성합니다.
4. 위빙(Weaving): 포인트컷으로 지정한 핵심 비즈니스 로직을 가진 메소드가 호출될 때, 어드바이스에 해당하는 공통기능의 메소드가 삽입되는 과정을 의미합니다. 위빙을 통해서 공통기능과 핵심 기능을 가진 새로운 프록시를 생성하게 됩니다.
5. Aspect: 포인트컷과 어드바이스의 결합입니다. 어떤 포인트컷 메소드에 대해 어떤 어드바이스 메소드를 실행할지 결정합니다.

### Spring AOP - Proxy
JDK Dynamic Proxy에서 InvocationHandler, CGlib 에서 MethodInterceptor 는 Spring AOP에서 **JoinPoint** 라는 개념과 일치합니다
그리고, 위에서 특정 조건에 의해 필터링 하는 로직은 Spring AOP에서 **PointCut** 라는 개념과 일치합니다
마지막으로 Proxy 로직이 실행되는 JDK Dynamic Proxy에 invoke 메서드, CGlib 에서 intercept 메서드는 Spring AOP에서 **Advice** 라는 개념과 일치합니다