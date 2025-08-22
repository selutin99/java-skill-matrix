# Проект Spring Cloud, какие в нем есть компоненты и как их использовать

## 1. Что это такое

**Spring Cloud** — набор проектов (надстройка над Spring Boot), которые решают **типовые задачи микросервисной архитектуры**:

* регистрация сервисов,
* балансировка нагрузки,
* конфигурация,
* отказоустойчивость,
* взаимодействие через сообщения,
* распределённый трейсинг.

Spring Cloud не привязан к одной технологии — он оборачивает популярные решения (Eureka, Consul, Zookeeper, Netflix OSS, Kubernetes API и др.) в привычный для Spring Boot интерфейс.

---

## 2. Основные компоненты Spring Cloud

### 2.1 Spring Cloud Config

* Централизованное хранилище конфигурации (Git, SVN, Vault, файловая система).
* Приложение поднимается, тянет конфиг с сервера и может обновляться «на лету».
* Использование:

    * `spring-cloud-config-server` (поднимаем сервер конфигурации),
    * `spring-cloud-starter-config` (клиенты).

---

### 2.2 Service Discovery

* Регистрация и обнаружение сервисов.
* Поддерживаются:

    * **Netflix Eureka** (`spring-cloud-starter-netflix-eureka-server`),
    * Consul, Zookeeper, Kubernetes Service Discovery.
* Клиенты регистрируются на сервере и находят друг друга по имени.

Пример:

```java
@EnableEurekaClient
@SpringBootApplication
public class UserServiceApp {}
```

---

### 2.3 Load Balancing (Spring Cloud LoadBalancer)

* Ранее использовался **Ribbon** (Netflix), сейчас **Spring Cloud LoadBalancer**.
* Автоматически распределяет запросы между инстансами сервиса.
* Пример:

```java
@LoadBalanced
@Bean
RestTemplate restTemplate() {
    return new RestTemplate();
}
```

Теперь вызов `http://user-service/users` распределяется по всем инстансам `user-service`.

---

### 2.4 API Gateway

* **Spring Cloud Gateway** — современный API-шлюз (замена Zuul).
* Задачи: маршрутизация, фильтры (аутентификация, логирование, метрики, CORS).

Пример конфигурации (YAML):

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user_route
          uri: lb://user-service
          predicates:
            - Path=/users/**
          filters:
            - AddRequestHeader=X-Request-Source, gateway
```

---

### 2.5 Circuit Breaker / Resilience

* Для отказоустойчивости.
* Netflix Hystrix (deprecated), сейчас используют **Resilience4j** (`spring-cloud-starter-circuitbreaker-resilience4j`).
* Позволяет:

    * ограничить время ожидания (timeout),
    * fallback при сбое,
    * bulkhead (изоляция потоков),
    * retry.

Пример:

```java
@CircuitBreaker(name = "userService", fallbackMethod = "fallback")
public String getUser() {
    return restTemplate.getForObject("http://user-service/users/1", String.class);
}
```

---

### 2.6 Messaging (Spring Cloud Stream)

* Унифицированная абстракция для сообщений (Kafka, RabbitMQ, Pulsar).
* Пишем код на уровне биндингов → под капотом драйвер очереди.

Пример:

```java
@EnableBinding(Sink.class)
public class MessageConsumer {
    @StreamListener(Sink.INPUT)
    public void handle(String message) {
        System.out.println("Received: " + message);
    }
}
```

*(в новых версиях — через функциональный стиль `Supplier`/`Consumer`).*

---

### 2.7 Distributed Tracing (Spring Cloud Sleuth + Zipkin/Jaeger)

* Добавляет **traceId** и **spanId** во все запросы/логи.
* Поддерживает Zipkin/Jaeger/Brave.
* Автоматически встраивает ID в HTTP-заголовки (`X-B3-TraceId`, `X-B3-SpanId`).

---

### 2.8 Spring Cloud Bus

* Распространяет события через брокеры сообщений (Kafka, RabbitMQ).
* Пример: обновление конфигурации (refresh) на всех сервисах через шину.

---

## 3. Как использовать Spring Cloud

1. **Подключить стартер**:

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```
2. **Аннотации и конфигурация**:

    * `@EnableEurekaServer`, `@EnableEurekaClient`, `@EnableConfigServer`.
3. **YAML настройки**:

    * указываем discovery server, config server, gateway правила.
4. **Интеграция с Kubernetes**:

    * вместо Eureka можно использовать `spring-cloud-starter-kubernetes`.

---

## 4. Выжимка для собеседования

* **Spring Cloud** — набор инструментов для микросервисов.
* Основные модули:

    * **Config** — централизованная конфигурация,
    * **Eureka/Consul/Zookeeper** — service discovery,
    * **LoadBalancer** — балансировка вызовов между сервисами,
    * **Gateway** — API-шлюз, маршрутизация и фильтры,
    * **Circuit Breaker (Resilience4j)** — отказоустойчивость,
    * **Stream** — унифицированный доступ к брокерам сообщений,
    * **Sleuth** — распределённый трейсинг,
    * **Bus** — события и refresh конфигурации.
* Большинство компонентов Spring Cloud — это **обёртки над популярными решениями (Netflix OSS, Consul, Kubernetes, Kafka и др.)** с единым API в стиле Spring Boot.
