# Как писать Dockerfile'ы и docker-compose.yaml файлы

## 1. Dockerfile — описание образа

### 1.1. Базовая структура

```dockerfile
# Базовый образ
FROM eclipse-temurin:17-jdk-alpine

# Рабочая директория
WORKDIR /app

# Копируем зависимости (для кеша)
COPY pom.xml .
COPY src ./src

# Сборка (если делаем внутри контейнера)
RUN ./mvnw package -DskipTests

# Копируем JAR (если multi-stage — см. ниже)
COPY target/app.jar app.jar

# Открываем порт
EXPOSE 8080

# Команда запуска
ENTRYPOINT ["java","-jar","app.jar"]
```

### 1.2. Multi-stage build (рекомендуется)

```dockerfile
# --- stage 1: build ---
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /build
COPY pom.xml .
COPY src ./src
RUN mvn -B -DskipTests clean package

# --- stage 2: runtime ---
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /build/target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```

* **Плюсы**: образ маленький, не содержит инструментов сборки.

### 1.3. Best practices

* Использовать **минимальные базовые образы** (например, `alpine`, `distroless`).
* **Multi-stage**: build → runtime.
* Всегда задавать **WORKDIR**.
* Использовать `COPY` вместо `ADD` (только для архива/URL — `ADD`).
* Стараться объединять `RUN` в одну строку:

  ```dockerfile
  RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
  ```
* Не хранить секреты в образе (использовать `--build-arg` или переменные среды при запуске).
* Для production — фиксировать версии пакетов/библиотек.

---

## 2. docker-compose.yaml — многоконтейнерные приложения

### 2.1. Базовый пример

```yaml
version: '3.9'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/mydb
      SPRING_DATASOURCE_USERNAME: user
      SPRING_DATASOURCE_PASSWORD: pass
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

### 2.2. Часто используемые параметры

* **build** — путь к Dockerfile (`.` или `./app`).
* **image** — готовый образ (`postgres:16`).
* **ports** — маппинг портов (`8080:8080`).
* **volumes** — монтирование данных/каталогов.
* **environment** — переменные окружения.
* **depends\_on** — порядок старта сервисов.
* **networks** — для кастомных сетей (по умолчанию один `bridge`).

### 2.3. Пример с Redis + Nginx

```yaml
version: "3.9"
services:
  web:
    build: ./web
    ports:
      - "80:80"
    depends_on:
      - api

  api:
    build: ./api
    environment:
      REDIS_HOST: redis
    depends_on:
      - redis

  redis:
    image: redis:7
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

---

## 3. Частые команды

```bash
# Собрать образ
docker build -t myapp:1.0 .

# Запустить контейнер
docker run -d -p 8080:8080 myapp:1.0

# Поднять окружение
docker-compose up -d

# Остановить и удалить
docker-compose down -v

# Логи сервиса
docker-compose logs -f app
```

---

## 4. Best practices для docker-compose

* Хранить `docker-compose.override.yml` для dev-версии (с отладкой, live reload).
* Для production использовать отдельный файл:

  ```bash
  docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
  ```
* Секреты хранить в `.env` файле (подключается автоматически).
* Логические группы сервисов лучше выносить в разные сети (`frontend`, `backend`).

---

## 5. Выжимка для собеседования

* **Dockerfile**: инструкция для сборки образа; важные команды `FROM`, `WORKDIR`, `COPY`, `RUN`, `CMD`, `ENTRYPOINT`, `EXPOSE`.
* **Multi-stage build** уменьшает размер образа и исключает сборочные зависимости.
* **Best practices**: минимальные образы, объединение RUN, не хранить секреты, фиксировать версии.
* **docker-compose.yaml**: декларативное описание многоконтейнерного окружения, YAML-ключи: `services`, `volumes`, `networks`, `depends_on`, `environment`, `ports`.
* **Разделение окружений**: override-файлы (`docker-compose.override.yml`, `docker-compose.prod.yml`).
* **Volumes** → для сохранения данных, **networks** → для связи сервисов.
