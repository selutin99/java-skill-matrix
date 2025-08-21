# Как тестировать Spring приложение

## 1. Пирамида и цели

* **Unit**: быстрые тесты без Spring‑контекста (логика, мапперы, сервисы с замоканными зависимостями).
* **Slice‑тесты** (*конфигурация «кусочка» приложения*): `@WebMvcTest`, `@DataJpaTest`, `@JdbcTest`, `@JsonTest`, `@WebFluxTest` — поднимают **минимальный** контекст.
* **Интеграционные**: `@SpringBootTest` (часто с Testcontainers), проверяют склейку модулей, конфигурацию, миграции, безопасность, messaging.
* **E2E/контракты**: живые окружения, WireMock, Spring Cloud Contract.

---

## 2. Базовый стек

* **JUnit 5** (Jupiter), **AssertJ/Hamcrest**, **Mockito / MockK (Kotlin)**.
* **spring-boot-starter-test**: включает JUnit 5, AssertJ, Mockito, Spring Test, JSONassert и т.д.
* **Testcontainers**: реальные сервисы в Docker (PostgreSQL, Kafka, Redis, MinIO…).
* **WireMock / MockWebServer**: стаб внешних HTTP.
* **spring-kafka-test**, **EmbeddedKafkaBroker** — для Kafka.
* **REST‑клиент**: `MockMvc` (MVC), `WebTestClient` (WebFlux), для внешних вызовов — `MockRestServiceServer` (RestTemplate/RestClient).

---

## 3. Unit‑тесты (без Spring)

* Максимально дешёвые и быстрые.
* Мокаем зависимости:

  ```java
  class PriceServiceTest {
    @Test
    void calc() {
      var repo = mock(PriceRepo.class);
      when(repo.find("A")).thenReturn(10);
      var s = new PriceService(repo);
      assertThat(s.calcTotal("A", 2)).isEqualTo(20);
    }
  }
  ```
* Для Kotlin: MockK + kotest/junit.

---

## 4. Slice‑тесты: поднимаем только нужный слой

### 4.1 `@WebMvcTest`

* Тестируем контроллеры, фильтры, `@ControllerAdvice`, валидацию.
* Автоконфигурирует `MockMvc`, но **не поднимает сервисы/репозитории** → используем `@MockBean`.

  ```java
  @WebMvcTest(UserController.class)
  class UserControllerTest {
    @Autowired MockMvc mvc;
    @MockBean UserService userService;

    @Test void getById() throws Exception {
      when(userService.get(1L)).thenReturn(new User(1L,"A"));
      mvc.perform(get("/users/1"))
         .andExpect(status().isOk())
         .andExpect(jsonPath("$.name").value("A"));
    }
  }
  ```

### 4.2 `@DataJpaTest` / `@JdbcTest`

* Поднимает JPA/Jdbc + `TestEntityManager`, откатывает транзакции после теста.
* По умолчанию H2; лучше **Testcontainers PostgreSQL**:

  ```java
  @DataJpaTest
  @Testcontainers
  class UserRepoTest {
    @Container static PostgreSQLContainer<?> pg = new PostgreSQLContainer<>("postgres:16");
    @DynamicPropertySource
    static void props(DynamicPropertyRegistry r){
      r.add("spring.datasource.url", pg::getJdbcUrl);
      r.add("spring.datasource.username", pg::getUsername);
      r.add("spring.datasource.password", pg::getPassword);
    }
    @Autowired UserRepository repo;
    @Test void saveAndFind() { /* ... */ }
  }
  ```

### 4.3 `@JsonTest`

* Тестируем (де)сериализацию Jackson (настройки модулей, `@JsonComponent` и т.п.).

### 4.4 `@WebFluxTest`

* Контроллеры на WebFlux + `WebTestClient`.

---

## 5. Интеграционные: `@SpringBootTest`

* Полный контекст: конфигурация, бины, security, миграции, messaging.
* Варианты:

    * `webEnvironment = RANDOM_PORT` — поднять HTTP и тестировать через `TestRestTemplate`/`WebTestClient`.
    * `@AutoConfigureMockMvc` — без реального сервера, но со всей цепочкой Spring MVC.
* Переопределение конфигурации:

    * `@ActiveProfiles("test")`
    * `@TestPropertySource(properties = "key=value")`
    * `@DynamicPropertySource` (для Testcontainers).
* Работа с БД:

    * миграции **Flyway/Liquibase** должны прогоняться в тестах;
    * не полагайтесь на H2, если в проде PostgreSQL — используйте Testcontainers.
* Очистка состояния:

    * `@Sql`/`@SqlGroup` для подготовки/очистки данных;
    * транзакционные тесты откатывают состояние после каждого теста.

---

## 6. Тестирование безопасности (Spring Security)

* MVC: `SecurityMockMvcRequestPostProcessors`

  ```java
  mvc.perform(get("/admin").with(user("alice").roles("ADMIN")))
     .andExpect(status().isOk());
  ```
* Аннотации: `@WithMockUser`, `@WithUserDetails`.
* WebFlux: `WebTestClient` с `.mutateWith(mockOAuth2Login())` и т.п.

---

## 7. Тестирование реактивного кода

