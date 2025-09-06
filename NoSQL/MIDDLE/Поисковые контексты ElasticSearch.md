# Типы поисковых контекстов ElasticSearch и как строить запросы оптимальным образом

## 1. Основные поисковые контексты в ElasticSearch

1. **Query context** — вычисляет *релевантность* (score).

    * Примеры: `match`, `multi_match`, `bool` (`must/should/must_not`), `dis_max`, `function_score`, `range` (в режиме query).
    * Используется, когда важен порядок по score.

2. **Filter context** — *не влияет* на score, только отбирает документы; идеально для кэширования.

    * Пример: `bool.filter`, `constant_score.filter`, `term`/`terms`/`range` (как фильтр), `exists`.
    * Используйте для «жёстких» ограничений: статус, дата, категория, tenant.

3. **Post-filter** — фильтр, применяемый *после* агрегаций.

    * Нужен, когда в агрегациях должны участвовать документы до финального «дочищающего» фильтра.

4. **Aggregation context** — вычисление метрик и группировок (без score).

    * Примеры: `terms`, `date_histogram`, `stats`, `cardinality`, `filters`, `composite`.

5. **Rescore context** — пересчёт top-N результатов более «тяжёлым» запросом (обычно `match_phrase`).

    * Ускоряет: сначала дешёвый recall, потом точный re-rank для верхушки.

6. **Sorting context** — сортировка по полям.

    * Вытесняет score из порядка (если сортируете по полю, релевантность не влияет).

7. **Highlight / Suggest / Collapse**

    * **Highlight** — подсветка терминов (дорого; включайте адресно).
    * **Suggest** — подсказки (term/phrase/completion).
    * **Field collapse** — дедупликация по полю (например, по `product_id`).

8. **Пагинация/сканирование**

    * `from/size` — просто, но не для глубокой пагинации.
    * `search_after` + индексная сортировка — для «глубоких» листингов.
    * `point_in_time (PIT)` — стабильный срез для последовательной навигации.

---

## 2. Как строить запросы оптимально

1. **Разделяйте «поиск» и «фильтрацию»**

    * Релевантность: `query` (например, `bool.must`/`should`).
    * Узкие срезы: `bool.filter` (кэшируемо, дешевле).

   ```json
   {
     "query": {
       "bool": {
         "must":   [{ "multi_match": { "query": "iphone 13", "fields": ["title^3","description"] } }],
         "filter": [
           { "term": { "status": "active" } },
           { "range": { "price": { "lte": 1000 } } }
         ],
         "must_not": [{ "term": { "banned": true } }]
       }
     }
   }
   ```

2. **Используйте правильные поля: `text` vs `keyword`**

    * Полнотекст: `text` (анализируется).
    * Точное сравнение/агрегации/сортировка: `keyword`.
    * Мультиполя — `fields`: `name` (text) и `name.keyword` (keyword).

3. **Минимизируйте «тяжёлые» конструкции в основном запросе**

    * Скрипты/`script_score` — только при необходимости; лучше `function_score` с `field_value_factor`/decay-функциями.
    * Фразы — в `rescore`, а не в основном запросе:

   ```json
   {
     "query": { "match": { "title": "iphone 13" } },
     "rescore": {
       "window_size": 200,
       "query": {
         "rescore_query": {
           "match_phrase": { "title": { "query": "iphone 13", "slop": 1 } }
         },
         "query_weight": 1.0,
         "rescore_query_weight": 2.0
       }
     }
   }
   ```

4. **Контролируйте булеву логику для качества**

    * `bool.must` — обязателен и влияет на score.
    * `bool.filter` — обязателен, но без влияния на score.
    * `bool.should` — «хорошо бы»; срабатывает, если `minimum_should_match` задано (или нет must/filter).
    * Настройте `minimum_should_match` для тонкой управляемости.

   ```json
   {
     "query": {
       "bool": {
         "must": [{ "match": { "query": "smartphone" } }],
         "should": [
           { "match": { "title": "iphone" } },
           { "match": { "brand": "apple" } }
         ],
         "minimum_should_match": 1
       }
     }
   }
   ```

5. **Агрегации делайте экономно и осознанно**

    * Высокая кардинальность ⇒ осторожно с `terms`; подумайте о `composite` (пагинация по бケットам).
    * Если нужны фильтры по фасетам без влияния на агрегации — используйте `post_filter`.
    * Для «многих фильтров» — `filters` aggregation удобнее ряда `terms`.

