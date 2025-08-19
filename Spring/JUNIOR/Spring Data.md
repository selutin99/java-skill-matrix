# Spring Data

## 1. Что это такое

**Spring Data** — это часть экосистемы Spring, упрощающая работу с базами данных.
Она решает задачи:

* убрать рутину при написании DAO,
* дать единый API для разных источников данных (SQL, NoSQL, графовые БД),
* предоставить декларативный способ работы с репозиториями.

---

## 2. Основные возможности Spring Data

1. **Spring Data JPA**

    * Расширение над JPA/Hibernate.
    * Позволяет писать репозитории как интерфейсы без реализации.

2. **CRUD-репозитории "из коробки"**

    * Методы `save()`, `findById()`, `findAll()`, `deleteById()`.

3. **Query Methods**

    * Автогенерация запросов из названий методов:

      ```java
      List<User> findByLastName(String lastName);
      User findByEmailAndStatus(String email, Status status);
      ```

4. **JPQL/SQL/Native Queries**

    * Поддержка `@Query` для гибких запросов.

5. **Pagination и Sorting**

    * Интерфейсы `Pageable` и `Sort` для постраничного вывода и сортировки.

6. **Auditing**

    * Автоматическое заполнение полей вроде `createdDate`, `lastModifiedBy`.

7. **Поддержка разных хранилищ**

    * SQL: JPA, JDBC.
    * NoSQL: MongoDB, Cassandra, Redis, Elasticsearch.

---

## 3. Основные интерфейсы репозиториев

* `Repository<T, ID>` — базовый маркер.
* `CrudRepository<T, ID>` — CRUD-операции.
* `PagingAndSortingRepository<T, ID>` — добавляет пагинацию и сортировку.
* `JpaRepository<T, ID>` — расширение для JPA (чаще всего используется).

Пример:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByLastName(String lastName);
}
```

---

## 4. Как работает под капотом

* Spring создаёт **прокси** для интерфейса репозитория.
* Анализирует сигнатуру методов и генерирует SQL/JPA-запрос.
* Если метод не стандартный, можно использовать `@Query`.
* Вся интеграция строится через `EntityManager`.

---

## 5. Примеры

### Репозиторий

```java
public interface BookRepository extends JpaRepository<Book, Long> {
    List<Book> findByAuthor(String author);
    
    @Query("SELECT b FROM Book b WHERE b.title LIKE %:title%")
    List<Book> searchByTitle(@Param("title") String title);
}
```

### Сервис

```java
@Service
public class BookService {
    private final BookRepository repository;

    public BookService(BookRepository repository) {
        this.repository = repository;
    }

    public List<Book> findByAuthor(String author) {
        return repository.findByAuthor(author);
    }
}
```

---

## 6. Подводные камни

* **N+1 проблема** — нужно использовать `@EntityGraph` или `fetch join`.
* **LazyInitializationException** при работе вне транзакции.
* Автоматическая генерация запросов может быть неэффективной для сложных кейсов.
* Для крупных проектов часто комбинируют Spring Data + кастомный DAO слой.

---

## 7. Выжимка для собеседования

* **Spring Data** — упрощает работу с БД через репозитории.
* **JpaRepository** — самый популярный интерфейс (CRUD + пагинация + сортировка).
* **Запросы**: методы по имени (`findByX`), аннотация `@Query`, нативный SQL.
* **Pagination & Sorting** встроены (`Pageable`, `Sort`).
* Поддерживает **SQL и NoSQL** БД.
* Работает через **прокси над EntityManager**.
* Подводные камни: **N+1, lazy loading, производительность**.
