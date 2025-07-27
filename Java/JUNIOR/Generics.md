# Generics

#### ✅ Что это:

**Generics** — это механизм обобщённого программирования, 
позволяющий писать **универсальный, типобезопасный код**, работающий с разными типами данных.

---

## 📌 Зачем нужны:

* Избавляют от кастов (`(Type)`)
* Позволяют компилятору проверять типы на этапе компиляции
* Повышают читаемость и переиспользуемость кода

---

## 🧱 Объявление Generic'ов

### 1. Классы:

```java
class Box<T> {
    private T value;
    void set(T val) { value = val; }
    T get() { return value; }
}
```

**Использование:**

```java
Box<String> box = new Box<>();
box.set("Hello");
String val = box.get();
```

---

### 2. Методы:

```java
public <T> void print(T value) {
    System.out.println(value);
}
```

---

### 3. Интерфейсы:

```java
interface Repository<T> {
    void save(T entity);
}
```

---

## 🧭 Ограничения (bounded types):

### 🔹 Верхняя граница (`extends`)

```java
<T extends Number> T sum(T a, T b)
```

✅ Разрешены `Integer`, `Double`, `BigDecimal` и т.д.

### 🔸 Нижняя граница (`super`)

```java
<? super Integer> // любой родитель Integer: Number, Object
```

---

## ⚠️ Особенности:

* Нельзя создавать массивы generic-типа: `new T[10]` — ❌
* Есть **type erasure** — типы стираются в runtime
* `List<?>` — неизвестный тип, только для чтения

---

# 🎯 Wildcards (`?`, `<? extends T>`, `<? super T>`)

#### ✅ Что это:

**Wildcard (`?`)** — это подстановочный символ в обобщённом типе, который означает: **"некоторый неизвестный тип"**.

---

## 🔹 `?` — неизвестный тип (read-only)

```java
List<?> list = List.of("a", 1, true);
Object val = list.get(0); // ✅ можно читать
list.add("x");            // ❌ нельзя добавлять (кроме null)
```

Используется, когда **неважно, с каким типом работает коллекция**, и вы **только читаете**.

---

## 🔼 `<? extends T>` — "любой подкласс T" (covariant, только **read**)

```java
List<? extends Number> nums = List.of(1, 2.0);
Number n = nums.get(0);   // ✅ читать можно
nums.add(5);              // ❌ писать нельзя
```

**Чтение безопасно**, потому что всё — это как минимум `T`,
**запись небезопасна**, потому что тип неизвестен точно.

> ✅ Подходит, когда вы **читаете из коллекции**

---

## 🔽 `<? super T>` — "любой родитель T" (contravariant, только **write**)

```java
List<? super Integer> ints = new ArrayList<Number>();
ints.add(42);            // ✅ можно писать
Object val = ints.get(0); // ❌ читается как Object
```

**Запись безопасна**, потому что `Integer` — точно допустим,
**чтение небезопасно**, потому что тип — может быть `Object`, `Number` и т.п.

> ✅ Подходит, когда вы **записываете в коллекцию**

---

## 🧠 Запомни правило PECS:

> **Producer — Extends, Consumer — Super**

* **`? extends T`** → коллекция **производит** объекты (read-only)
* **`? super T`** → коллекция **потребляет** объекты (write-only)
