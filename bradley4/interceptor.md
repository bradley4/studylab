# INTERCEPTOR VS FILTER

![image.png](https://justforchangesake.files.wordpress.com/2014/05/spring-request-lifecycle.jpg)
> https://justforchangesake.wordpress.com/2014/05/07/spring-mvc-request-life-cycle/

### Filter
- Filter는 Web Application에 등록한다.
- FIlter는 servlet에 도달하기전의 요청을 조작하거나 응답을 조작할 수 있다.
- 인코딩 변환이나 XSS 방어 처리 

### Interceptor
- Spring의 context에 등록
- Spring MVC framework의 부분이다. 
- 디스패쳐 서블릿과 컨트롤러 사이에서 요청 처리.
- 로그인, 권한, 실행시간, 로그 작업

### Interface

- Filter

``` java
public interface Filter {
  void doFilter(ServletRequest request, ServletResponse response, FilterChain chain);
}
```
- HandlerInterceptor

``` java
public interface HandlerInterceptor {
  boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler);

  void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView mav);

  void afterCompletion(HttpServletRequest request, HttpServeletResponse response, Object handler, Exception ex);
}
```

### 차이점은?
- Interceptor에서만 할 수 있는 것
1. AOP 흉내를 낼 수 있다. handler라는 이름으로 HandlerMethod가 들어온다. HandlerMethod로 메서드 시그니처 등 추가적인 정보를 파악해서 로직 실행 여부를 판단할 수 있다.
2.  View를 렌더링하기 전에 추가 작업을 할 수 있다. 예를 들어 웹 페이지가 권한에 따라 GNB(Global Navigation Bar)이 항목이 다르게 노출되어야 할 때 등의 처리를 하기 좋다.

- Filter에서만 할 수 있는 것
1. ServletRequest 혹은 ServletResponse를 교체할 수 있다. 


### 고민해볼점?
- Filter, Interceptor, AOP 중에 어떤 전략을 가져갈 것인가?
- AOP는 파라미터, 주소, 애노테이션등 다양한 전략 활용가능. 

### see also
> https://www.baeldung.com/spring-mvc-handlerinterceptor-vs-filter
