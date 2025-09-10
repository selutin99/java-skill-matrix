# Как пишутся логи в приложении, что такое ELK

## 1. Как пишутся логи в приложении

### 1.1. Для чего нужны логи

* Отладка (debugging) → понять поведение кода.
* Аудит (кто что сделал).
* Мониторинг → SLA/SLO.
* Инциденты → «что пошло не так».

### 1.2. Подходы к логированию

* **Уровни логов** (обычные для Java/Spring):

    * `TRACE` — подробные шаги, обычно выключены.
    * `DEBUG` — отладка, детали.
    * `INFO` — бизнес-события, нормальный флоу.
    * `WARN` — потенциальные проблемы.
    * `ERROR` — ошибки, требующие внимания.

* **Лучшие практики**

    * Логировать в **структурированном формате** (JSON).
    * Не логировать пароли/секреты.
    * Не злоупотреблять DEBUG/TRACE в продакшене.
    * Логи должны быть **машиночитаемые** (для сбора ELK/Prometheus).

### 1.3. Пример (Spring Boot + Kotlin/Java)

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MyService {
    private static final Logger log = LoggerFactory.getLogger(MyService.class);

    public void process(String input) {
        log.info("Processing request: {}", input);
        try {
            // ...
        } catch (Exception e) {
            log.error("Failed to process input {}", input, e);
        }
    }
}
```

**application.yaml**

```yaml
logging:
  level:
    root: INFO
    com.myapp: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{36} - %msg%n"
```

---

## 2. Что такое ELK

**ELK стек** = **Elasticsearch + Logstash + Kibana**

* **Elasticsearch** — хранилище логов (search engine, JSON-документы, full-text поиск).
* **Logstash** — сбор и обработка логов (парсинг, фильтры, доставка в ES).
* **Kibana** — визуализация (дашборды, поиск по логам, аналитика).

Позже добавили **Beats** (Filebeat, Metricbeat и т.д.) → часто пишут **ELK/EFK**:

* **Filebeat/Fluentd** — лёгкие агенты на серверах/Pod’ах, собирают логи и отправляют в Elasticsearch.

---

## 3. Как устроена система логирования с ELK

1. **Приложение** пишет логи в stdout (или файл).

    * В Kubernetes обычно → stdout/stderr.
2. **Агент (Filebeat/Fluentd)** собирает эти логи.

    * Читает файлы или Docker-логи.
3. **Logstash** (опционально) парсит, фильтрует, обогащает.
4. **Elasticsearch** хранит структурированные записи.
5. **Kibana** визуализирует и даёт поиск (по уровням, трассам, пользователям).

---

## 4. Пример docker-compose (ELK + Filebeat)

```yaml
version: "3.9"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.14.0
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"

  kibana:
    image: docker.elastic.co/kibana/kibana:8.14.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.14.0
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
```

**filebeat.yml** (сбор stdout контейнеров)

```yaml
filebeat.inputs:
  - type: container
    paths:
      - '/var/lib/docker/containers/*/*.log'

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
```

---

## 5. Пример записи лога в Elasticsearch (JSON-документ)

```json
{
  "@timestamp": "2025-09-10T08:21:22Z",
  "log.level": "ERROR",
  "message": "Failed to connect to DB",
  "service.name": "myapp",
  "host": "node-1",
  "stacktrace": "java.sql.SQLException..."
}
```

---

## 6. Best practices для логирования + ELK

* **Логи = stdout/stderr** → удобнее собирать в контейнерах.
* Использовать **JSON-формат** (легко парсить).
* В Kibana → дашборды с фильтрами по уровню/сервису/traceId.
* В связке с **Tracing (Jaeger/Tempo)** — можно связать логи и трассировки.
* Для облаков: лучше использовать готовые сервисы (AWS OpenSearch, GCP Logging, Azure Monitor).

---

## 7. Выжимка для собеседования

* **Логи** = основной источник информации о работе системы. Пишутся через логгер (SLF4J, Logback, Log4j).
* Уровни логов: TRACE, DEBUG, INFO, WARN, ERROR.
* Логировать лучше структурировано (JSON), без секретов.
* **ELK** = Elasticsearch (хранилище + поиск), Logstash (сбор/парсинг), Kibana (визуализация).
* **EFK** = Elasticsearch + Fluentd/Filebeat + Kibana.
* Поток: приложение → stdout → Filebeat/Fluentd → Elasticsearch → Kibana.
* **Плюсы ELK**: поиск, фильтрация, дашборды, централизованное хранилище.
