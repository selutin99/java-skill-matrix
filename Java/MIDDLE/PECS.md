# Инвариантность дженериков, PECS, стирание типов

## **1. Инвариантность дженериков**

В Java дженерики **инвариантны**:
`List<Number>` **не является** ни подтипом `List<Object>`, ни подтипом `List<Integer>`.

**Почему так?**
Если бы `List<Integer>` можно было присвоить в `List<Number>`, то мы могли бы сделать так:

```java
List<Integer> integers = new ArrayList<>();
List<Number> numbers = integers; // если бы компилятор позволял
numbers.add(3.14); // Double — это Number
Integer i = integers.get(0); // BOOM! ClassCastException
```

Поэтому Java требует, чтобы тип параметра совпадал **точно**, если не используется **wildcard** (`?`).

---

## **2. Правило PECS (Producer Extends, Consumer Super)**

Это правило, введённое Джошуа Блохом, помогает запомнить, когда использовать `? extends` и `? super`.

* **PE** (Producer Extends) — если контейнер **производит** значения (мы только читаем), используем `? extends T`:

```java
List<? extends Number> list = List.of(1, 2, 3);
Number n = list.get(0); // читать можно
// list.add(42); // нельзя — неизвестно, Integer там или Double
```

* **CS** (Consumer Super) — если контейнер **потребляет** значения (мы только пишем), используем `? super T`:

```java
List<? super Integer> list = new ArrayList<Number>();
list.add(42); // можно — Integer подойдёт
Object obj = list.get(0); // читаем только как Object
```

* **Если контейнер и читает, и пишет** — используем **точный тип** `List<T>`.

---

## **3. Стирание типов (Type Erasure)**

Generics в Java — **фича времени компиляции**. В рантайме все типовые параметры **стираются**:
`List<String>` и `List<Integer>` во время выполнения оба становятся просто `List`.

**Последствия:**

1. **Нельзя** создавать объекты дженериков напрямую:

```java
List<String> list = new ArrayList<String>(); // OK
List<String> list = new ArrayList<>();       // OK
List<String> list = new ArrayList<String>(); // но new ArrayList<String>() — не новый тип
```

2. **Нельзя** создавать массивы параметризованных типов:

```java
List<String>[] arr = new List<String>[10]; // Compile error
```

3. **Нельзя** перегружать методы, различающиеся только generic-параметрами:

```java
void process(List<String> list) { }
void process(List<Integer> list) { } // Compile error — одинаковая сигнатура после стирания
```

4. **`instanceof` работает только с raw-типами**:

```java
if (obj instanceof List<String>) { } // Compile error
if (obj instanceof List<?>) { }      // OK
```

**Как это работает под капотом**
Компилятор подставляет **upper bound** (по умолчанию `Object`) вместо параметра:

```java
class Box<T> {
    T value;
    T get() { return value; }
}
```

После стирания:

```java
class Box {
    Object value;
    Object get() { return value; }
}
```

А касты добавляются автоматически в местах вызова.

---

💡 **Краткий конспект для собеседования:**

* Инвариантность — без wildcard типы должны совпадать точно.
* PECS — `? extends` для чтения, `? super` для записи.
* Type erasure — generics существуют только на этапе компиляции, в рантайме это обычные объекты без информации о параметре типа.
