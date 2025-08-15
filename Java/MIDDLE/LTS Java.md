# Основные нововведения последних LTS версий Java

## **Java 5 (2004) — революция языка**

* **Generics** — параметризованные типы (`List<String>`).
* **Аннотации** — `@Override`, `@Deprecated`, пользовательские аннотации.
* **Enums** — полноценные enum-классы с методами и константами.
* **Varargs** — методы с переменным числом аргументов.
* **Autoboxing/unboxing** — автоматическая конвертация примитивов и обёрток.
* **Concurency utilities (java.util.concurrent)** — `Executor`, `Callable`, `Future`, `CountDownLatch`, `ConcurrentHashMap` и др.

---

## **Java 6 (2006) — стабильность и библиотечные улучшения**

* Компилятор получил **JSR 199/269 API** (инструменты и аннотационные процессоры).
* Улучшения `javax.script` — встроенная поддержка скриптов (JS через Rhino).
* Повышена производительность и поддержка **JDBC 4.0**.
* Улучшения `java.util.concurrent`, `Desktop API`, `JAX-WS` и `JAXB`.
  *(Главное: стабилизация после крупного скачка в Java 5)*

---

## **Java 7 (2011) — Project Coin и ForkJoin**

* **Try-with-resources** — автоматическое закрытие ресурсов (`AutoCloseable`).
* **Diamond operator (`<>`)** — вывод типов при создании дженериков.
* **Strings in switch** — можно использовать строки в `switch`.
* **Underscores in numbers** — `1_000_000` для читаемости.
* **ForkJoinPool / Parallelism** — основа для параллельных стримов в будущем.
* Улучшения `NIO.2` — работа с файловой системой (`Paths`, `Files`, watch service).

---

## **Java 8 (2014) — эпоха Stream API и функционального программирования**

* **Lambda expressions** — компактные анонимные функции.
* **Stream API** — декларативная работа с коллекциями.
* **Functional interfaces** — `@FunctionalInterface`, `Predicate`, `Function`, `Supplier`.
* **Default и static методы в интерфейсах**.
* **Optional** — обёртка вместо `null`.
* **java.time API (JSR-310)** — новая современная дата/время (замена `Date` и `Calendar`).
* **Nashorn JS engine** (позже удалён).
* **PermGen → Metaspace** — изменена модель памяти классов.

---

## **Java 11 (2018) — первый быстрый LTS после 8**

* **var в локальных переменных (Java 10 → LTS)**.
* **HTTP Client API (java.net.http)** — современный клиент (замена HttpURLConnection).
* **Новые методы строк**: `isBlank()`, `lines()`, `repeat()`, `strip()`.
* **Запуск без компиляции**: `java Hello.java` напрямую.
* **Удалены**: JavaFX, JEE/JAX-WS, Nashorn (частично в 15).
  *(Главное: модульная платформа из Java 9, но многие приложения ещё её обходят)*

---

## **Java 17 (2021) — современный стандарт**

* **Sealed classes** — ограничение иерархий (`sealed`, `permits`).
* **Pattern matching for instanceof** — приведение без явного каста.
* **Records** — компактные immutable-POJO (`record User(String name, int age)`).
* **Switch expressions (Java 14 → LTS)** — новый функциональный switch.
* **Text blocks (`"""` )** — многострочные строки.
* **Hidden classes, JEP 411 (удаление SecurityManager), JEP 356 (RandomGenerator API)**.
* **JEP 376 — Vector API (incubator)**.
  *(Фокус: синтаксический сахар, упрощение кода, новые возможности JVM)*

---

## **Java 21 (2023) — максимально насыщенный LTS**

* **Record patterns** — распаковка record прямо в pattern matching.
* **Pattern matching for switch (finalized)**.
* **Virtual threads (Project Loom)** — лёгкие потоки (`Thread.ofVirtual()`), масштабируемость.
* **Sequenced Collections API** — новые интерфейсы для коллекций с упорядоченными элементами.
* **String Templates (preview)** — `"Hello \{name}!"` (ещё в превью).
* **Scoped values (preview)** — альтернатива ThreadLocal для виртуальных потоков.
* **Generational ZGC и улучшения G1/ZGC** — производительный GC без пауз.
  *(Фокус: производительность JVM + Project Loom + улучшение читаемости кода)*

---

## **Краткая шпаргалка (для собеседования)**

* **Java 5** — generics, annotations, enums, concurrency.
* **Java 6** — стабилизация, аннотационные процессоры, скрипты.
* **Java 7** — try-with-resources, diamond, NIO.2, ForkJoin.
* **Java 8** — лямбды, Stream API, Optional, java.time, default methods.
* **Java 11** — var, новый HTTP client, методы String, удаление старого JEE.
* **Java 17** — records, sealed, pattern matching, switch expressions, text blocks.
* **Java 21** — virtual threads, record patterns, pattern switch, sequenced collections.
