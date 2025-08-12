# Dead lock, как он образуется и как этого избежать

## **1. Что такое deadlock**

**Deadlock (взаимная блокировка)** — это ситуация, когда два или более потока навсегда ждут освобождения ресурсов, которые они блокируют друг у друга.
В результате программа зависает, хотя процессор может быть свободен.

---

## **2. Как он образуется**

Классическая схема:

1. Поток A захватывает **Ресурс 1**.
2. Поток B захватывает **Ресурс 2**.
3. Поток A пытается взять **Ресурс 2**, но он уже занят Потоком B.
4. Поток B пытается взять **Ресурс 1**, но он уже занят Потоком A.
5. Оба потока ждут друг друга бесконечно.

Пример на Java:

```java
class DeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void method1() {
        synchronized (lock1) {
            System.out.println("Thread 1: lock1");
            try { Thread.sleep(50); } catch (InterruptedException ignored) {}
            synchronized (lock2) {
                System.out.println("Thread 1: lock2");
            }
        }
    }

    public void method2() {
        synchronized (lock2) {
            System.out.println("Thread 2: lock2");
            try { Thread.sleep(50); } catch (InterruptedException ignored) {}
            synchronized (lock1) {
                System.out.println("Thread 2: lock1");
            }
        }
    }
}
```

Запустим так:

```java
DeadlockExample ex = new DeadlockExample();
new Thread(ex::method1).start();
new Thread(ex::method2).start();
```

Оба потока застрянут.

---

## **3. Четыре условия возникновения deadlock** (по **Коффману**)

Чтобы произошла взаимная блокировка, должны выполняться все 4 условия:

1. **Взаимное исключение** — ресурс может быть занят только одним потоком одновременно.
2. **Удержание и ожидание** — поток, удерживающий ресурс, ждёт другой ресурс.
3. **Отсутствие принудительного освобождения** — ресурсы нельзя отобрать, только поток сам их освобождает.
4. **Циклическое ожидание** — есть цикл из потоков, где каждый ждёт ресурс у следующего.

Разрушив любое из условий, можно избежать deadlock.

---

## **4. Как избежать deadlock**

### **A. Фиксированный порядок захвата ресурсов**

Всегда брать локи в одном и том же порядке.

```java
void methodSafe() {
    synchronized (lock1) {
        synchronized (lock2) {
            // безопасно, так как все потоки блокируют lock1 → lock2
        }
    }
}
```

### **B. Таймаут при блокировке**

Использовать `tryLock()` с таймаутом (например, `ReentrantLock`):

```java
if (lock1.tryLock(100, TimeUnit.MILLISECONDS)) {
    try {
        if (lock2.tryLock(100, TimeUnit.MILLISECONDS)) {
            try {
                // критическая секция
            } finally { lock2.unlock(); }
        }
    } finally { lock1.unlock(); }
}
```

Если не удалось — повторить или выйти.

### **C. Минимизировать время удержания блокировки**

Держать локи как можно меньше, не вызывать `sleep()`, I/O и тяжёлые операции в секциях `synchronized`.

### **D. Использовать структуры без блокировок**

Например, `ConcurrentHashMap`, `Atomic*` или алгоритмы на CAS.

### **E. Обнаружение deadlock**

В реальных системах можно мониторить через:

* `jconsole` / `VisualVM` (`Thread Dump` покажет deadlock)
* `ThreadMXBean.findDeadlockedThreads()` из `java.lang.management`

---

## **5. Краткая формула для интервью**

> Deadlock возникает, когда потоки ждут ресурсы друг у друга, и выполнение останавливается навсегда.
> Чтобы избежать — захватывать ресурсы в одном порядке, использовать таймауты, минимизировать время удержания блокировок и при необходимости использовать неблокирующие структуры.