6. **Пагинация: избегайте глубокого `from`**

    * Вместо `from: 20000` используйте `search_after` + устойчивую сортировку по уникальному ключу.
    * Для «длинного листинга» — `PIT` + `search_after`.

7. **Снижение нагрузки на I/O**

    * Возвращайте только нужные поля: `_source` include/exclude.
    * Отключайте `track_total_hits` или ставьте лимит (например, `10000`), если точное количество не критично.
    * `terminate_after` — для раннего стопа по шардy, когда достаточно n-документов.

8. **Используйте кэш там, где он работает**

    * Всё, что в `filter` и детерминировано (без скриптов/дат now) — отлично кэшируется.
    * Стабильные фильтры выносите в `filter`; даже `constant_score` может помочь:

   ```json
   {
     "query": {
       "constant_score": {
         "filter": { "term": { "category": "phones" } }
       }
     }
   }
   ```

9. **Сортировка и индексная сортировка**

    * Для частых сортировок по одному-двум полям задумайтесь о `index.sort` при создании индекса — ускоряет `search_after` и топ-k.
    * Для сортировки по `text` используйте `keyword`-подполе.

10. **Моделирование данных под запросы**

    * `nested` поля — если массив объектов должен фильтроваться как связанная единица.
    * Parent/Child — когда денормализация невозможна, но нужна раздельная жизнедеятельность документов.
    * Синонимы — используйте **synonym graph** analyzer для фразовых синонимов.
    * Нормализаторы для `keyword` (lowercase, asciifolding).

11. **Диагностика и контроль качества**

    * `profile: true` — профилирование запроса.
    * `explain: true` — разбора score (точечно, не в проде).
    * Пересматривайте маппинг: не включайте `fielddata` на `text`, используйте `keyword`.
    * Runtime fields — точечно; для постоянного доступа лучше материализовать.

12. **Типовые шаблоны запросов**

    * **Multi-field полнотекст + фильтры:**

      ```json
      {
        "query": {
          "bool": {
            "must": [{
              "multi_match": {
                "query": "wireless headphones",
                "fields": ["title^3","description","categories"],
                "type": "best_fields",
                "tie_breaker": 0.2
              }
            }],
            "filter": [
              { "terms": { "brand": ["sony","bose"] } },
              { "range": { "price": { "gte": 100, "lte": 300 } } }
            ]
          }
        },
        "_source": { "includes": ["id","title","price","brand"] },
        "track_total_hits": 10000
      }
      ```
    * **Фасеты с `post_filter`:**

      ```json
      {
        "aggs": {
          "brands": { "terms": { "field": "brand.keyword", "size": 20 } },
          "price_ranges": {
            "range": { "field": "price", "ranges": [{ "to":100 },{ "from":100,"to":300 },{ "from":300 }] }
          }
        },
        "query": { "match": { "title": "headphones" } },
        "post_filter": { "term": { "brand.keyword": "sony" } }
      }
      ```
    * **Глубокая пагинация:**

      ```json
      {
        "pit": { "id": "PIT_ID", "keep_alive": "1m" },
        "size": 50,
        "search_after": ["sony", "0000012389"],
        "sort": [{ "brand.keyword": "asc" }, { "id": "asc" }],
        "query": { "match_all": {} }
      }
      ```

---

## 3. Частые ошибки и как их избежать

* Смешивание фильтров в `must` → переносите в `filter` для кэша и скорости.
* Агрегации по `text` → используйте `keyword`.
* Глубокая пагинация `from/size` → переход на `search_after` + `PIT`.
* Скрипты в scoring «повсюду» → выносите в `rescore`/`function_score` для top-N.
* Полнотекст по `keyword` → ищите по `text`.
* «Толстый» `_source` → включайте только нужные поля.

---

## 4. Мини-шпаргалка для собеседования

* **Контексты:** `query` (score) vs `filter` (без score, кэшируется); `post_filter` для корректных агрегаций; `rescore` — точность на top-N; `aggregations` — метрики/фасеты.
* **Практики:** фильтры в `bool.filter`, текст в `text`, точные сравнения/сортировки в `keyword`, `search_after` + `PIT` для глубины, `_source`-фильтрация, `track_total_hits` разумно.
* **Качество поиска:** `multi_match`, `dis_max`/`tie_breaker`, `minimum_should_match`, синонимы (synonym graph), `rescore` с `match_phrase`.
* **Производительность:** кэшируемые фильтры, меньше скриптов, индексная сортировка, аккуратные агрегации (`composite` при глубине).