* Контроллеры: `@WebFluxTest` + `WebTestClient`.
* Сервисы/Reactor: `StepVerifier`:

  ```java
  StepVerifier.create(service.findAll())
              .expectNextCount(3)
              .verifyComplete();
  ```
* Транзакции: использовать **R2DBC** + reactive transaction manager в тестах.

---

## 8. Внешние системы

* **HTTP‑зависимости**: `WireMock`/`MockWebServer`, либо `MockRestServiceServer` для RestTemplate/RestClient.
* **Kafka/Rabbit**: `spring-kafka-test` + `EmbeddedKafkaBroker` или Testcontainers Kafka/RabbitMQ.
* **S3/MinIO**: Testcontainers MinIO.
* **Redis/Elasticsearch/Mongo**: соответствующие контейнеры.

---

## 9. Быстрота и стабильность тестов

* **Контекст кэшируется** в Spring Test — избегайте `@DirtiesContext` без нужды.
* Делайте тесты **идемпотентными** (каждый сам готовит свои данные).
* Минимизируйте использование `@SpringBootTest` там, где достаточно slice.
* Параллельный запуск: изолируйте ресурсы (порты, БД), используйте уникальные имена/схемы.
* Явно задавайте **Clock**/`TimeProvider` (внедрение `Clock`) для детерминизма по времени.

---

## 10. Частые паттерны/ошибки

* **Self‑invocation & AOP**: в интеграционных тестах проверяйте, что аннотации (`@Transactional`, `@Cacheable`, `@Async`) реально работают через прокси.
* **`@MockBean` злоупотребление**: slice — ок; в интеграционных лучше реальные бины.
* **Валидаторы/Advice**: отдельно тестируйте `@ControllerAdvice`, `@Valid` (и сообщения локализации).
* **Отсутствие миграций** в тестовом профиле → тесты «зелёные», прод падает. Всегда гоняйте миграции.
* **Случайные порты** и конфликты: `RANDOM_PORT` + инжект клиента по порту.

---

## 11. Примеры мини‑рецептов

**MockMvc с валидацией и глобальным обработчиком**

```java
@WebMvcTest(controllers = OrderController.class)
@Import(GlobalExceptionHandler.class)
class OrderControllerValidationTest {
  @Autowired MockMvc mvc;
  @MockBean OrderService service;

  @Test void badRequestOnInvalidPayload() throws Exception {
    mvc.perform(post("/orders")
        .contentType(MediaType.APPLICATION_JSON)
        .content("{\"quantity\":0}"))
      .andExpect(status().isBadRequest())
      .andExpect(jsonPath("$.errors[0].field").value("quantity"));
  }
}
```

**Mock внешнего HTTP для RestClient**

```java
@SpringBootTest
@AutoConfigureWebClient // если используете RestClient autoconfig
class ExternalClientTest {
  private WireMockServer wm = new WireMockServer(0);

  @DynamicPropertySource
  static void props(DynamicPropertyRegistry r){ /* baseUrl = wm.baseUrl() */ }

  @BeforeEach void up(){ wm.start(); }
  @AfterEach void down(){ wm.stop(); }

  @Test void callsExternal() {
    wm.stubFor(get(urlEqualTo("/api/users/1"))
       .willReturn(aResponse().withBody("{\"id\":1,\"name\":\"Bob\"}")));
    var user = client.getUser(1);
    assertThat(user.getName()).isEqualTo("Bob");
  }
}
```

**Kotlin: сервис с MockK**

```kotlin
class PriceServiceTest {
  private val repo = mockk<PriceRepo>()
  private val service = PriceService(repo)

  @Test
  fun calc() {
    every { repo.find("A") } returns 10
    expectThat(service.calcTotal("A", 2)).isEqualTo(20)
    verify { repo.find("A") }
  }
}
```

---

## 12. Инструменты качества

* **Coverage**: Jacoco (на уровне модульных и slice‑тестов).
* **Static analysis**: SpotBugs/PMD/Checkstyle/Detekt (Kotlin).
* **Контрактные тесты**: Spring Cloud Contract для потребитель‑поставщик сценариев.

---

## Выжимка для собеседования

* **Стратегия**: пирамида — много unit, умеренно slice, немного `@SpringBootTest`.
* **Slice‑тесты**: `@WebMvcTest` (контроллеры + `MockMvc`), `@DataJpaTest` (репозитории + транзакции + Testcontainers), `@WebFluxTest` (контроллеры + `WebTestClient`).
* **Интеграция**: `@SpringBootTest` с `RANDOM_PORT`/`@AutoConfigureMockMvc`, миграции через Flyway/Liquibase, реальные сервисы через **Testcontainers**.
* **Внешние зависимости**: HTTP — WireMock/MockRestServiceServer; Kafka — `spring-kafka-test`/Testcontainers.
* **Security‑тесты**: `@WithMockUser`, `user("...").roles("...")`.
* **Reactor**: проверяем с `StepVerifier`.
* **Производительность**: избегать `@DirtiesContext`, переиспользовать контекст, минимизировать область конфигурации.
* **Аннотации‑прокси** (`@Transactional`, `@Cacheable`, `@Async`) реально работают только через прокси — интеграционным тестом проверяем поведение, а не реализацию.
