## **stereotype annotation이 무엇일까?**

전체 아키텍처에서 type 혹은 method의 역할을 나타내는 어노테이션을 뜻합니다.

종류로는 Component, Controller, Indexed, Repository, Service가 있습니다.

### **1. Indexed**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Indexed {
}
```

어노테이션이 달린 요소가 index에 대한 stereotype임을 나타냅니다.

CandidateComponentsIndex는 컴파일 시 생성된 메타데이터 파일을 사용하는 classpath scanning 대상입니다.

index를 사용하면 stereotype에 따라 candidate components(정규화된 이름)를 검색할 수 있습니다.

Indexed 어노테이션은 색인화하도록 제너레이터에 지시합니다.

- 어노테이션이 달린 요소(element)
- 주석이 달린 요소를 implements 혹은 extends하는 요소(element)

stereotype은 어노테이션이 달린 요소의 정규화된 이름입니다.

meta-annotated를 사용하는 디폴트 컴포넌트 어노테이션을 Indexed 어노테이션으로 여깁니다.

meta-annotated는 어노테이션을 선언할 때 사용하는 어노테이션이라고 합니다.

예시로, @Target, @Retention, @Inherited 등이 있습니다.

Component 어노테이션이 있는 component인 경우, org.springframework.stereotype.Component sterotype을 사용하여 해당 component에 대한 항목이 Index에 추가됩니다.

Indexed 어노테이션은 meta-annotation에도 적용됩니다.

```java
package com.example;

 @Target(ElementType.TYPE)
 @Retention(RetentionPolicy.RUNTIME)
 @Documented
 @Indexed
 @Service
 public @interface PrivilegedService { ... }
```

위에 어노테이션이 type에 있으면 두 가지 stereotype로 index됩니다.

- org.springframework.stereotype.Component
- com.example.PrivilegedService

Service 어노테이션은 인덱싱된 상태로 직접 annotated되지는 않지만 Component를 사용하여 meta-annotated됩니다.

@Indexed를 추가하여 특정 인터페이스의 모든 구현체 혹은 특정 클래스의 모든 하위 클래스를 인덱싱할 수도 있습니다.

다음 기본 인터페이스를 살펴봅시다.

```java
package com.example;

 @Indexed
 public interface AdminService { ... }
```

이제 이 AdminService를 다음과 같이 구현할 수 있습니다.

```java
package com.example.foo;

import com.example.AdminService;

public class ConfigurationAdminService implements AdminService { ... }
```

이 클래스는 인덱싱된 인터페이스를 구현하므로 com.example.AdminService stereotype에 자동으로 포함됩니다.

계층에 @Indexed 인터페이스 및 슈퍼클래스가 더 많으면 클래스는 모든 stereotypes에 매핑됩니다.

### **2. Component**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
    String value() default "";
}
```

Component 어노테이션이 달린 클래스가 "Component"임을 나타냅니다.

이러한 클래스는 annotation-based configuration 및 classpath scanning을 사용할 때 자동 탐지의 후보로 간주됩니다.

다른 클래스 레벨 어노테이션(ex @Repository, @Aspect)도 같은 component를 식별하는 것으로 간주될 수 있습니다.

### **3. Controller**

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {

  /**
   * The value may indicate a suggestion for a logical component name,
   * to be turned into a Spring bean in case of an autodetected component.
   * @return the suggested component name, if any (or empty String otherwise)
   */
  @AliasFor(annotation = Component.class)
  String value() default "";

}
```

Controller 어노테이션이 달린 클래스가 "Controller"임을 나타냅니다.

Controller 어노테이션은 @Component의 specialization 역할을 하며, classpath scanning을 통해 구현 클래스를 자동으로 탐지할 수 있습니다.

일반적으로 RequestMapping 어노테이션을 기반으로 어노테이션이 달린 핸들러 메서드와 함께 사용됩니다.

예시로, InboundApi class를 가져왔습니다.

```java
@RestController
@RequiredArgsConstructor
@RequestMapping(value = "/inbounds")
public class InboundApi {
    // ...
}
```

### **4. Service**

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 */
  @AliasFor(annotation = Component.class)
  String value() default "";

}
```

Service 어노테이션이 달린 클래스가 "Service"임을 나타냅니다.

원래 도메인 중심 설계(Evans, 2003)에 의해 "캡슐화된 상태 없이 모델에서 독립적으로 제공되는 인터페이스로 제공되는 작업"으로 정의됩니다.

클래스가 "Business Service Facade" 또는 이와 유사한 것임을 나타낼 수도 있습니다.

Service 어노테이션은 일반적인 목적의 stereotype이며 팀마다 의미를 좁히고 적절하게 사용할 수 있습니다.

