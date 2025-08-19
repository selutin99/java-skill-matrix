# Spring Web MVC и Spring Flux

## 1. Введение

Оба модуля — часть экосистемы Spring для построения веб-приложений.

* **Spring Web MVC** — классическая модель, синхронная, на основе Servlet API.
* **Spring WebFlux** — реактивная модель, асинхронная, построена на Project Reactor и поддерживает non-blocking I/O.

---

## 2. Spring Web MVC

### Основные принципы

* Реализация шаблона **Model-View-Controller (MVC)**.
* Базируется на **Servlet API** и **Servlet containers** (Tomcat, Jetty и т.д.).
* Каждый запрос обрабатывается в **отдельном потоке** (thread-per-request).

### Архитектура

1. **DispatcherServlet** — главный фронт-контроллер (все запросы идут через него).
2. **HandlerMapping** — находит контроллер для запроса.
3. **Controller** — обрабатывает бизнес-логику.
4. **ModelAndView** — возвращает данные и view.
5. **ViewResolver** — выбирает представление (JSP, Thymeleaf и др.).

### Пример контроллера

```java
@Controller
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{id}")
    public String getUser(@PathVariable Long id, Model model) {
        User user = userService.findById(id);
        model.addAttribute("user", user);
        return "userView"; // JSP/Thymeleaf template
    }
}
```

---

## 3. Spring WebFlux

### Основные принципы

* Реактивный программный модель (на основе **Reactive Streams**).
* Использует **Project Reactor** (типовые объекты: `Mono<T>` и `Flux<T>`).
* Асинхронная обработка запросов (event-loop вместо thread-per-request).
* Может работать как на **Netty** (по умолчанию), так и на сервлетах (Tomcat).

### Архитектура

1. **DispatcherHandler** — аналог `DispatcherServlet`, но реактивный.
2. **HandlerMapping** — находит обработчик.
3. **HandlerAdapter** — связывает обработчик и запрос.
4. **Reactive Controller** — возвращает реактивные типы.

### Пример контроллера (аннотации)

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{id}")
    public Mono<User> getUser(@PathVariable Long id) {
        return userService.findById(id); // возвращает Mono<User>
    }

    @GetMapping
    public Flux<User> getAllUsers() {
        return userService.findAll(); // возвращает Flux<User>
    }
}
```

### Пример функционального стиля

```java
@Configuration
public class UserRouter {

    @Bean
    public RouterFunction<ServerResponse> route(UserHandler handler) {
        return RouterFunctions.route(GET("/users/{id}"), handler::getUser)
                              .andRoute(GET("/users"), handler::getAllUsers);
    }
}
```

---

## 4. Сравнение Spring MVC vs WebFlux

| Характеристика         | Spring MVC (Servlet)                         | Spring WebFlux (Reactive)                            |
| ---------------------- | -------------------------------------------- | ---------------------------------------------------- |
| **Модель**             | Блокирующая (thread-per-request)             | Неблокирующая (event-loop)                           |
| **Контейнер**          | Servlet API (Tomcat, Jetty)                  | Netty, Undertow, Servlet API                         |
| **API**                | `ModelAndView`, объекты                      | `Mono<T>`, `Flux<T>`                                 |
| **Подходит для**       | CRUD, монолитов, классических веб-приложений | High-load, микросервисов, стриминга                  |
| **Сложность**          | Ниже                                         | Выше (нужно понимать реактивное программирование)    |
| **Производительность** | Хорошая для большинства приложений           | Лучше при большом числе соединений (IO-bound задачи) |

---

## 5. Подводные камни

### Spring MVC

* Поток на каждый запрос → плохо масштабируется при очень высокой нагрузке.
* Но проще для понимания, большинство библиотек поддерживают.

### Spring WebFlux

* Сложнее отладка и дебаг.
* Требует реактивных драйверов (например, `R2DBC` вместо JDBC).
* Если использовать блокирующий код внутри реактивного контроллера → теряется весь смысл.

---

## 6. Выжимка для собеседования

* **Spring MVC** — синхронная модель, работает на Servlet API, поток на каждый запрос. Подходит для обычных веб-приложений.
* **Spring WebFlux** — реактивная, неблокирующая модель, основана на Project Reactor. Использует `Mono` и `Flux`. Подходит для high-load и IO-bound систем.
* **DispatcherServlet vs DispatcherHandler** — фронт-контроллеры MVC и WebFlux.
* **Mono = 0/1 элемент**, **Flux = 0..N элементов**.
* MVC проще, WebFlux даёт выигрыш при большом числе одновременных соединений.
