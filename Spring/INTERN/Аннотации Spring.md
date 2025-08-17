# Основные стереотипы (аннотации), используемые в Spring

В Spring **стереотипы** — это аннотации, которыми помечают классы для автоматического обнаружения и регистрации в контейнере IoC. Они показывают *роль* компонента в приложении. Все они по сути разновидности `@Component`.

## **Базовая аннотация**

* **`@Component`**

    * Универсальная аннотация, указывает, что класс — Spring-компонент, создаваемый и управляемый контейнером.
    * Используется, если класс не подпадает под более конкретную категорию.

### **Специализированные стереотипы**

1. **`@Service`**

    * Для бизнес-логики.
    * Семантически показывает, что класс выполняет *сервисные* операции.
    * Дополнительно может участвовать в обработке транзакций.

2. **`@Repository`**

    * Для DAO и работы с базой данных.
    * Помимо регистрации как бина, включает обработку исключений (перевод SQLException в DataAccessException).

3. **`@Controller`**

    * Для контроллеров в веб-приложениях Spring MVC.
    * Маршрутизирует HTTP-запросы и возвращает *View* или *ModelAndView*.

4. **`@RestController`**

    * То же, что `@Controller`, но с автоматическим добавлением `@ResponseBody`.
    * Используется для REST API: методы возвращают данные напрямую (JSON, XML).

*Все они разновидности `@Component` и регистрируются в контексте через component-scan.*

---

## **Подробно: аннотации @Configuration, @Bean, @Autowired, @Qualifier**

Эти аннотации относятся к настройке бинов и внедрению зависимостей в Spring.

---

### **1. `@Configuration`**

* Помечает класс как источник конфигурации Spring.
* Такой класс сам является бином, а его методы с `@Bean` создают другие бины.
* Работает как **Java-based конфигурация** (альтернатива XML).
* Пример:

  ```java
  @Configuration
  public class AppConfig {
      @Bean
      public MyService myService() {
          return new MyServiceImpl();
      }
  }
  ```

---

### **2. `@Bean`**

* Помечает метод, который возвращает бин для контекста Spring.
* Используется внутри классов с `@Configuration` (или других бинов).
* Пример:

  ```java
  @Bean
  public DataSource dataSource() {
      return new HikariDataSource();
  }
  ```

---

### **3. `@Autowired`**

* Автоматическое внедрение зависимостей (DI) по типу.
* Spring находит подходящий бин и подставляет его в конструктор, сеттер или поле.
* Наилучший стиль — **через конструктор**, чтобы зависимости были обязательными.
* Пример:

  ```java
  @Service
  public class OrderService {
      private final PaymentService paymentService;

      @Autowired
      public OrderService(PaymentService paymentService) {
          this.paymentService = paymentService;
      }
  }
  ```

---

### **4. `@Qualifier`**

* Используется вместе с `@Autowired`, если несколько бинов одного типа.
* Уточняет, какой именно бин внедрять.
* Пример:

  ```java
  @Service
  public class OrderService {
      private final PaymentService paymentService;

      @Autowired
      public OrderService(@Qualifier("paypalPaymentService") PaymentService paymentService) {
          this.paymentService = paymentService;
      }
  }
  ```

---

## **Выжимка для собеседования (короткий ответ)**

* **`@Component`** — общий Spring-компонент.
* **`@Service`** — бизнес-логика.
* **`@Repository`** — работа с базой, перевод SQL-исключений.
* **`@Controller`** — MVC-контроллер.
* **`@RestController`** — REST API (возвращает JSON).
* **`@Configuration`** — класс-конфигурация, источник бинов.
* **`@Bean`** — метод создаёт бин вручную.
* **`@Autowired`** — внедрение зависимости по типу (лучше через конструктор).
* **`@Qualifier`** — уточняет, какой бин внедрять, если их несколько.
