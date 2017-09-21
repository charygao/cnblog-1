title: Mocking Spring Boot beans with Spock
date: 2017/09/21 12:00
authorId: JMT
tags: [java, spring boot, spock, testing]
---

[Spock](https://github.com/spockframework/spock/) is a powerful Groovy-based framework that makes testing applications written in Java (as well  as Groovy, Kotlin etc.) a very pleasant experience. Because it's built on top of good ol' JUnit, it integrates well with most of the existing test/build tooling. However, making it play nicely with the testing infrastructure of Spring framework (especially [Spring Boot](https://projects.spring.io/spring-boot/)) used to be a bit tricky. Luckily, the latest Spock version 1.1 comes with a few improvements that make testing Spring apps (including mocking) easier.

<!-- more -->

## Hand-built mocks

First, let's see how we can replace a Spring component (aka a `@Bean`) with a different implementation during unit tests. For the sake of this article, let's say that we have a service class that provides the today's date. Because we want our tests to be reproducible, we don't want to depend on the actual system clock of the computer where we're running them, but we want to be able to inject the clock from outside. Luckily, the Java 8 date & time API allows us to do just that:

```java
@Service
class TimeService {

    private final Clock clock;

    public TimeService(Clock clock) {
        this.clock = clock;
    }

    public LocalDate today() {
        return LocalDate.now(clock);
    }
}
```

For the production code, we provide a clock implementation that actually uses the system clock:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public Clock clock() {
        return Clock.systemDefaultZone();
    }
}
```

But for the test, we want to use a clock that doesn't tick - or in other words, always returns the same moment in time (in this case 2017-09-20 00:00). To override the default one inside of a test, we define a nested class annotated with `@TestConfiguration` with a factory method that creates our mock. Note that names of both Spring beans must be the same ("clock" in this case). The bean name is by default given by the method name, but can be customised by a parameter of the `@Bean` annotation.

Then we just need to annotate the test class with `@SpringBootTest`. The previous Spock version also required playing with `@ContextConfiguration` annotation, but that's no longer needed. Once we've done that, the Spring context will inject the mock into our service under test. (Of course, we could avoid dealing with the Spring container in this test altogether and instantiate both classes manually, but we want to illustrate the general approach.)

```groovy
@SpringBootTest
class TimeServiceTest extends Specification {

    static NOW = LocalDateTime.of(2017, 9, 20, 0, 0).atZone(systemDefault()).toInstant()

    @Autowired
    TimeService timeService

    def "should return today's date"() {
        expect:
        timeService.today() == LocalDate.of(2017, 9, 20)
    }

    @TestConfiguration
    static class Mocks {
        @Bean
        Clock clock() {
            Clock.fixed(NOW, systemDefault())
        }
    }
}
```


## Spock mocks

Now, say we want to test a REST controller. When using Spring Web MVC, a lot of the functionality depends not only on plain Java code, but also on the metadata hidden inside of the annotations such as `@RequestMapping` (and its siblings) where the routes for different URLs and HTTP methods are defined.
 
 ```java
@RestController
class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "hello, world";
    }
}
```
 
We can easily instantiate this controller inside of a unit test and check that the `hello()` method works correctly, but to verify correctness of the routing, we need an integration test runs the whole MVC infrastructure. With Spring Boot, we only add `@AutoConfigureMockMvc` to the test class and then we can utilize the `MockMvc`, a fake HTTP client that behaves just like a real one, but doesn't actually start the whole web server and directly communicates with the Spring machinery. With that, we can simulate the GET request to `/hello` URL and verify the response status and body.

For the sake of simplicity, the controller always returns a hardcoded string, so we could just assert the response body, but for more complicated cases (for example having a `void` method mapped to a POST request), it's more convenient to generate a mock object using a testing framework, stub only the given method and verify the _interaction_ (that the method was called the given number of times with the given arguments). Spring by default provides a `@MockBean` annotation that creates and injects a Mockito mock, which can do all of the above, but its API is tedious to write and hard to read. Spock leverages the power of Groovy and provides same functionality in a much more convenient syntax.

Again, we can create a mock controller using a `@TestConfiguration`, but it wasn't possible to create Spock mocks this way in the version 1.0, because the `Stub()` & `Mock()` methods were defined only on the `Specification` instance and therefore not accessible from the static nested class. However, the current version provides a `DetachedMockFactory` that can be used instead. 

```groovy
@SpringBootTest
@AutoConfigureMockMvc
class HelloControllerTest extends Specification {

    @Autowired
    MockMvc mvc
    @Autowired
    HelloController mockController

    def 'GET /hello should call hello()'() {
        given:
        1 * mockController.hello() >> 'hello, test'

        when:
        def response = mvc.perform(get('/hello'))
                .andReturn().response

        then:
        response.status == 200
        response.contentAsString == 'hello, test'
    }

    @TestConfiguration
    static class Mocks {
        def factory = new DetachedMockFactory()

        @Bean
        HelloController helloController() {
            factory.Mock(HelloController)
        }
    }
}
```

Now, we can either stub the controller method to return a given value that is expected in the response using the `>>` operator or verify the number of invocations using the multiplication operator. In the example above, we do both at the same time and in order to do achieve that, both operators must be used in the same statement (it's one of the limitations of the framework that can cause some confusion).

## Conclusion
The complete working application can be found on GitHub:

https://github.com/natix643/spring-spock-demo
