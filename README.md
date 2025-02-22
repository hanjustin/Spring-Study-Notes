
# Table of Contents
* Spring Boot
* Core Technologies
    * IoC
    * **[Beans](#beans)**
        * **[Registering](#registering)**
            * [`@Bean`](#bean), [`@Component`](#component)
        * **[Dependency](#dependency)**
            * [Wiring](#wiring)
            * [Auto-wiring w/ `@Autowired`](#auto-wiring-w-autowired)
                * [Field](#field-injection), [Setter](#setter-injection), [Constructor](#constructor-injection) injection
        * **[Preference](#preference)**
            * `@Qualifier`, `@Primary`
        * **[Scope](#scope)**
            * [Singleton](#singleton-bean-default), [Prototype](#prototype-bean-scopeprototype)
    * Configurations
    * AOP (Aspect Oriented Programming)
* Spring Security
* Tools
    * **[Maven](#maven)**
        * [Wrapper](#wrapper), [Build lifecycles](#build-lifecycles), [Directory layout](#directory-layout)

# Spring Boot
* Opinionated framework on top of Spring framework with default configurations for a quick backend development.
* Use annotations instead of XML for configuration.

# Core Technologies

## IoC
Manage lifecycle of java objects. Spring context `ApplicationContext` represents the IoC container. Objects that are known to the Spring context gets used by the framework the way it was configured.

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

#### `@Component`
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

    **Preference selection order:**
    1. `@Qualifier`
    2. Bean with `@Primary`
    3. Bean with name matching parameter name or property name.
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

#### Prototype bean `@scope("prototype")`
* A different instance every time the same bean name requested from the container.
* Useful for stateful beans.

## Configurations

## AOP (Aspect Oriented Programming)
Separating cross-cutting concerns from the business logic code for increasesd code modularity. Cross-cutting concerns are aspects of a program that affect multiple parts of the application, such as logging, security, or transaction management. These concerns can lead to code duplication and tangled code if not handled properly. This modularization helps keep the business logic clean and uncluttered by separating the additional functionalities into aspects.

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