Service 어노테이션은 @Component의 specialization 역할을 하며, classpath scanning을 통해 구현 클래스를 자동으로 탐지할 수 있습니다.

### **5. Repository**

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {

  /**
   * The value may indicate a suggestion for a logical component name,
   * to be turned into a Spring bean in case of an autodetected component.
   * @return the suggested component name, if any (or empty String otherwise)
   */
  @AliasFor(annotation = Component.class)
  String value() default "";

}
```

Repository 어노테이션이 달린 클래스가 "Repository"임을 나타냅니다.

원래 도메인 중심 설계(Evans, 2003)에 의해 "객체 모음을 에뮬레이트하는 저장, 검색 및 검색 동작을 캡슐화하는 메커니즘"으로 정의됩니다.

"Data Access Object(DAO)"와 같은 전통적인 자바 EE 패턴을 구현하는 팀들은 DAO 클래스에 이러한 stereotype를 적용할 수 있지만, 그렇게 하기 전에 DAO와 DDD 스타일 저장소 간의 차이를 이해하도록 주의를 기울여야 합니다.

Repository 어노테이션은 일반적인 목적의 stereotype이며 팀마다 의미를 좁히고 적절하게 사용할 수 있습니다.

따라서 @Repository가 달린 클래스는 PersistenceExceptionTranslationPostProcessor와 함께 사용될 때 Spring Data Access Exception 변환에 적합합니다.

@Repository가 달린 클래스는 tooling, aspects 등을 목적으로 하는 전체 애플리케이션 아키텍처에서의 역할에 대해서도 명확히 설명됩니다.

스프링 2.5에서 Repository 어노테이션은 @Component의 specialization 역할을 하며, classpath scanning을 통해 구현 클래스를 자동으로 탐지할 수 있습니다.

## **stereotype annotation들을 어떻게 찾을까?**

스프링 부트가 시작되는 클래스에 붙어있는 SpringBootApplication 어노테이션을 조사해보았습니다.

여러 어노테이션이 있으나 @EnableAutoConfiguration와 @ComponentScan을 살펴보겠습니다.

### **1. EnableAutoConfiguration**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

  /**
   * Environment property that can be used to override when auto-configuration is
   * enabled.
   */
  String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

  /**
   * Exclude specific auto-configuration classes such that they will never be applied.
   * @return the classes to exclude
   */
  Class<?>[] exclude() default {};

  /**
   * Exclude specific auto-configuration class names such that they will never be
   * applied.
   * @return the class names to exclude
   * @since 1.3.0
   */
  String[] excludeName() default {};

}
```

스프링 애플리케이션 컨텍스트의 auto-configuration을 사용하도록 설정하여 필요할 것 같은 bean을 추측하고 구성합니다.

auto-configuration 클래스는 일반적으로 classpath와 정의한 bean을 기준으로 적용됩니다.

예를 들어, classpath에 tomcat-embedded.jar가 있는 경우, TomcatServletWebServerFactory을 원할 수 있습니다. (자신의 ServletWebServerFactory bean을 정의하지 않았음에도)

@SpringBootApplication을 사용할 때 컨텍스트의 auto-configuration이 자동으로 활성화되므로 이 주석을 추가해도 아무런 효과가 없습니다.

auto-configuration은 가능한 한 지능적이게 노력하며 사용자 자신의 configuration을 더 많이 정의할수록 후순위로 밀려납니다.

적용하지 않을 구성은 exclude()로 제외할 수 있습니다.

또한 spring.autoconfigure.exclude property를 통해 제외할 수 있습니다.

auto-configuration는 항상 사용자 정의 bean을 등록한 후에 적용됩니다.

일반적으로 @SpringBootApplication을 통해 @EnableAutoConfiguration이 선언된 클래스의 패키지는 특정 의미를 가지며 종종 '기본값'으로 사용됩니다.

예를 들어 @Entity 클래스를 검색할 때 사용됩니다.

일반적으로 모든 하위 패키지 및 클래스를 검색할 수 있도록 루트 패키지에 @EnableAutoConfiguration(@SpringBootApplication을 사용하지 않는 경우)를 배치하는 것이 좋습니다.

auto-configuration 클래스는 일반적으로 Spring의 @Configuration bean입니다.

ImportCandidates 및 SpringFactoriesLoader 메커니즘을 사용하여 위치를 파악합니다.

> SpringFactoriesLoader는 META-INF/spring.factories 파일을 사용합니다.

일반적으로 auto-configuration 빈은 @Conditional이 붙은 Bean입니다.

(대부분 @ConditionalOnClass 및 @ConditionalOnMissingBean 주석을 사용합니다).

