
# [Unorganized Notes](#unorganized-notes-1)

---

# Study Resources

- [x] [Spring Start Here](https://www.manning.com/books/spring-start-here) - (Done)
- [ ] [Spring Security in Action](https://www.manning.com/books/spring-security-in-action) - Reading Ch. 4 of 18
- [ ] [Spring in Action](https://www.manning.com/books/spring-in-action-sixth-edition)

# Table of Contents
* Spring Boot
* Core Technologies
    * **[IoC container](#ioc-container)**
        * **Types:** [BeanFactory](#beanfactory), [ApplicationContext](#applicationcontext)
        * **Configuration:** [XML-based](#xml-based), `@Configuration`
    * **[Beans](#beans)**
        * **Registering**
            * **Method level:** [`@Bean`](#bean)
            * **Class level (Stereotype):** [`@Component`](#component), `@Controller`, `@Service`, `@Repository`
        * **Dependency**
            * [Wiring](#wiring)
            * Auto-wiring w/ `@Autowired`: [Field](#field-injection), [Setter](#setter-injection), [Constructor](#constructor-injection) injection
        * **[Preference](#preference)**
            * **[Selection order:](#selection-order)** `@Qualifier(ID)`, `@Primary`
        * **Scope**
            * [Singleton](#singleton-bean-default), [Prototype](#prototype-bean-scopeprototype)
    * **[AOP (Aspect Oriented Programming)](#aop-aspect-oriented-programming)**
        * [`@Aspect`](#aspect)
        * **Concepts:** [Aspect](#aspect-1), [Advice](#advice), [Pointcut](#pointcut), [Join point](#join-point), [Weaving](#weaving), [Proxy object](#proxy-object)
* Spring Security
* Tools
    * **[Maven](#maven)**
        * `pom.xml`, [Wrapper](#wrapper), [Build lifecycles](#build-lifecycles), [Directory layout](#directory-layout)
    * **cURL**
        * `-u`

# Spring Boot
* Opinionated framework on top of Spring framework with default configurations for a quick backend development.
* Use annotations instead of XML for configuration.

## Auto-configuration
* To reduce the need for manual configuration, analyze classpath for auto-configurations. Use `application.properties` or `application.yml` to customize auto-configurations.
    * **Aspect:** No need to add `@EnableAspectJAutoProxy`.
    * **Embedded Web Servers:** Use Tomcat.

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
* Multiple aspects can be added to a method. `@Order(#)` can specify aspect's order. Aspect with lower number given will execute first.

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
    * `execution` - To pattern match method execution. Has format:
    
    ```
    execution(modifiers-pattern?
              return-type-pattern
              declaring-type-pattern?
              name-pattern(param-pattern)
              throws-pattern?)
    
    // Modifiers, declaring-type, and throws patterns are optional
    ```
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
* `ProceedingJoinPoint` parameter of the aspect method represents the intercepted method, and this can be used to get any information related to the intercepted method.

```java
public void methodInAspect(ProceedingJoinPoint joinPoint) {
    String methodName = joinPoint.getSignature().getName();
    Object[] arguments = joinPoint.getArgs();
    joinPoint.proceed(); // Execute the next aspect or the intercepted method
}
```

#### Weaving
Linking the aspect logic & the original method dynamically at runtime to use the interface of the original method.

#### Proxy object 
Substitute object that intercepts the original method execution to use the aspect logic. Spring uses CGLIB or JDK dynamic proxy implementation.

# Tools

## Maven
It is a project management and build automation tool. Maven projects are configured using a Project Object Model (POM) in a `pom.xml` file. The file contains information such as dependencies, source directory, plugin, etc.

### `pom.xml`

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

## cURL

### -u
For server authentication, provide the username & password.

```
curl -u user:secret https://example.com
```

---

# Unorganized Notes

Added section to accelerate my learning pace by postponing knowledge organization effort to reduce my inefficient usage of time. To go fast and then revisit & re-organize learned concepts later.

Listing of concepts I only know on the surface level that I don't even know where to organize in my head.


* [**Misc notes**](#misc-notes)
* **Book notes by chapter**

<table>
    <tr>
        <th>Ch.</th>
        <th><a href="#spring-start-here">Spring<br>Start Here</a></th>
        <th><a href="#spring-security-in-action">Spring<br>Security in Action</a></th>
        <th>Spring<br>in Action</th>
    </tr>
    <tr>
        <td>1</td>
        <td>x</td>
        <td><a href="#1-security-today"><b>Security today</b></a></td>
        <td></td>
    </tr>
    <tr>
        <td>2</td>
        <td>x</td>
        <td><a href="#2-high-level"><b>High level</b></a></td>
        <td></td>
    </tr>
    <tr>
        <td>3</td>
        <td>x</td>
        <td><a href="#3-managing-users"><b>Managing users</b></a></td>
        <td></td>
    </tr>
    <tr>
        <td>4</td>
        <td>x</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>5</td>
        <td>x</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>6</td>
        <td>x</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>7</td>
        <td><a href="#7-spring-mvc"><b>MVC</b></a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>8</td>
        <td><a href="#8-spring-boot--spring-mvc"><b>Boot & MVC</b></a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>9</td>
        <td><a href="#9-web-scopes"><b>Web scopes</b></a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>10</td>
        <td><a href="#10-rest-services"><b>REST services</b></a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>11</td>
        <td><a href="#11-rest-client"><b>REST client</b></a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>12</td>
        <td><a href="#12-data-sources"><b>Data sources</b></a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>13</td>
        <td><a href="#13-transactions"><b>Transactions</b></a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>14</td>
        <td><a href="#14-data-persistence-implementation"><b>Spring Data</b></a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>15</td>
        <td><a href="#15-testing"><b>Testing</b></a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>16</td>
        <td>x</td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>17</td>
        <td>x</td>
        <td></td>
        <td></td>
    </tr>
</table>

* **Youtube notes**

## Misc Notes

* **User** --(interact)--> **Client** --(establish)--> **TCP Connection** --(with)--> **Servlet container** --(translate)--> **HTTP Request** --(sent)--> **Dispatcher servlet** --(use)--> **Handler Mapping** --(find)--> **Controller** --(delegate)--> **Service** --(utilize)--> **Repository** --(use)--> **Data sourcer** --(manage)--> **JDBC driver** --(connect)--> **DBMS** --(update)--> **DB**

## Spring Start Here
### 7 Spring MVC
##### 7.1.3 Tomcat
* **Servlet container:** HTTP request/response translator for Java apps.
* **Servlet:** Java object in the container registered to a specific path for handling HTTP request.

#### 7.2 Spring Boot

Without Spring Boot, Spring web app needed a lot of configurations for a servlet container & servlet instances.

* **Dependency starters:** Grouping dependencies for a specific purpose.
* **Autoconfiguration based on dependencies:** Based on the dependencies, use default configurations.

##### 7.2.1 Initial Spring Boot from Spring.io
* Main class w/ `@SpringBootApplication`
* Spring Boot parent node in `pom.xml`. Set compatible versions for dependencies.
* Spring Boot Maven plugin <build> <plugins> ... </plugins></build> tags in `pom.xml`
* `application.properties`

##### 7.2.2 Dependency starter
* Capability-oriented groups of compatible dependencies. Group of dependencies you add to configure your app for a specific purpose. Add a particular capability needed and Spring Boot will add add the right dependencies.

##### 7.2.3 Autoconfiguration by convention
* convention-over-configuration principle. Based on the dependencies, default configurations used.

#### 7.3 Spring MVC

* `@Controller`: component of the web app that contains methods executed for a specific HTTP request
* `@RequestMapping`: specifying the path. method needs to return the name of the document to send as a response.
* Tomcat uses a servlet component known as the dispatcher servlet (front controller) as the entry point of the Spring web app.
* Dispatcher servlet uses the handler mapping to find a controller action with `@RequestMapping` for the HTTP request. Then, the dispatcher servlet uses View resolver to get the view content.

### 8 Spring Boot & Spring MVC
#### 8.1 Dynamic view using template engines
* Template engine: Dependency to display variable data the controller sends. Thymeleaf template engine used in this section.
* `resources/templates` folder used for dynamic view. ${key} in html to get value of the attribute from `Model`. `@RequestMapping` method uses `Model` type as method parameter and returned string as view name

```java
@RequestMapping("/home")
public String home(Model page) {
    page.addAttribute("key", value);
    return "home.html";
}
```

##### 8.1.2 Request parameters to send data
* For small or optional data. Often used for search & filter.
* Use `@RequestParam` to get brand & honda `http://example.com/products?name=honda&color=blue`

```java
@RequestMapping("/home")
public String home(
      @RequestParam(required = false) String name,
      @RequestParam(required = false) String color,
      Model page) {
    page.addAttribute("key", value);
    return "home.html";
}
```

##### 8.1.3 Path variables to send data

```java
@RequestMapping("/home/{color}")
  public String home(
      @PathVariable String color,
      Model page) {
    return "home.html";
}
```

#### 8.2 HTTP GET and POST methods
* Singleton beans aren’t thread-safe
* `@RequestMapping` uses GET by default. Instead, can use `@GetMapping`, `@PostMapping`
```java
@RequestMapping(path = "/products",
                method = RequestMethod.POST)
```

```java
/*
Can use the model class instead of @RequestParam
if request parameters’ names are the same as
the model class attributes’ names
*/

public class Product {
  private String name;
  private double price;
  // Omitted getters and setters
}

@PostMapping("/products")
public String addProduct(
    @RequestParam String name,
    @RequestParam double price,
    Model model
) {

}

// Shorthand form
@PostMapping("/products")
public String addProduct(
    Product p,
    Model model
) {

}
```

### 9 Web scopes
* Web apps have more bean scopes.
    * Request scope: Creates an instance of the bean class for every HTTP request. The instance exists only for that specific HTTP request.
    * Session scope: The instance is in the server’s memory for the full HTTP session. Spring links the instance in the context with the client’s session.
    * Application scope: The instance is unique in the app’s context while the app is running.

#### 9.1 Request scope
* New bean created for every request, so a lot of instances could get created. Not a big problem as long as the instances are short-lived by not having a time consuming logic.
* Not prone to multithread related issues.
* Put `@RequestScope` above class declaration.

#### 9.2 Session scope
* Each HTTP session will have different instances of session-scoped bean. During the same session, multiple requests from the same client will use the shared data throughout the session. `@SessionScope`
* Concurrency issues possible from multiple concurrent requests from the same client changing the data.
* Depending on the logic, requests could become dependent one on the other because of the session's statefulness.
* `"redirect:/"` can be used to redirect to the main page.

#### 9.3 Application scope
* All different users share an application-scoped bean. Close to how a singleton works. `@ApplicationScope`

### 10 REST services

#### 10.1 REST services for data
* The view resolver no longer used by the MVC dispatcher servlet.

#### 10.2 Implementing a REST endpoint
* Method level: `@ResponseBody`. Class level: `@RestController`. The annotations tell the dispatcher servlet that methods are returning HTTP response data and not a view name.

#### 10.3 Managing the HTTP response
##### 10.3.2 Setting the response status and headers
* Use `ResponseEntity` class to customize HTTP response.

```java
@GetMapping("/france")
public ResponseEntity<Country> france() {
Country c = Country.of("France", 67);
return ResponseEntity
        .status(HttpStatus.ACCEPTED)
        .header("continent", "Europe")
        .body(c);
}
```

##### 10.3.3 Using aspect for exceptions from REST controller

* Create an aspect using REST controller advice to intercept exceptions to separate the exception logic from controllers’ methods.

```java
public class ErrorDetails {
  private String message;
}

@RestControllerAdvice
public class ExceptionControllerAdvice {

    @ExceptionHandler(NotEnoughMoneyException.class)
    public ResponseEntity<ErrorDetails> exceptionNotEnoughMoneyHandler() {
        ErrorDetails errorDetails = new ErrorDetails();
        return ResponseEntity
                .badRequest()
                .body(errorDetails);
    }
}
```

#### 10.4 Getting data from a request body
* Use `@RequestBody` in method param.

```java
@PostMapping("/payment")
public ResponseEntity<PaymentDetails> makePayment(
    @RequestBody MyModel bodyDetails) {

    return ResponseEntity
            .status(HttpStatus.ACCEPTED)
            .body(bodyDetails);
}
```

### 11 REST Client
* **OpenFeign:** Recommended Spring Cloud tool. Use WebClient if using a reactive approach.
* **RestTemplate:** For a new app, avoid RestTemplate and use OpenFeign instead for less boilerplate code.
* **WebClient:** A reactive solution for calling REST endpoints as an alternative to RestTemplate.

#### 11.1 Spring Cloud OpenFeign
* Add `@EnableFeignClients`.
* Add `@FeignClient` to an interface with methods for REST requests and the bean gets implemented. The context will inject the bean to where it's needed.
* `application.properties` can be used to supply the base URL. The property `spring.cloud.openfeign.client.config.<interface-name>.url` is used for this. The <interface-name> is the value of the name attribute in the `@FeignClient`. With this practice, no need to recompile the code when running the app in different environments.

```java
@FeignClient(name = "payments",
             url = "${name.service.url}")
public interface PaymentsProxy {
    @PostMapping("/payment")
    Payment createPayment(
        @RequestHeader String requestId,
        @RequestBody Payment payment);
}
```

#### 11.2 RestTemplate
* Use `HttpEntity` to build request data headers & body.

```java
HttpHeaders headers = new HttpHeaders();
HttpEntity<MyModel> httpEntity =
  new HttpEntity<>(requestDataObj, headers);

RestTemplate rest = new RestTemplate();
ResponseEntity<MyModel> response =
    rest.exchange("URL",
        HttpMethod.POST,
        httpEntity,
        MyModel.class);

MyModel responseBody = response.getBody();
```

#### 11.3 WebClient
* Using pub/sub model, `Mono` class is used to create task dependencies.
* `@Value("${name.service.url}")` to get the base URL from the properties file.

### 12 Data sources
#### 12.1 Data source
* Component to manage database connections. Uses the JDBC (Java Database Connectivity) driver to connect to the DBMS. Improve performance by making new connections only when necessary and managing to reuse connections.
* JDK provides JDBC abstractions. The abstraction is implemented by a JDBC driver connect to a specific DBMS. i.e. MySQL JDBC driver
* HikariCP (Hikari connection pool) data source is commonly used.

```java
// Make new connection every time
Connection con = DriverManager.getConnection(dbURL, username, password);
```

#### 12.2 JdbcTemplate
* H2 database = in-memory database
* `schemas.sql` in resources folder to define db structure. This file will contain structural SQL queries or data description language (DDL). Used for simple projects.
* Boot adds `JdbcTemplate` bean automatically when seeing H2 dependency.

```java
@Repository
public class MyOrderRepo {

    public void storeOrder(Order order) {
        String sql = "INSERT INTO orders VALUES (NULL, ?, ?)";
        jdbc.update(sql, order.getProduct(), order.getPrice());
    }

    public List<Order> findAllOrders() {
        String sql = "SELECT * FROM order";

        RowMapper<Order> orderRowMapper = (r, i) -> {
            Order rowOrder = new Order();
            rowOrder.setId(r.getInt("id"));
            rowOrder.setProduct(r.getString("product"));
            rowOrder.setPrice(r.getBigDecimal("price"));
            return rowOrder;
        };

        return jdbc.query(sql, orderRowMapper);
    }
}
```

#### 12.3 Data source configuration
##### 12.3.1 application.properties
```
// For MySQL
spring.datasource.url=url
spring.datasource.username=<dbms username>
spring.datasource.password=<dbms password>

// Instruct Boot to run the “schema.sql” file
spring.datasource.initialization-mode=always
```

##### 12.3.2 Custom dataSource bean
* Boot adds default `DataSource` bean, but a data source with custom implementation can be added to the context.
* Can create multiple data source objects, each with their own JdbcTemplate object associated to use multiple databases. Will need to use `@Qualifier`.

### 13 Transactions
#### 13.1 Transactions
* Atomicity: Execute a set of multiple operations changing data altogether or not at all.
* Commit: Succesful end of a transaction.
* Rollback: Restoring data to the way it looked before the transaction.

#### 13.2 AOP aspect for transactions
* AOP aspect lies behind the scenes of a transaction.
* `@Transactional` rolls back the transaction if the annotated method throws an exception.

```java
// simplistic representation of the transaction aspect logic
try {
    // Start transaction
    // Call intercepted method
    // Commit transaction
} catch (RuntimeException e) {
    // Rollback transaction
}
```

#### 13.3 Using transactions
* Mutable operations should be in a method with `@Transactional`
* `@Transactional` can be applied on a method and class level. Method level's config overrides the one on the class.

```java
var rowObj = jdbc.queryForObject(selectSQL, new MyRowMapper(), id);

public class MyRowMapper implements RowMapper<MyData> {

    @Override
    public MyData mapRow(ResultSet resultSet, int i)
        throws SQLException {
        MyData o = new MyData();
        o.setSomeProperty(resultSet.getString("someProperty"));
        return o;
    }
}
```

### 14 Data persistence implementation
#### 14.1 Spring Data
* Abstraction layer over persistence technologies.

#### 14.2 Inside Spring Data
* `Repository` is a marker interface. Rarely extended directly.
* `CrudRepository` extends `Repository` for CRUD functionalities.
* `PagingAndSortingRepository` extends `CrudRepository` with paging & sorting.
* Independent sub-modules exist to support different persistence technologies. Module specific interfaces exist for added functionalities.

#### 14.3 Spring Data JDBC
* `@Id` to mark property as a primary key.
* Method naming convention to create SQL query behind the scenes. Not recommended for complex queries.
* `@Query` to specify a query and use `:paramName` to use parameter. Need `@Modifying` for modifying operation methods.
```java
public interface MyRepo extends CrudRepository<MyDataType, PrimaryKeyType>
```

```java
List<Account> findAccountsByName(String name)
// translates to:
SELECT * FROM account WHERE name = ?
```

### 15 Testing
##### 15.2.1 Unit tests
* `@DisplayName` to put test description.
* `mock()` to create a mock, which is a method from Mockito. Control the mock’s behavior using `given()`.
* `@Mock` for a mock to get inject to a property. `@InjectMocks` for object to test to get created with mocks injected.

##### 15.2.1 Integration tests
* `@MockBean` adds the mock to the application context, so it can be used elsewhere with `@Autowired`

## Spring Security in Action
### 1 Security today
* Commonly intercept the requests and that ensure whoever makes the requests has permission to access protected resources
* Through authentication, identify a user. Through authorization, identify the user's permissions.
* A heap dump could find sensitive data in internal memory of an executing app.
* Apply security in layers & use different practices for each layer.

### 2 High level
#### 2.1 Defaults
* Two default authentication mechanisms: HTTP Basic and Form Login. For HTTP Basic authentication, use `Authorization` header and attach prefix `Basic` to the value followed by the Base64 encoding of `<username>:<password>` string. Base64 is only an encoding method for the convenience of the transfer, so doesn’t offer confidentiality of the credentials.
* `@SpringBootApplication` marks package to scan. To scan other packages, use `@ComponentScan`.

#### 2.2 Spring Security Flow
1. `Authentication filter` delegates the request to the authentication manager.
2. `Authentication manager` uses `authentication provider` to process authentication.
3. `Authentication provider` implements the authentication logic by using `UserDetailsService` & `PasswordEncoder`
4. `UserDetailsService` implements user details management responsibility. Authorities are actions that you allow a user to do in the context of the application.
5. `PasswordEncoder` implements password management. It encodes a password (usually with an encryption or a hashing algorithm) and verifies if the password matches an existing encoding
6. `Security context` keeps the authentication data until the action ends. 

#### 2.3 Customization
##### 2.3.1 User details management
* Create methods with `@Bean` to return `UserDetailsService` & `PasswordEncoder`.

##### 2.3.2 Endpoint level authorization customization.
```java
@Configuration
public class ProjectConfig {

    private final CustomAuthenticationProvider authenticationProvider;

    public ProjectConfig(
    CustomAuthenticationProvider authenticationProvider) {
        this.authenticationProvider = authenticationProvider;
    }

    @Bean
    SecurityFilterChain configure(HttpSecurity http)
        throws Exception {

        // Use HTTP Basic auth
        http.httpBasic(Customizer.withDefaults());

        http.authenticationProvider(authenticationProvider);

        // Require authentication for all requests
        http.authorizeHttpRequests(
            c -> c.anyRequest().authenticated()
        );

        var user = User.withUsername("john")
                        .password("12345")
                        .authorities("read")
                        .build();
        
        var userDetailsService =
            new InMemoryUserDetailsManager(user);
        http.userDetailsService(userDetailsService);

        return http.build();
    }
}
```

##### 2.3.4 AuthenticationProvider
```java
@Component
public class CustomAuthenticationProvider
    implements AuthenticationProvider {

    @Override
    public Authentication authenticate(
        Authentication authentication)
        throws AuthenticationException {

            // Custom auth logic
            // Using UserDetailsService and PasswordEncoder
    }
}
```

### 3 Managing users
#### 3.1 Auth interfaces
* `UserDetails`: describe users.
* `GrantedAuthority`: actions that the user can execute.
* `UserDetailsService`: retrieve user by username.
* `UserDetailsManager`: add actions to modify users & passwords to `UserDetailsService`.

#### 3.2 UserDetails
* Only `getUsername()` & `getPassword()` methods from `UserDetails` are used for authentication.
* `GrantedAuthority` represents a privilege granted to the user.
```java
@Entity
public class User {
    @Id
    private int id;
    // Omitted code for username, password & authority properties
}

public class SecurityUser implements UserDetails {
    private final User user;

    public SecurityUser(User user) {
        this.user = user;
    }

    // Omitted code for UserDetails
}
```

#### 3.3 Managing users
##### 3.3.1 UserDetailsService
```java
public class MyService implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String username)
        throws UsernameNotFoundException {
            // Find user from
            // a database, an external system, a vault, etc.
    }
}

public class MyUser implements UserDetails {
}
```

##### 3.3.3 UserDetailsManager
```java
public interface UserDetailsManager extends UserDetailsService {
  void createUser(UserDetails user);
  void updateUser(UserDetails user);
  void deleteUser(String username);
  void changePassword(String oldPassword, String newPassword);
  boolean userExists(String username);
}
```

* `JdbcUserDetailsManager` implementation expects `username`, `password`, and `enabled` columns in the `users` table.
* In the `/resources folder`, put
    * `schema.sql` for queries such as creating, altering, or dropping tables to structure the database
    * `data.sql` for queries such as INSERT, UPDATE, or DELETE to use the data inside the tables