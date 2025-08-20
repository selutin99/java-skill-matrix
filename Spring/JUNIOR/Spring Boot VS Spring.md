# Отличия Spring Boot от Spring

## 1. Spring Framework

* Это **ядро экосистемы Spring**.
* Предоставляет фундаментальные возможности:

    * **IoC / DI** (контейнер бинов),
    * **AOP**,
    * **Data Access (JDBC, JPA)**,
    * **Spring MVC**,
    * **Security**,
    * поддержка интеграции с другими технологиями.
* Требует много **ручной конфигурации**: XML, Java-конфигурация, настройка серверов, зависимостей.
* Гибкий, но **шумный в настройке**.

---

## 2. Spring Boot

* Построен **поверх Spring Framework**.
* Его цель — **сократить конфигурацию** и **ускорить разработку**.
* Основные возможности:

    1. **Автоконфигурация**

        * Spring Boot анализирует classpath и автоматически настраивает beans.
        * Например, наличие `spring-boot-starter-data-jpa` → создаёт `EntityManager`, `DataSource`.
    2. **Starters**

        * Готовые зависимости для популярных стеков:

            * `spring-boot-starter-web`,
            * `spring-boot-starter-data-jpa`,
            * `spring-boot-starter-security`.
    3. **Встроенные сервлет-контейнеры**

        * Tomcat, Jetty, Undertow "из коробки".
        * Не нужно отдельно деплоить на сервер приложений — можно запускать как **обычный Java JAR**.
    4. **Actuator**

        * Метрики, health-checks, мониторинг.
    5. **Externalized Configuration**

        * Лёгкое управление конфигурацией через `application.properties` или `application.yml`.

---

## 3. Сравнение

| Характеристика               | Spring Framework                | Spring Boot                                            |
| ---------------------------- | ------------------------------- | ------------------------------------------------------ |
| **Назначение**               | Фундамент, ядро экосистемы      | Упрощает использование Spring                          |
| **Конфигурация**             | Ручная (XML/Java config)        | Автоконфигурация                                       |
| **Запуск приложения**        | Требуется сервер (Tomcat и др.) | Встроенный Tomcat/Jetty/Undertow                       |
| **Подключение зависимостей** | Нужно самому подбирать версии   | Starters (группы зависимостей с совместимыми версиями) |
| **Мониторинг**               | Отдельно интегрировать          | Actuator "из коробки"                                  |
| **Фокус**                    | Гибкость, расширяемость         | Быстрый старт и удобство                               |

---

## 4. Пример

### Spring (без Boot)

```java
@Configuration
@EnableWebMvc
@ComponentScan("com.example")
public class AppConfig { }

public class WebAppInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext container) {
        AnnotationConfigWebApplicationContext context =
                new AnnotationConfigWebApplicationContext();
        context.register(AppConfig.class);
        container.addListener(new ContextLoaderListener(context));
        container.addServlet("dispatcher", new DispatcherServlet(context))
                 .addMapping("/");
    }
}
```

### Spring Boot

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

---

## 5. Выжимка для собеседования

* **Spring = фреймворк**, фундамент (IoC, AOP, MVC, Security и др.).
* **Spring Boot = надстройка над Spring**, упрощающая конфигурацию и запуск.
* Boot даёт:

    * **автоконфигурацию**,
    * **стартеры**,
    * **встроенные сервера**,
    * **Actuator**,
    * **упрощённую конфигурацию через properties/yml**.
* Boot = быстрый старт и удобство, Spring = гибкость и фундамент.