### **2. ComponentScan**

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {

  @AliasFor("basePackages")
  String[] value() default {};

  @AliasFor("value")
  String[] basePackages() default {};

  Class<?>[] basePackageClasses() default {};

  Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

  Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;

  ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

  String resourcePattern() default ClassPathScanningCandidateComponentProvider.DEFAULT_RESOURCE_PATTERN;

  boolean useDefaultFilters() default true;

  Filter[] includeFilters() default {};

  Filter[] excludeFilters() default {};

  boolean lazyInit() default false;

  @Retention(RetentionPolicy.RUNTIME)
  @Target({})
  @interface Filter {

    FilterType type() default FilterType.ANNOTATION;

    @AliasFor("classes")
    Class<?>[] value() default {};

    @AliasFor("value")
    Class<?>[] classes() default {};

    String[] pattern() default {};
  }
}
```

@Configuration을 사용하는 클래스에서 사용할 component scanning 지시문을 구성합니다.

Spring XML의 <context:component-scan> 요소를 병렬로 지원합니다.

검색할 특정 패키지를 정의하기 위해 basePackageClasses() 또는 basePackages() 또는 해당 별칭 값()을 지정할 수 있습니다.

특정 패키지가 정의되지 않은 경우 ComponentScan 어노테이션을 선언하는 클래스의 패키지에서 검색이 수행됩니다.

<context:component-scan> 요소에는 annotation-config 속성이 있지만, ComponentScan 주석에는 없습니다.

이는 @ComponentScan을 사용할 때 거의 모든 경우에 기본 주석 구성 처리(예: @Autowired)가 가정되기 때문입니다.

또한 AnnotationConfigApplicationContext를 사용할 때 annotation config 프로세서는 항상 등록되므로 @ComponentScan 수준에서 사용하지 않도록 설정하려는 시도는 무시됩니다.

> @Configuration을 사용하는 클래스를 스캔한다고 하니 @Configuration도 살펴보겠습니다.

### **3. Configuration**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {

  @AliasFor(annotation = Component.class)
  String value() default "";

  boolean proxyBeanMethods() default true;

}
```

클래스가 하나 이상의 @Bean 메서드를 선언하고 런타임에 해당 bean에 대한 bean 정의 및 서비스 요청을 생성하기 위해 스프링 컨테이너에 의해 처리될 수 있음을 나타냅니다.

```java
@Configuration
 public class AppConfig {

     @Bean
     public MyBean myBean() {
         // instantiate, configure and return bean ...
     }
 }
```

@Configuration 클래스는 일반적으로 AnnotationConfigApplicationContext 혹은 AnnotationConfigWebApplicationContext를 사용하여 부트스트랩됩니다.

AnnotationConfigApplicationContext의 간단한 예는 다음과 같습니다.

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
 ctx.register(AppConfig.class);
 ctx.refresh();
 MyBean myBean = ctx.getBean(MyBean.class);
 // use myBean ...
```

AnnotationConfigApplicationContext에 직접 @Configuration 클래스를 등록하는 대신 @Configuration 클래스를 Spring XML 파일 내에서 일반 <bean> 정의로 선언할 수 있습니다.

```xml
<beans>
    <context:annotation-config/>
    <bean class="com.acme.AppConfig"/>
 </beans>
```

위의 예에서 ConfigurationClassPostProcessor 및 @Configuration 클래스를 쉽게 처리할 수 있는 다른 주석 관련 포스트 프로세서를 활성화하려면 <context:annotation-config/>가 필요합니다.

@Configuration은 @Component에 meta-annotated되므로 @Configuration 클래스는 component scanning의 후보입니다.

따라서 일반 @Component처럼 @Autowired/@Inject를 활용할 수도 있습니다.

특히, 단일 생성자가 존재하는 경우 자동으로 해당 생성자에 대해 적용될 것입니다.

```java
@Configuration
 public class AppConfig {

     private final SomeBean someBean;

     public AppConfig(SomeBean someBean) {
         this.someBean = someBean;
     }

     // @Bean definition using "SomeBean"

 }
```

@Configuration 클래스는 component scanning을 사용하여 부트스트랩할 수 있을 뿐만 아니라 @ComponentScan 주석을 사용하여 구성 요소 검색을 구성할 수도 있습니다.

```java
@Configuration
 @ComponentScan("com.acme.app.services")
 public class AppConfig {
     // various @Bean definitions ...
 }
```

## **결론**

@SpringBootApplication에 있는 @ComponentScan에서 @Configuration을 사용하는 클래스에서 사용할 component scanning 지시문을 구성합니다.

@Configuration은 @Component에 meta-annotated되므로 @Component 클래스 또한 component scanning의 후보입니다.

위에서 살펴봤듯이 Indexed 제외한 stereotype 어노테이션이 붙은 어노테이션들은 @Component가 붙어있습니다.

따라서, stereotype 어노테이션이 bean으로 등록될 수 있었던 겁니다.
