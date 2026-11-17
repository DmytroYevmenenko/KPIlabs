# Слід роботи з ШІ (ai/prompts.md)

## 1. Вхідний документ: намітки та вимоги (написано власноруч)
Треба зробити базу даних для прокату електросамокатів.
Основні сутності:
- користувач (ім'я, пошта)
- станція (адреса, місткість)
- самокат (модель, заряд, на якій станції стоїть)
- поїздка (хто взяв, який самокат, коли почав, коли закінчив, скільки коштувало)

Вимоги:
1. Всі ID мають бути рядками (string, під UUID).
2. Самокат у процесі поїздки їде по місту і не прив'язаний до жодної станції, тому його station_id стає null.
3. Зв'язок між користувачем і самокатом обов'язково робимо через окрему сутність поїздки, щоб зберігати час і гроші.

---

## 2. Лог діалогу з ШІ

### Промпт 1 (дослівний)
> Склади специфікацію та Mermaid ER-діаграму для сервісу прокату електросамокатів на основі моїх наміток:
> 1. USER (Користувач): id (string), name (string), email (string)
> 2. STATION (Паркувальна станція): id (string), location_name (string), capacity (int)
> 3. SCOOTER (Самокат): id (string), model (string), battery_level (int), station_id (string, FK, nullable)
> 4. RENTAL (Поїздка): id (string), user_id (string, FK), scooter_id (string, FK), start_time (datetime), end_time (datetime, nullable), total_cost (decimal)
> Важливо: всі ключі string, самокат під час поїздки не на станції (station_id nullable), поїздка як окрема сутність.

### Сира відповідь ШІ на Промпт 1
> Ось проєкт специфікації та ER-діаграми у форматі Mermaid для вашого сервісу прокату самокатів:
>
> ```mermaid
> erDiagram
>     USER {
>         string id PK
>         string name
>         string email
>     }
>     STATION {
>         string id PK
>         string location_name
>         int capacity
>     }
>     SCOOTER {
>         string id PK
>         string model
>         int battery_level
>         string station_id FK
>     }
>     RENTAL {
>         string id PK
>         string user_id FK
>         string scooter_id FK
>         datetime start_time
>         datetime end_time
>         decimal total_cost
>     }
>     STATION ||--o{ SCOOTER : hosts
>     USER ||--o{ RENTAL : makes
>     SCOOTER ||--o{ RENTAL : used_in
> ```
>
> Уточнювальні питання:
> 1. Чи потрібно додати обмеження унікальності (UK) для email користувача?
> 2. Які допустимі межі значень для заряду батареї (battery_level) та місткості станції (capacity)?
> 3. Як система розуміє, що поїздка ще триває?

### Мої зауваження до виходу ШІ та відповіді на уточнювальні питання
1. **Зауваження до діаграми:** У згенерованому коді ти поставив жорсткий зв'язок `STATION ||--o{ SCOOTER : hosts`. Символ `||` означає, що у самоката завжди обов'язково є станція. Це суперечить моїй вимозі, що під час поїздки `station_id` є `nullable`. Треба змінити кардинальність на опціональну: `STATION |o--o{ SCOOTER : hosts`. Також додай у діаграму позначки `"nullable"` біля `station_id` та `end_time`.
2. **Відповідь на питання 1 (email):** Так, `email` має бути унікальним (`UK`), щоб один користувач не реєстрував дублікати акаунтів.
3. **Відповідь на питання 2 (межі значень):** Заряд батареї `battery_level` має бути від 0 до 100 (відсотків), а місткість станції `capacity` — суворо більше 0.
4. **Відповідь на питання 3 (активна поїздка):** Поїздка вважається активною, поки поле `end_time` має значення `null`. Коли користувач завершує поїздку, записується час завершення і розраховується `total_cost`.

---

### Промпт 2 (дослівний)
> Врахуй мої правки: зміни зв'язок між STATION і SCOOTER на опціональний (`|o--o{`), додай UK для email, додай у специфікацію та діаграму обмеження для battery_level (0..100), capacity (>0) та nullable для end_time. Перевір модель на відповідність 3NF.

### Сира відповідь ШІ на Промпт 2
> Враховано. Ось оновлена Mermaid-діаграма з виправленою кардинальністю та обмеженнями:
>
> ```mermaid
> erDiagram
>     USER {
>         string id PK
>         string name
>         string email UK
>     }
>     STATION {
>         string id PK
>         string location_name
>         int capacity "greater than 0"
>     }
>     SCOOTER {
>         string id PK
>         string model
>         int battery_level "0 to 100"
>         string station_id FK "nullable"
>     }
>     RENTAL {
>         string id PK
>         string user_id FK
>         string scooter_id FK
>         datetime start_time
>         datetime end_time "nullable"
>         decimal total_cost
>     }
>     STATION |o--o{ SCOOTER : hosts
>     USER ||--o{ RENTAL : makes
>     SCOOTER ||--o{ RENTAL : used_in
> ```
>
> Перевірка 3NF:
> - 1NF: Усі атрибути атомарні, відсутні масиви чи повторювані групи.
> - 2NF: Усі сутності мають простий первинний ключ (`id`), часткових залежностей немає.
> - 3NF: Відсутні транзитивні залежності між неключовими атрибутами (вартість `total_cost` залежить лише від конкретної сесії `RENTAL`, а заряд `battery_level` — лише від конкретного `SCOOTER`).

### Моє фінальне зауваження до виходу ШІ
Тепер діаграма відповідає моїм вимогам і специфікації. Перевірив закриття блоку коду Mermaid, переніс фінальну версію в `diagram.md` та оновив критерії прийняття в `spec.md`.