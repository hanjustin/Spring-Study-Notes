
# Study Resources

- [ ] [Spring Start Here](https://www.manning.com/books/spring-start-here) - Reading Ch. 7
- [ ] [Spring Security in Action](https://www.manning.com/books/spring-security-in-action)
- [ ] [Spring in Action](https://www.manning.com/books/spring-in-action-sixth-edition)

# Table of Contents
* Spring Boot
* Core Technologies
    * **[IoC container](#ioc-container)**
        * **Types**
            * [BeanFactory](#beanfactory), [ApplicationContext](#applicationcontext)
        * **Configuration**
            * [XML-based](#xml-based), `@Configuration`
    * **[Beans](#beans)**
        * **Registering**
            * Method level - [`@Bean`](#bean)
            * Class level (Stereotype) - [`@Component`](#component), `@Controller`, `@Service`, `@Repository`
        * **Dependency**
            * [Wiring](#wiring)
            * [Auto-wiring w/ `@Autowired`](#auto-wiring-w-autowired)
                * [Field](#field-injection), [Setter](#setter-injection), [Constructor](#constructor-injection) injection
        * **[Preference](#preference)**
            * [Selection order](#selection-order)
                * `@Qualifier(ID)`, `@Primary`
        * **Scope**
            * [Singleton](#singleton-bean-default), [Prototype](#prototype-bean-scopeprototype)
    * **[AOP (Aspect Oriented Programming)](#aop-aspect-oriented-programming)**
        * [`@Aspect`](#aspect)
        * **Concepts**
            * [Aspect](#aspect-1), [Advice](#advice), [Pointcut](#pointcut), [Join point](#join-point), [Weaving](#weaving), [Proxy object](#proxy-object)
* Spring Security
* Tools
    * **[Maven](#maven)**
        * [Wrapper](#wrapper), [Build lifecycles](#build-lifecycles), [Directory layout](#directory-layout)

# Spring Boot
* Opinionated framework on top of Spring framework with default configurations for a quick backend development.
* Use annotations instead of XML for configuration.

# Core Technologies

## IoC container
* Use dependency injection to manage life cycle of objects known to the container.

### Types
#### BeanFactory
* The most basic container.
* Load beans on-demand. Only supports singleton & prototype bean scopes.

#### ApplicationContext
* Extends the features of `BeanFactory`
* Load all beans at startup. Supports all types of bean scopes.

### Configuration

#### XML-based

```xml
<!-- Creating two beans. IDs are myBean & anotherBean -->

<beans xmlns="..." xmlns:xsi="..." xsi:schemaLocation="...">

    <bean id="myBean" class="com.example.MyClass">
        <property name="propertyName" value="Hello World!">
        <property name="anotherObj" ref="anotherBean">
    </bean>

    <bean id="anotherBean" class="com.example.AnotherClass">
</beans>
```

```java
public class MyClass {
    private String propertyName;
    private AnotherClass anotherObj;

    // Omitted getter & setter
}

public class AnotherClass {

}
```

#### `@Configuration`
* Used to indicate a source of `@Bean` methods.

## Beans
Objects managed by the Spring framework.

### Registering

#### `@Bean`
* Can be even used to a class not defined in my project.

```java
@Configuration
public class MyConfig {

    //Method name used as the default bean name.
    @Bean(name = "customName")
    public MyClass myClass() {
        return new MyClass();
    }
}
```

#### Stereotype annotations
##### `@Component`
* Use `@ComponentScan` to scan for `@Component`
* Use `@PostConstruct` to manage the instance after creation

```java
@Component
public class MyClass {

    @PostConstruct
    public void doSomething() {

    }
}

@Configuration
@ComponentScan
public class MyConfig {

}
```

### Dependency

#### Wiring
* Accessing a bean by directly calling `@Bean` annotated method
* Receiving a bean by defining a parameter to the method creating the relationship 

```java
/*
 * Only one Car instance is created in the context.
 * Spring uses the same instance even with multiple calls to car().
 * Similar to SwiftUI's single source of truth.
 */

@Configuration
public class MyConfig {

    @Bean
    public Car car() {
        return new car();
    }

    @Bean
    public Driver driver1() {
        Driver me = new Driver();
        me.setCar(car()); // Directly calling car()
        return me;
    }

    @Bean
    public Driver driver2(Car c) { // Spring injects car here
        Driver me = new Driver();
        me.setCar(c);
        return me;
    }
}
```

#### Auto-wiring w/ `@Autowired`

##### Field injection
* Quick & easy, but not recommended.
* Hides the dependencies of the class.
* Lack of immutability. A final field can't be used with `@Autowired`.
* Harder to test

```java
@Component
public class Driver {

    @Autowired
    private Car car;
}
```

##### Setter injection

```java
@Component
public class Driver {

    private Car car;

    @Autowired
    public void setCar(Car car) {
        this.car = car;
    }
}
```

##### Constructor injection
* If only one constructor, `@Autowired` can be omitted.

```java
@Component
public class Driver {

    private final Car car;  // Can use final

    public Driver(Car car) {
        this.car = car;
    }

    // Omitted getter
}
```

### Preference
* When multiple beans of the same type exist, Spring needs to know which one to choose and inject.

#### Selection order
1. `@Qualifier`
2. Bean with `@Primary`
    * When both `@Qualifier` and `@Primary` are present, then the `@Qualifier` annotation will have precedence. Basically, `@Primary` defines a default, while `@Qualifier` is very specific.
3. Parameter, property, or class name gets used as Bean's name.
4. Error if none of the above were used to specify a bean.

```java
/*
 * Multiple beans for Car class & Candy class.
 * driver1 uses parameter name to choose a car & @Primary to choose a candy.
 * driver2 uses @Qualifier to choose a car.
 */
@Configuration
public class MyConfig {

    // Inject smallCar & largeCandy
    @Bean
    public Driver driver1(Car smallCar, Candy largeChocolate) {
        Driver me = new Driver();
        me.setCar(smallCar);
        return me;
    }

    // Inject largeCar
    @Bean
    public Driver driver2(@Qualifier("largeCar") Car c) { 
        Driver me = new Driver();
        me.setCar(c);
        return me;
    }

    @Bean
    public Car smallCar() {
        return new Car("smallCar");
    }

    @Bean
    public Car largeCar() {
        return new Car("largeCar");
    }

    @Bean
    @Primary
    public Candy largeCandy() {
        return new Candy();
    }

    @Bean
    public Candy smallCandy() {
        return new Candy();
    }
}
```

### Scope

#### Singleton bean (Default)
* Can have multiple instances of the same type and always get the same instance when using ID of a specific bean. Not to be confused with singleton design pattern.
* For stateless beans. Recommend immutability to avoid a race conditions.
* Instantiation approaches:
    * **Eager (Default)**
        * Instantiate all singleton beans when initializing the context.
    * **Lazy (Use `@Lazy`)**
        * Instantiate bean when it gets used the first time.

#### Prototype bean `@Scope("prototype")`
* A different instance every time the same bean name requested from the container.
* Useful for stateful beans.

## AOP (Aspect Oriented Programming)
* Separating business logic code from repetitious cross-cutting concerns code, such as logging, security, or transaction management, for increasesd code modularity.
* Method calls are intercepted and altered using aspects to provide additional functionalities.

### `@Aspect`

```java
@Aspect
@Component
public class MyLoggingAspect {

    @Around("execution(* services.*.*(..))")
    public void log(ProceedingJoinPoint joinPoint) {
        joinPoint.proceed();
    }
}

// Omitted MyConfig class with @EnableAspectJAutoProxy
```

### Concepts
#### Aspect
* **`What`** to execute when you call specific methods.

#### Advice
* **`When`** the aspect logic gets executed. (e.g., before or after the method call).
* **Types**
    * `@Before` - Run before the method execution.
    * `@After` - Run after the method execution, regardless of its outcome.
    * `@AfterReturning` - Run after the method execution, only if the method completes successfully.
    * *`@AfterThrowing`* - Run if the method throws exception.
    * `@Around` - Fully customize behaviors before & after the method execution. The original method execution could even be skipped if `joinPoint.proceed()` is not called. Avoid using `@Around` if other types are sufficient for your requirements.

#### Pointcut
* **`Which`** methods will execute the aspect logic. Methods matching pointcut expression/pattern will execute the aspect logic. (Similar concept to regular expression)
* Empty method can be used to create a pointcut signature as a shorthand for advice annotations.
* **Designators**
    * `execution` - To pattern match method execution.
    * `within` - To pattern match method in certain namespace (i.e. package or class).
    * `@within` - To pattern match type of annotations on the namespace.
    * `this` - To pattern match type of the bean reference. Used for CGLIB-based proxy.
    * `target` - To pattern match type of the target object. Used for JDK-based proxy.
    * `@target` - To pattern match type of annotations on the target object.
    * `args` - To pattern match type of method arguments.
    * `@args` - To pattern match annotations of method arguments.
    * `@annotation` - To pattern match annotations of methods.
    * `bean` - To pattern match id of bean.

#### Join point
* **`Trigger`** of the aspect logic. In Spring, always a method call.
* `ProceedingJoinPoint` parameter of the aspect method represents the intercepted method, and this can be used to get any information related to the intercepted method

```java
public void methodInAspect(ProceedingJoinPoint joinPoint) {
    String methodName = joinPoint.getSignature().getName();
    Object[] arguments = joinPoint.getArgs();
}
```

#### Weaving
Linking the aspect logic & the original method dynamically at runtime to use the interface of the original method.

#### Proxy object 
Substitute object that intercepts the original method execution to use the aspect logic. Spring uses CGLIB or JDK dynamic proxy implementation.

# Tools

## Maven
It is a project management and build automation tool. Maven projects are configured using a Project Object Model (POM) in a `pom.xml` file. The file contains information such as dependencies, source directory, plugin, etc.

### Wrapper
Utility script to run the Maven project without having Maven installed. Ensures that the correct Maven version is used, providing consistency across different development environments.

```
./mvnw clean install
```

### Build lifecycles
* `clean` - Remove all files generated by the previous build
* `compile` - Turns source files into class files
* `test` - Runs unit tests
* `package` - Creates an artifact such as a JAR, ZIP or WAR file

### Directory layout
* `src/main/java` - Application/Library sources
* `src/main/resources` - Application/Library resources
* `src/test/java` - Test sources
* `target` - Contains all the final products from the build