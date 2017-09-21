title: Mocking Spring Boot beans with Spock
date: 2017/09/21 12:00
authorId: JMT
tags: [java, spring boot, spock, testing]
---

[Spock](https://github.com/spockframework/spock/) is a powerful Groovy-based testing framework that makes testing applications written in Java (as well  as Groovy, Kotlin etc.) a very pleasant experience. Because it's built on top of good ol' JUnit, it integrates well with most of the existing testing/building tooling. However, making it play nicely with the testing infrastructure of Spring framework (especially [Spring Boot](https://projects.spring.io/spring-boot/)) used to be a bit tricky. Luckily, the latest Spock version 1.1 comes with a few improvements that make testing Spring apps (including mocking) easier.

<!-- more -->

## Hand-built mocks

First, let's see how we can replace a Spring component (aka a `@Bean`) with a different implementation during unit tests. For the sake of this article, let's say that we have a service class that provides the today's date. Because we want our tests to be reproducible, we don't want to depend on the actual system clock of the computer where we're running them, but we want to be able to inject the clock. Luckily, the Java 8 date & time API allows us to do just that:

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

For the production code, we use provide a clock implementation that actually uses the system clock:

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

But for the test, we want to use a clock that doesn't tick, or in other words, always returns the same moment in time (in this case 2017-09-20 00:00). To override the default one inside of a test, we define a nested class annotated with `@TestConfiguration` with a factorz method that creates our mock. Note that names of both Spring beans must be the same ("clock" in this case). The bean name is by default given by method name, but can be customised by a parameter of the `@Bean` annotation.

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
