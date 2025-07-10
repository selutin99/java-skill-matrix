# Компоненты Spring WebFlux

## 🔧 Что такое Spring WebFlux

Это **реактивный веб-фреймворк**, основанный на:

* **Project Reactor (`Mono`, `Flux`)**
* **Netty/WebServer**, не использует Servlet API

Позволяет строить **неблокирующие, асинхронные HTTP-сервисы**, особенно эффективен под нагрузкой.

---

## ⚛️ Ключевые компоненты

### 1. `Mono` и `Flux` (из Project Reactor)

* `Mono<T>` — 0 или 1 элемент
  Пример: `Mono.just(User(...))`
* `Flux<T>` — 0…N элементов
  Пример: `Flux.fromIterable(list)`

> Используются как **типы возвращаемых значений** в контроллерах и сервисах.

---

### 2. Роутинг и хендлеры (функциональный стиль)

Альтернатива `@Controller`.

#### 🔸 Определение маршрута:

```kotlin
@Configuration
class RouterConfig {
  @Bean
  fun route(userHandler: UserHandler): RouterFunction<ServerResponse> =
    RouterFunctions.route()
      .GET("/users/{id}", userHandler::getUser)
      .POST("/users", userHandler::createUser)
      .build()
}
```

#### 🔹 Обработка запроса:

```kotlin
@Component
class UserHandler(private val userService: UserService) {
  fun getUser(request: ServerRequest): Mono<ServerResponse> {
    val id = request.pathVariable("id")
    return userService.findUser(id)
      .flatMap { user -> ServerResponse.ok().bodyValue(user) }
      .switchIfEmpty(ServerResponse.notFound().build())
  }
}
```

> Можно использовать `ServerRequest`, `ServerResponse`, `bodyToMono()`, `bodyValue()`.

---

### 3. `WebClient` — реактивный HTTP-клиент

Альтернатива `RestTemplate`, **неблокирующий**.

```kotlin
val client = WebClient.create("https://api.example.com")

client.get()
  .uri("/users/{id}", id)
  .retrieve()
  .bodyToMono(User::class.java)
```

Поддерживает:

* `retrieve()` — простая модель,
* `exchangeToMono()` — полное управление ответом.

---

### 4. Реактивные репозитории (Spring Data R2DBC / Mongo)

#### Пример интерфейса:

```kotlin
interface UserRepository : ReactiveCrudRepository<User, String>
```

Поддержка:

* `findById(id): Mono<User>`
* `findAll(): Flux<User>`
* `save(user): Mono<User>`
* `deleteById(id): Mono<Void>`

Работает с MongoDB, Postgres (через R2DBC), Cassandra и др.

---

## 💡 Общий жизненный цикл:

```
Client → Router → Handler → Service → Repository
                      ↓
                 Mono/Flux
                      ↓
                ServerResponse
```

---

## 🔍 Когда использовать WebFlux:

✅ Нужна **высокая масштабируемость**
✅ Сервис делает много **внешних вызовов**
✅ Ваша база и HTTP-клиенты тоже **реактивные**

❌ Не нужно, если всё синхронное или простое CRUD API.
