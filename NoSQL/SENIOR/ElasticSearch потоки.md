# Что произойдет при одновременно изменении документа ElasticSearch разными потоками, каким образом обновить маппинг на существующем индексе

## 1. Конкурентные изменения документа в ElasticSearch

* **Модель**: ES использует **оптимистическую блокировку**. У каждого документа есть `_seq_no` и `_primary_term`.
* **Что произойдёт «по умолчанию»**: если два потока почти одновременно делают `index/update` без контроля версий, победит **последняя запись** («last write wins»). Предыдущие изменения могут быть перезаписаны.
* **Конфликты**: если вы укажете контроль версий, ES вернёт `409 version_conflict_engine_exception`, и запись **не применится**.
* **Почему update опасен**: `POST /_update` делает read-modify-write. Между чтением `_source` и записью другой поток может изменить документ → получите конфликт.

### Как безопасно обновлять

* **if\_seq\_no & if\_primary\_term** (рекомендуется):

  ```http
  PUT /idx/_doc/42?if_seq_no=17&if_primary_term=3
  { "field": "new" }
  ```

  Если `_seq_no`/`_primary_term` не совпадут — 409.
* **retry\_on\_conflict** для `_update` (авторетрай при конфликте):

  ```http
  POST /idx/_update/42?retry_on_conflict=5
  { "script": { "source": "ctx._source.cnt += params.inc", "params": { "inc": 1 } } }
  ```
* **Скриптовые инкременты/мердж**: изменения атомарно применяются на шарде, минимизируя race conditions.
* **External versioning** (реже): `?version=123&version_type=external` — вы контролируете номер версии извне.

---

## 2. Что можно/нельзя менять в mapping существующего индекса

### Можно (без переиндексации)

* **Добавлять новые поля** в `properties`.

  ```http
  PUT /idx/_mapping
  {
    "properties": {
      "newField": { "type": "keyword" }
    }
  }
  ```
* **Добавлять multi-fields** (подполя) к существующему `text/keyword` (если тип не меняется).
* **Добавлять runtime fields** (вычисляемые на чтении).
* **Добавлять новые analyzer-компоненты в settings** (но это **не** меняет анализ уже проиндексированных полей).
* **Менять некоторые динамические settings** (не связанные с анализом/индексной сортировкой).

### Нельзя (требуется новый индекс + reindex)

* Менять **тип поля** (`text` → `keyword`, `long` → `date`, и т.п.).
* Менять **анализатор** существующего поля (`analyzer`, `search_analyzer`).
* Включать/отключать **doc\_values**, **norms**, **index: true/false** для уже созданного поля.
* Менять **index.sort** (индексная сортировка задаётся только при создании).

---

## 3. Правильное обновление схемы без простоя (zero-downtime)

1. **Создайте новый индекс** с нужными settings/mapping (например, `idx_v2`).

   ```http
   PUT /idx_v2
   {
     "settings": { "index": { "number_of_shards": 3, "analysis": { /* ... */ } } },
     "mappings": { "properties": { "title": { "type": "text", "fields": { "raw": { "type": "keyword" }}}}}
   }
   ```
2. **Переиндексируйте** данные:

   ```http
   POST /_reindex
   { "source": { "index": "idx" }, "dest": { "index": "idx_v2" } }
   ```
3. **Переключите alias** атомарно:

   ```http
   POST /_aliases
   {
     "actions": [
       { "remove": { "index": "idx", "alias": "products" } },
       { "add":    { "index": "idx_v2", "alias": "products" } }
     ]
   }
   ```

   Клиенты всегда ходят через alias `products` — переключение прозрачно.
4. (Опционально) Запустите **dual-write** на время миграции, или сделайте **reindex from remote/PIT** + финальный дельта-догон.

---

## 4. Короткая выжимка для собеседования

* **Concurrency**: ES — оптимистическая блокировка. Без контроля версий — «последняя запись победит». Используйте `if_seq_no` + `if_primary_term`, `retry_on_conflict`, скриптовые апдейты.
* **Mapping live-update**: можно только **добавлять** поля/multi-fields/runtime. Менять тип/анализатор — **нельзя** → нужен новый индекс и **reindex**.
* **Zero-downtime**: новый индекс → `_reindex` → переключение **alias** одним действием.
