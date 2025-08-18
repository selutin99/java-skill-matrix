# Как сконфигурировать и запустить Spring приложение с нуля

## **1. Создание проекта**

Есть два пути:

* **Чистый Spring (ручная конфигурация)** — подходит для учебных примеров или старых проектов.
* **Spring Boot (рекомендуется)** — быстрый старт без сложной настройки.

### Через Spring Boot

1. Перейти на [https://start.spring.io](https://start.spring.io)
2. Выбрать:

    * **Project:** Maven или Gradle
    * **Language:** Java
    * **Spring Boot version** (обычно latest stable)
    * **Dependencies:** Spring Web, Spring Data JPA, Lombok, H2/PostgreSQL и т.д.
3. Сгенерировать проект и распаковать.

---

## **2. Конфигурация приложения**

### **Вариант A: Через аннотации (Spring Boot)**

* Основной класс приложения:

  ```java
  @SpringBootApplication
  public class DemoApplication {
      public static void main(String[] args) {
          SpringApplication.run(DemoApplication.class, args);
      }
  }
  ```

  `@SpringBootApplication` включает:

    * `@Configuration` — класс-конфигурация
    * `@EnableAutoConfiguration` — автоподбор настроек
    * `@ComponentScan` — поиск бинов в текущем пакете

* Настройки пишутся в `application.properties` или `application.yml`:

  ```properties
  server.port=8081
  spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
  spring.datasource.username=postgres
  spring.datasource.password=pass
  ```

### **Вариант B: Чистый Spring (без Boot)**

1. Создать конфигурационный класс:

   ```java
   @Configuration
   @ComponentScan(basePackages = "com.example")
   public class AppConfig {
   }
   ```
2. В Main-классе поднять контекст:

   ```java
   public class Main {
       public static void main(String[] args) {
           AnnotationConfigApplicationContext context =
               new AnnotationConfigApplicationContext(AppConfig.class);
           
           MyService service = context.getBean(MyService.class);
           service.run();
           context.close();
       }
   }
   ```
3. Зависимости указываются вручную (в pom.xml / build.gradle), нет автоконфигурации.

---

## **3. Добавление логики**

* Создать бин:

  ```java
  @Service
  public class MyService {
      public void run() {
          System.out.println("Hello from Spring!");
      }
  }
  ```

* При старте Spring автоматически создаст бин и внедрит зависимости.

---

## **4. Запуск приложения**

* Через IDE (`main()` метод).
* Через Maven/Gradle:

    * Maven: `mvn spring-boot:run`
    * Gradle: `./gradlew bootRun`
* Через jar-файл:

  ```bash
  mvn clean package
  java -jar target/demo-0.0.1-SNAPSHOT.jar
  ```

---

## **Выжимка для собеседования (короткий ответ)**

* **Spring Boot:** создаём проект через [start.spring.io](https://start.spring.io), добавляем зависимости, пишем основной класс с `@SpringBootApplication`, запускаем через `SpringApplication.run()`.
* **Чистый Spring:** создаём конфигурацию с `@Configuration` и `@ComponentScan`, поднимаем контекст вручную через `AnnotationConfigApplicationContext`.
* **Зависимости и порты** задаются через `application.properties` / `application.yml`.
* **Бины создаются аннотациями `@Component`, `@Service`, `@Repository`, `@Controller`.**
