# Механизм обработки исключений try-catch-finally

## ✅ Зачем нужен:

Для **безопасного перехвата и обработки ошибок во время выполнения**, чтобы программа не завершалась аварийно.

## 🔧 Синтаксис

```java
try {
    // код, в котором может возникнуть исключение
} catch (ExceptionType e) {
    // обработка исключения
} finally {
    // выполняется всегда (даже при исключении)
}
```

---

## 📌 Поведение:

* **`try`**: содержит код, который может выбросить исключение.
* **`catch`**: перехватывает конкретный тип исключения (`IOException`, `NullPointerException` и т.д.).

    * Можно использовать несколько `catch` блоков (от более конкретного к более общему).
* **`finally`**: выполняется всегда, даже если:

    * произошло исключение,
    * был `return` или `throw`,
    * исключение не было поймано.

---

## 💡 Пример

```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Деление на ноль!");
} finally {
    System.out.println("Завершение блока.");
}
```

**Вывод:**

```
Деление на ноль!
Завершение блока.
```

---

## 🧠 Советы:

* Используй `finally` для **закрытия ресурсов** (`close()`, `disconnect()`).
* Для Java 7+ лучше применять `try-with-resources` с `AutoCloseable` для работы с файлами, потоками и т.п.
* Не лови `Exception` или `Throwable`, если нет крайней необходимости — это анти-паттерн.
