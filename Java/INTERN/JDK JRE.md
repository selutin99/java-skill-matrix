# Что такое JDK и JRE, чем отличаются

## 🔹 **JDK (Java Development Kit)**

Это **набор для разработки** Java-приложений.

**Содержит:**

* **JRE** (внутри)
* Компилятор `javac`
* Отладчик `jdb`
* Инструменты (`jarsigner`, `javadoc`, `javap`, `jshell`, и т.д.)

> 📦 Всё, что нужно для **разработки** Java-кода.

---

## 🔹 **JRE (Java Runtime Environment)**

Это **среда выполнения** Java-программ.

**Содержит:**

* JVM (Java Virtual Machine)
* Базовые библиотеки (`rt.jar`, modules)
* Файлы конфигурации

> 📦 Всё, что нужно, чтобы **запустить** готовое Java-приложение.

---

## 🔍 В чём разница?

|                   | JDK                 | JRE           |
| ----------------- | ------------------- | ------------- |
| Назначение        | Разработка и запуск | Только запуск |
| Включает JRE?     | ✅ Да                | ❌ Нет         |
| Содержит `javac`? | ✅ Да                | ❌ Нет         |

---

👉 С 2019 года (с JDK 11) **JRE как отдельный пакет не распространяется** — используется JDK даже на проде.
