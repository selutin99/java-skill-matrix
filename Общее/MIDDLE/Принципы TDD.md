# Принципы TDD, интеграционное и нагрузочное тестирование

## ✅ TDD — Test-Driven Development

**Определение:**
**TDD (разработка через тестирование)** — это подход, при котором написание кода **начинается с тестов**, а затем создаётся минимальная реализация, удовлетворяющая этим тестам.

### 🔁 Цикл TDD: **Red → Green → Refactor**

1. **Red** — Написать падающий юнит-тест для новой функциональности.
2. **Green** — Реализовать минимальный код, чтобы тест прошёл.
3. **Refactor** — Упростить реализацию, сохраняя прохождение тестов.

### 🧱 Принципы TDD

* **Писать только тот код, который необходим для прохождения теста**
* **Каждая новая фича начинается с теста**
* **Тесты = спецификация поведения**
* **Много маленьких итераций**

### 📦 Преимущества TDD

* Более надёжный и протестированный код
* Повышение уверенности при рефакторинге
* Спецификация требований прямо в коде
* Меньше багов на проде

### ⚠️ Подводные камни

* Требует времени и дисциплины
* Не подходит для слишком быстро меняющихся требований
* Не заменяет другие виды тестирования (например, нагрузочное)

---

## 🔗 Интеграционное тестирование

**Цель:** Проверить, как **разные модули** системы **взаимодействуют между собой**.

### 👇 Отличие от юнит-тестов:

| Юнит-тесты                   | Интеграционные                                          |
| ---------------------------- | ------------------------------------------------------- |
| Тестируют *один класс/метод* | Тестируют *взаимодействие компонентов*                  |
| Заменяют зависимости (моки)  | Используют реальные зависимости (или частично реальные) |
| Быстрые                      | Медленнее, сложнее отлаживать                           |

### 🔧 Примеры:

* REST-контроллер + сервис + репозиторий + БД
* Kafka producer + consumer
* Микросервисы, взаимодействующие через API

### 🛠 Инструменты:

* **Spring Boot Test** (`@SpringBootTest`, `TestRestTemplate`)
* **Testcontainers** (для запуска реальных БД, брокеров и т.п.)
* **WireMock** / **MockServer** (моки для внешних HTTP-сервисов)

### ✅ Хорошая практика:

* Использовать интеграционные тесты для проверки "сквозных" бизнес-сценариев
* Не дублировать то, что уже покрыто юнит-тестами

---

## 🔥 Нагрузочное тестирование (Load Testing)

**Цель:** Проверить, как система **ведёт себя под нагрузкой**: высокой частотой запросов, большим объёмом данных и пользователей.

### 📈 Виды нагрузочного тестирования:

| Тип                | Цель                                                       |
| ------------------ | ---------------------------------------------------------- |
| **Load Testing**   | Оценка производительности под типичной и пиковой нагрузкой |
| **Stress Testing** | Проверка поведения при экстремальной нагрузке              |
| **Spike Testing**  | Резкий рост нагрузки (всплески)                            |
| **Soak Testing**   | Проверка устойчивости при длительной нагрузке              |

### 🛠 Инструменты:

* **JMeter** — классический инструмент для нагрузочного тестирования
* **Gatling** — сценарии на Scala, высокопроизводительный
* **k6** — лёгкий, скрипты на JavaScript
* **Locust** — Python-базированный фреймворк

### 📊 Метрики:

* Время ответа (Latency)
* Количество запросов в секунду (RPS, TPS)
* Утилизация CPU, RAM
* Процент ошибок
* Throughput (пропускная способность)

### 🧪 Хорошая практика:

* Тестировать с реалистичными сценариями (на основе прод-логов)
* Готовить среду, близкую к продакшену
* Анализировать узкие места (база, сеть, CPU, GC и т.д.)

---

## 🧩 Итоговое сравнение

| Вид теста          | Цель                      | Частота запуска   | Зависимости       |
| ------------------ | ------------------------- | ----------------- | ----------------- |
| **TDD/Юнит**       | Проверка логики классов   | Каждый коммит     | Изолированные     |
| **Интеграционные** | Проверка взаимодействий   | CI / Pull Request | Частично реальные |
| **Нагрузочные**    | Оценка производительности | Перед релизами    | Реальные системы  |
