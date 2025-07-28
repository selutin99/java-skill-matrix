# Reflection, функционал и пример

**Reflection (рефлексия)** — механизм Java, позволяющий **изучать 
и изменять структуру классов, интерфейсов, методов и полей во время выполнения**.

---

### Что позволяет делать:

* Получать информацию о классах (`Class`, `Method`, `Field`, `Constructor`)
* Вызывать методы и конструкторы
* Изменять значения полей (в т.ч. `private`)
* Создавать экземпляры классов динамически
* Работать с аннотациями

---

### Пример использования:

```java
import java.lang.reflect.*;

public class Demo {
    private String msg = "Hello";

    private void printMsg() {
        System.out.println(msg);
    }

    public static void main(String[] args) throws Exception {
        Demo obj = new Demo();
        Class<?> cls = obj.getClass();

        // Доступ к приватному полю
        Field field = cls.getDeclaredField("msg");
        field.setAccessible(true);
        field.set(obj, "Hi via Reflection");

        // Вызов приватного метода
        Method method = cls.getDeclaredMethod("printMsg");
        method.setAccessible(true);
        method.invoke(obj);  // Выведет: Hi via Reflection
    }
}
```

---

### Когда используется:

* Фреймворки (Spring, Hibernate)
* Тестирование (JUnit, Mockito)
* Сериализация/десериализация
* Работа с аннотациями

---

📌 Минусы:

* Медленнее обычного вызова
* Нарушает инкапсуляцию
* Может вызывать ошибки в рантайме

**Вывод:** рефлексия — мощный инструмент для динамической работы с кодом, но использовать её нужно осознанно.
