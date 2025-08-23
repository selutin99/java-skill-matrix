# Что такое Bean Factory Processor и Bean Factory Post Processor, для чего они используются и что можно сделать с их помощью

## 1. BeanFactoryPostProcessor

### Что это

* Интерфейс:

  ```java
  public interface BeanFactoryPostProcessor {
      void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory);
  }
  ```
* Вызывается **после загрузки всех `BeanDefinition`**, но **до создания бинов**.
* То есть работает **с метаданными (BeanDefinition)**, а не с самими объектами.

### Для чего

* Изменение конфигурации перед созданием бинов:

    * правка свойств (`BeanDefinition`),
    * замена классов,
    * регистрация новых `BeanDefinition`,
    * чтение/обработка кастомных аннотаций.

### Примеры применения

* **PropertySourcesPlaceholderConfigurer** (обрабатывает `${...}` в `@Value`).
* **Custom BeanDefinitionRegistryPostProcessor** для добавления динамических бинов.
* Подмена класса реализации:

  ```java
  beanFactory.getBeanDefinition("service")
             .setBeanClass(AlternativeService.class);
  ```

---

## 2. BeanPostProcessor

### Что это

* Интерфейс:

  ```java
  public interface BeanPostProcessor {
      Object postProcessBeforeInitialization(Object bean, String beanName);
      Object postProcessAfterInitialization(Object bean, String beanName);
  }
  ```
* Вызывается **для каждого созданного бина**:

    * до вызова `@PostConstruct/afterPropertiesSet()`,
    * после инициализации.

### Для чего

* Изменение/обёртка самого бина.
* Можно вернуть **оригинальный бин** или **заменить на прокси**.

### Примеры применения

* **AutowiredAnnotationBeanPostProcessor** — внедряет зависимости по `@Autowired`.
* **TransactionalAnnotationBeanPostProcessor** — оборачивает бин в транзакционный прокси.
* **AsyncAnnotationBeanPostProcessor** — оборачивает бин в async-прокси.
* Кастомная логика: логирование создания бинов, внедрение дополнительных зависимостей, подмена поведения.

---

## 3. Сравнение

| Характеристика   | BeanFactoryPostProcessor             | BeanPostProcessor                           |
| ---------------- | ------------------------------------ | ------------------------------------------- |
| Когда вызывается | До создания бинов                    | При создании каждого бина                   |
| Работает с       | `BeanDefinition` (метаданные)        | С объектом бина                             |
| Уровень          | «Метаданные конфигурации»            | «Runtime объект»                            |
| Типичные задачи  | Изменить конфигурацию, добавить бины | Обернуть бин в прокси, внедрить зависимости |
| Примеры          | PropertySourcesPlaceholderConfigurer | Autowired, Async, Transactional, Cacheable  |

---

## 4. BeanDefinitionRegistryPostProcessor

* Подвид `BeanFactoryPostProcessor`.
* Даёт доступ к `BeanDefinitionRegistry` → можно **регистрировать новые бины программно**.
* Используется в Spring Boot Autoconfiguration (например, чтобы подтянуть классы через `@Conditional`).

---

## 5. Выжимка для собеседования

* **BeanFactoryPostProcessor** работает на уровне **конфигурации**: изменяет `BeanDefinition` до создания бинов.

    * Пример: PropertyPlaceholderConfigurer, динамическая регистрация бинов.
* **BeanPostProcessor** работает на уровне **объектов**: перехватывает каждый бин при создании.

    * Пример: внедрение зависимостей, создание прокси (`@Transactional`, `@Async`, `@Cacheable`).
* Разница: один меняет **чертёж** (BeanDefinition), другой — **готовый объект**.
* Для расширения Spring чаще всего создают **BeanPostProcessor** (AOP, прокси).
* Для фреймворков и сложных интеграций → **BeanFactoryPostProcessor** (динамическая конфигурация).
