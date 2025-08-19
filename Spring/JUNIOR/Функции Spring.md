# Функции Spring, что такое bean и что такое context

## 1. Что такое Spring и зачем он нужен

Spring — это фреймворк для разработки на Java, который решает задачу:

* уменьшить связность кода,
* управлять зависимостями,
* упростить работу с инфраструктурой (БД, веб, безопасность).

Его ядро — **Inversion of Control (IoC) Container**, который управляет объектами приложения.

---

## 2. Основные функции Spring

1. **IoC / DI (Inversion of Control / Dependency Injection)**

    * Контейнер сам создает объекты и внедряет их в зависимости.
    * Убирает ручное создание зависимостей через `new`.

2. **AOP (Aspect-Oriented Programming)**

    * Возможность внедрять функциональность "по периметру": логирование, транзакции, безопасность, кэш.
    * Без изменения основного кода.

3. **Data Access / JDBC Template / ORM**

    * Упрощение работы с БД: транзакции, шаблоны для JDBC, интеграция с Hibernate/JPA.

4. **Spring MVC / WebFlux**

    * Построение веб-приложений (MVC — синхронно, WebFlux — реактивно).

5. **Spring Security**

    * Аутентификация, авторизация, защита API.

6. **Spring Boot**

    * Автоконфигурация, встроенные серверы (Tomcat/Jetty/Undertow), готовые стартеры.
    * Минимум настроек для быстрого старта.

---

## 3. Что такое Bean

* **Bean** — это объект, которым управляет Spring IoC Container.
* Он живет внутри контейнера и создается на основании конфигурации (аннотации, XML, Java-класс с `@Configuration`).

Пример:

```java
@Component
public class UserService { }
```

```java
@Service
public class OrderService {
    private final UserService userService;

    @Autowired
    public OrderService(UserService userService) {
        this.userService = userService;
    }
}
```

Здесь `UserService` и `OrderService` — **beans**. Контейнер сам создает `UserService`, а затем внедряет его в `OrderService`.

---

## 4. Что такое Context (ApplicationContext)

* **ApplicationContext** — это главный интерфейс IoC-контейнера Spring.
* Отвечает за:

    * создание и управление жизненным циклом beans,
    * хранение конфигурации приложения,
    * разрешение зависимостей.

Типы контекста:

* `AnnotationConfigApplicationContext` — для конфигурации через аннотации и `@Configuration`.
* `ClassPathXmlApplicationContext` — для XML-конфигурации.
* `WebApplicationContext` — используется в веб-приложениях.

Пример:

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

UserService userService = context.getBean(UserService.class);
```

---

## 5. Выжимка для собеседования

* **Spring** — это фреймворк, упрощающий создание приложений на Java.
* **Главная функция**: IoC/DI (контейнер управляет зависимостями).
* **Другие ключевые функции**: AOP, работа с БД, MVC/WebFlux, Security, Boot.
* **Bean** — объект, которым управляет Spring-контейнер.
* **Context (ApplicationContext)** — IoC-контейнер, в котором живут beans.
