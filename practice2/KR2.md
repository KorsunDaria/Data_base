# Контрольная работа №2 по курсу «Базы данных»


# Вариант 1. Авиакомпания

## 1.1 Допущения

- Модель самолёта — отдельная сущность: тип (пассажирский/грузовой) и вместимость определяются моделью. Вместимость — число мест для пассажирского типа, грузоподъёмность (кг) для грузового.
- У самолёта может быть несколько сертификатов (история продлений); «действительный» определяется по сроку действия. Должен быть минимум один сертификат.
- Неисправности — отдельная сущность, у самолёта их 0..N; исправленная неисправность получает дату устранения.
- Аэропорты вынесены в справочник и идентифицируются кодом IATA.
- Тип перевозки рейса должен совпадать с типом самолёта, выполняющего рейс (проверяется триггером/приложением).
- Бронирование — ассоциативная сущность между пассажиром и рейсом. Для пассажирского рейса заполняется `seats_count`, для грузового — `cargo_weight`; ровно одно из двух (CHECK).
- Рейс идентифицируется уникальным номером `flight_number`.

## 1.2 ER-диаграмма (Crow's Foot)

```mermaid
erDiagram
    AIRCRAFT_MODEL["Модель самолёта"] {
        string model_name PK "Название модели"
        string aircraft_type "Тип: пассажирский/грузовой"
        int capacity "Места или грузоподъёмность"
    }
    AIRCRAFT["Самолёт"] {
        string reg_number PK "Регистрационный номер"
    }
    FLIGHT_CERT["Летный сертификат"] {
        int cert_id PK
        string cert_number UK "Номер сертификата"
        date issue_date "Дата выдачи"
        date valid_until "Действителен до"
        blob scan "Скан"
    }
    DEFECT["Неисправность"] {
        int defect_id PK
        string description "Описание"
        date detected_at "Дата обнаружения"
        date fixed_at "Дата устранения"
    }
    AIRPORT["Аэропорт"] {
        string iata_code PK "Код IATA"
        string name "Название"
        string city "Город"
    }
    FLIGHT["Рейс"] {
        string flight_number PK "Номер рейса"
        string cargo_type "Тип перевозки"
        datetime departure_dt "Дата и время вылета"
        datetime expected_arrival_dt "Ожидаемое прибытие"
    }
    PASSENGER["Пассажир"] {
        int passenger_id PK
        string full_name "ФИО"
        string passport_no UK "Документ"
        string phone "Телефон"
    }
    BOOKING["Бронирование"] {
        int booking_id PK
        datetime booked_at "Дата брони"
        int seats_count "Кол-во мест"
        decimal cargo_weight "Вес груза, кг"
    }

    AIRCRAFT_MODEL ||--o{ AIRCRAFT : "определяет"
    AIRCRAFT ||--|{ FLIGHT_CERT : "имеет"
    AIRCRAFT ||--o{ DEFECT : "имеет неисправности"
    AIRCRAFT ||--o{ FLIGHT : "выполняет"
    AIRPORT ||--o{ FLIGHT : "вылет из"
    AIRPORT ||--o{ FLIGHT : "прилёт в"
    PASSENGER ||--o{ BOOKING : "оформляет"
    FLIGHT ||--o{ BOOKING : "бронируется на"
```

## 1.3 Логическая схема (нотация Мартина)

```mermaid
erDiagram
    AIRCRAFT_MODEL["AIRCRAFT_MODEL"] {
        varchar model_name PK
        varchar aircraft_type
        int capacity
    }
    AIRCRAFT["AIRCRAFT"] {
        varchar reg_number PK
        varchar model_name FK
    }
    FLIGHT_CERT["FLIGHT_CERT"] {
        int cert_id PK
        varchar reg_number FK
        varchar cert_number UK
        date issue_date
        date valid_until
        blob scan
    }
    DEFECT["DEFECT"] {
        int defect_id PK
        varchar reg_number FK
        text description
        date detected_at
        date fixed_at
    }
    AIRPORT["AIRPORT"] {
        char iata_code PK
        varchar name
        varchar city
    }
    FLIGHT["FLIGHT"] {
        varchar flight_number PK
        varchar cargo_type
        char dep_airport FK
        char arr_airport FK
        datetime departure_dt
        datetime expected_arrival_dt
        varchar reg_number FK
    }
    PASSENGER["PASSENGER"] {
        int passenger_id PK
        varchar full_name
        varchar passport_no UK
        varchar phone
    }
    BOOKING["BOOKING"] {
        int booking_id PK
        int passenger_id FK
        varchar flight_number FK
        datetime booked_at
        int seats_count
        decimal cargo_weight
    }

    AIRCRAFT_MODEL ||--o{ AIRCRAFT : "model_name"
    AIRCRAFT ||--|{ FLIGHT_CERT : "reg_number"
    AIRCRAFT ||--o{ DEFECT : "reg_number"
    AIRCRAFT ||--o{ FLIGHT : "reg_number"
    AIRPORT ||--o{ FLIGHT : "dep_airport"
    AIRPORT ||--o{ FLIGHT : "arr_airport"
    PASSENGER ||--o{ BOOKING : "passenger_id"
    FLIGHT ||--o{ BOOKING : "flight_number"
```

## 1.4 Описание таблиц

**AIRCRAFT_MODEL** (модели самолётов)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| model_name | + | + | + | |
| aircraft_type | | + | | |
| capacity | | + | | |

*Ограничения:* `aircraft_type IN ('пассажирский','грузовой')`, `capacity > 0`.

**AIRCRAFT** (самолёты)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| reg_number | + | + | + | |
| model_name | | + | | AIRCRAFT_MODEL(model_name) |

**FLIGHT_CERT** (лётные сертификаты)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| cert_id | + | + | + | |
| reg_number | | + | | AIRCRAFT(reg_number) |
| cert_number | | + | + | |
| issue_date | | + | | |
| valid_until | | + | | |
| scan | | + | | |

*Ограничение:* `valid_until > issue_date`. Самолёт допускается к рейсу, если есть сертификат с `valid_until >= дата вылета`.

**DEFECT** (некритические неисправности)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| defect_id | + | + | + | |
| reg_number | | + | | AIRCRAFT(reg_number) |
| description | | + | | |
| detected_at | | + | | |
| fixed_at | | | | |

*`fixed_at IS NULL` — неисправность ещё ждёт ближайшего ТО.*

**AIRPORT** (аэропорты)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| iata_code | + | + | + | |
| name | | + | | |
| city | | + | | |

**FLIGHT** (рейсы)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| flight_number | + | + | + | |
| cargo_type | | + | | |
| dep_airport | | + | | AIRPORT(iata_code) |
| arr_airport | | + | | AIRPORT(iata_code) |
| departure_dt | | + | | |
| expected_arrival_dt | | + | | |
| reg_number | | + | | AIRCRAFT(reg_number) |

*Ограничения:* `dep_airport <> arr_airport`, `expected_arrival_dt > departure_dt`, `cargo_type IN ('пассажирская','грузовая')`.

**PASSENGER** (пассажиры)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| passenger_id | + | + | + | |
| full_name | | + | | |
| passport_no | | + | + | |
| phone | | | | |

**BOOKING** (бронирования)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| booking_id | + | + | + | |
| passenger_id | | + | | PASSENGER(passenger_id) |
| flight_number | | + | | FLIGHT(flight_number) |
| booked_at | | + | | |
| seats_count | | | | |
| cargo_weight | | | | |

*Ограничение:* ровно одно из `seats_count` / `cargo_weight` не NULL и > 0; какое именно — определяется типом рейса.

---

# Вариант 2. Сеть отелей

## 2.1 Допущения

- Название отеля уникально только в пределах города: уникальность составная `(city_id, name)`.
- Номер идентифицируется суррогатным `room_id`; уникальна пара `(hotel_id, room_number)`.
- Горничная закреплена за несколькими номерами (1:N); заявки на работы оставляет горничная, закреплённая за номером.
- Заявка на работы — супертип с двумя подтипами: **уборка** (тип: экспресс/стандарт/генеральная) и **ремонт** (специалист: электрик/сантехник/…). Подтип связан с супертипом 1:0..1.
- Санитарный документ — один на номер, хранится как скан в атрибуте номера.
- Цена номера зависит от пары (тип номера, город): отдельная таблица цен.
- Бронь — на один номер на период дат; пересечение периодов для одного номера запрещено.
- Контактные данные клиента: телефон и e-mail.

## 2.2 ER-диаграмма (Crow's Foot)

```mermaid
erDiagram
    CITY["Город"] {
        int city_id PK
        string name UK "Название города"
    }
    HOTEL["Отель"] {
        int hotel_id PK
        string name "Название"
        string address "Адрес"
    }
    ROOM_TYPE["Тип номера"] {
        int type_id PK
        string name UK "Стандарт/комфорт/люкс"
    }
    ROOM_PRICE["Цена номера"] {
        decimal price_per_night "Цена за ночь"
    }
    ROOM["Номер"] {
        int room_id PK
        string room_number "Номер комнаты"
        int floor "Этаж"
        blob sanitary_doc_scan "Скан сан. документа"
    }
    HOUSEKEEPER["Горничная"] {
        int housekeeper_id PK
        string full_name "ФИО"
        string phone "Телефон"
    }
    WORK_REQUEST["Заявка на работы"] {
        int request_id PK
        datetime created_at "Дата создания"
        string description "Описание"
        string status "Статус"
    }
    CLEANING["Уборка"] {
        string cleaning_type "Экспресс/стандарт/генеральная"
    }
    REPAIR["Ремонт"] {
        string specialist "Электрик/сантехник/..."
    }
    CLIENT["Клиент"] {
        int client_id PK
        string first_name "Имя"
        string last_name "Фамилия"
        string phone "Телефон"
        string email "E-mail"
    }
    BOOKING["Бронирование"] {
        int booking_id PK
        date check_in "Дата заезда"
        date check_out "Дата выезда"
    }

    CITY ||--o{ HOTEL : "расположен в"
    HOTEL ||--|{ ROOM : "содержит"
    ROOM_TYPE ||--o{ ROOM : "классифицирует"
    CITY ||--o{ ROOM_PRICE : "задаёт цену"
    ROOM_TYPE ||--o{ ROOM_PRICE : "имеет цену"
    HOUSEKEEPER ||--o{ ROOM : "закреплена за"
    ROOM ||--o{ WORK_REQUEST : "требует работ"
    HOUSEKEEPER ||--o{ WORK_REQUEST : "оставляет"
    WORK_REQUEST ||--o| CLEANING : "является"
    WORK_REQUEST ||--o| REPAIR : "является"
    CLIENT ||--o{ BOOKING : "бронирует"
    ROOM ||--o{ BOOKING : "бронируется"
```

## 2.3 Логическая схема (нотация Мартина)

```mermaid
erDiagram
    CITY["CITY"] {
        int city_id PK
        varchar name UK
    }
    HOTEL["HOTEL"] {
        int hotel_id PK
        int city_id FK
        varchar name
        varchar address
    }
    ROOM_TYPE["ROOM_TYPE"] {
        int type_id PK
        varchar name UK
    }
    ROOM_PRICE["ROOM_PRICE"] {
        int city_id PK, FK
        int type_id PK, FK
        decimal price_per_night
    }
    HOUSEKEEPER["HOUSEKEEPER"] {
        int housekeeper_id PK
        varchar full_name
        varchar phone
    }
    ROOM["ROOM"] {
        int room_id PK
        int hotel_id FK
        varchar room_number
        int floor
        int type_id FK
        int housekeeper_id FK
        blob sanitary_doc_scan
    }
    WORK_REQUEST["WORK_REQUEST"] {
        int request_id PK
        int room_id FK
        int housekeeper_id FK
        datetime created_at
        text description
        varchar status
        varchar work_kind
    }
    CLEANING["CLEANING"] {
        int request_id PK, FK
        varchar cleaning_type
    }
    REPAIR["REPAIR"] {
        int request_id PK, FK
        varchar specialist
    }
    CLIENT["CLIENT"] {
        int client_id PK
        varchar first_name
        varchar last_name
        varchar phone
        varchar email
    }
    BOOKING["BOOKING"] {
        int booking_id PK
        int client_id FK
        int room_id FK
        date check_in
        date check_out
    }

    CITY ||--o{ HOTEL : "city_id"
    HOTEL ||--|{ ROOM : "hotel_id"
    ROOM_TYPE ||--o{ ROOM : "type_id"
    CITY ||--o{ ROOM_PRICE : "city_id"
    ROOM_TYPE ||--o{ ROOM_PRICE : "type_id"
    HOUSEKEEPER ||--o{ ROOM : "housekeeper_id"
    ROOM ||--o{ WORK_REQUEST : "room_id"
    HOUSEKEEPER ||--o{ WORK_REQUEST : "housekeeper_id"
    WORK_REQUEST ||--o| CLEANING : "request_id"
    WORK_REQUEST ||--o| REPAIR : "request_id"
    CLIENT ||--o{ BOOKING : "client_id"
    ROOM ||--o{ BOOKING : "room_id"
```

## 2.4 Описание таблиц

**CITY**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| city_id | + | + | + | |
| name | | + | + | |

**HOTEL**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| hotel_id | + | + | + | |
| city_id | | + | | CITY(city_id) |
| name | | + | | |
| address | | + | | |

*Составное ограничение уникальности: `UNIQUE (city_id, name)`.*

**ROOM_TYPE**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| type_id | + | + | + | |
| name | | + | + | |

**ROOM_PRICE** (составной ПК `(city_id, type_id)`)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| city_id | + | + | | CITY(city_id) |
| type_id | + | + | | ROOM_TYPE(type_id) |
| price_per_night | | + | | |

*Ограничение:* `price_per_night > 0`.

**HOUSEKEEPER**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| housekeeper_id | + | + | + | |
| full_name | | + | | |
| phone | | | | |

**ROOM**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| room_id | + | + | + | |
| hotel_id | | + | | HOTEL(hotel_id) |
| room_number | | + | | |
| floor | | + | | |
| type_id | | + | | ROOM_TYPE(type_id) |
| housekeeper_id | | + | | HOUSEKEEPER(housekeeper_id) |
| sanitary_doc_scan | | + | | |

*Составное ограничение уникальности: `UNIQUE (hotel_id, room_number)`.*

**WORK_REQUEST**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| request_id | + | + | + | |
| room_id | | + | | ROOM(room_id) |
| housekeeper_id | | + | | HOUSEKEEPER(housekeeper_id) |
| created_at | | + | | |
| description | | | | |
| status | | + | | |
| work_kind | | + | | |

*`work_kind IN ('уборка','ремонт')` — дискриминатор подтипов.*

**CLEANING**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| request_id | + | + | + | WORK_REQUEST(request_id) |
| cleaning_type | | + | | |

*`cleaning_type IN ('экспресс','стандарт','генеральная')`.*

**REPAIR**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| request_id | + | + | + | WORK_REQUEST(request_id) |
| specialist | | + | | |

*`specialist IN ('электрик','сантехник', ...)`.*

**CLIENT**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| client_id | + | + | + | |
| first_name | | + | | |
| last_name | | + | | |
| phone | | + | | |
| email | | | | |

**BOOKING**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| booking_id | + | + | + | |
| client_id | | + | | CLIENT(client_id) |
| room_id | | + | | ROOM(room_id) |
| check_in | | + | | |
| check_out | | + | | |

*Ограничения:* `check_out > check_in`; периоды для одного `room_id` не пересекаются.

---

# Вариант 3. Галерея

## 3.1 Допущения

- Творческое имя художника уникально, поэтому является естественным первичным ключом. Вместо «возраста» хранится год рождения (возраст — вычисляемое значение, иначе устаревает).
- У художника один основной стиль (N:1 к справочнику стилей).
- Параметры, специфичные для вида произведения, хранятся по схеме «параметр вида → значение параметра произведения»: у каждого вида свой набор параметров (глубина для скульптуры, тип краски для живописи и т. д.), и новый вид добавляется без изменения схемы.
- Экспертиза подлинности — не более одной на произведение (1:0..1); реставраций — 0..N.
- Коллекции идентифицируются названием; связь произведение–коллекция M:N.
- Покупка — отдельная сущность; произведение продаётся не более одного раза. Атрибут `total_spent` покупателя хранится по условию задачи и поддерживается как сумма его покупок (триггер).
- Предпочтения покупателя (любимые художники и коллекции) — две связи M:N.

## 3.2 ER-диаграмма (Crow's Foot)

```mermaid
erDiagram
    ARTIST["Художник"] {
        string stage_name PK "Творческое имя"
        string birth_place "Место рождения"
        int birth_year "Год рождения (возраст вычисляется)"
    }
    ART_STYLE["Стиль искусства"] {
        int style_id PK
        string name UK "Название стиля"
    }
    WORK_KIND["Вид произведения"] {
        int kind_id PK
        string name UK "Живопись/скульптура/..."
    }
    KIND_PARAM["Параметр вида"] {
        int param_id PK
        string name "Глубина, тип краски..."
        string unit "Единица измерения"
    }
    ARTWORK["Произведение"] {
        int artwork_id PK
        string title "Название"
        int creation_year "Год изготовления"
        decimal price "Цена"
    }
    PARAM_VALUE["Значение параметра"] {
        string value "Значение"
    }
    EXPERTISE["Экспертиза подлинности"] {
        int expertise_id PK
        date expertise_date "Дата"
        blob conclusion_scan "Скан заключения"
    }
    RESTORATION["Реставрация"] {
        int restoration_id PK
        date restoration_date "Дата"
        string description "Описание работ"
    }
    COLLECTION["Коллекция"] {
        string name PK "Название коллекции"
    }
    CUSTOMER["Покупатель"] {
        int customer_id PK
        string full_name "Имя"
        string address "Адрес"
        decimal total_spent "Всего потрачено"
    }
    PURCHASE["Покупка"] {
        int purchase_id PK
        date purchase_date "Дата"
        decimal amount "Сумма"
    }

    ART_STYLE ||--o{ ARTIST : "характеризует"
    ARTIST ||--o{ ARTWORK : "создал"
    WORK_KIND ||--o{ ARTWORK : "классифицирует"
    WORK_KIND ||--o{ KIND_PARAM : "имеет параметры"
    ARTWORK ||--o{ PARAM_VALUE : "описывается"
    KIND_PARAM ||--o{ PARAM_VALUE : "задаёт"
    ARTWORK ||--o| EXPERTISE : "проходит"
    ARTWORK ||--o{ RESTORATION : "реставрируется"
    ARTWORK }o--o{ COLLECTION : "входит в"
    CUSTOMER ||--o{ PURCHASE : "совершает"
    ARTWORK ||--o| PURCHASE : "продаётся"
    CUSTOMER }o--o{ ARTIST : "любит"
    CUSTOMER }o--o{ COLLECTION : "любит"
```

## 3.3 Логическая схема (нотация Мартина)

```mermaid
erDiagram
    ART_STYLE["ART_STYLE"] {
        int style_id PK
        varchar name UK
    }
    ARTIST["ARTIST"] {
        varchar stage_name PK
        varchar birth_place
        int birth_year
        int style_id FK
    }
    WORK_KIND["WORK_KIND"] {
        int kind_id PK
        varchar name UK
    }
    KIND_PARAM["KIND_PARAM"] {
        int param_id PK
        int kind_id FK
        varchar name
        varchar unit
    }
    ARTWORK["ARTWORK"] {
        int artwork_id PK
        varchar title
        int creation_year
        decimal price
        int kind_id FK
        varchar artist_name FK
    }
    ARTWORK_PARAM_VALUE["ARTWORK_PARAM_VALUE"] {
        int artwork_id PK, FK
        int param_id PK, FK
        varchar value
    }
    EXPERTISE["EXPERTISE"] {
        int expertise_id PK
        int artwork_id FK, UK
        date expertise_date
        blob conclusion_scan
    }
    RESTORATION["RESTORATION"] {
        int restoration_id PK
        int artwork_id FK
        date restoration_date
        text description
    }
    COLLECTION["COLLECTION"] {
        varchar name PK
    }
    ARTWORK_COLLECTION["ARTWORK_COLLECTION"] {
        int artwork_id PK, FK
        varchar collection_name PK, FK
    }
    CUSTOMER["CUSTOMER"] {
        int customer_id PK
        varchar full_name
        varchar address
        decimal total_spent
    }
    PURCHASE["PURCHASE"] {
        int purchase_id PK
        int customer_id FK
        int artwork_id FK, UK
        date purchase_date
        decimal amount
    }
    CUSTOMER_FAV_ARTIST["CUSTOMER_FAV_ARTIST"] {
        int customer_id PK, FK
        varchar artist_name PK, FK
    }
    CUSTOMER_FAV_COLLECTION["CUSTOMER_FAV_COLLECTION"] {
        int customer_id PK, FK
        varchar collection_name PK, FK
    }

    ART_STYLE ||--o{ ARTIST : "style_id"
    ARTIST ||--o{ ARTWORK : "artist_name"
    WORK_KIND ||--o{ ARTWORK : "kind_id"
    WORK_KIND ||--o{ KIND_PARAM : "kind_id"
    ARTWORK ||--o{ ARTWORK_PARAM_VALUE : "artwork_id"
    KIND_PARAM ||--o{ ARTWORK_PARAM_VALUE : "param_id"
    ARTWORK ||--o| EXPERTISE : "artwork_id"
    ARTWORK ||--o{ RESTORATION : "artwork_id"
    ARTWORK ||--o{ ARTWORK_COLLECTION : "artwork_id"
    COLLECTION ||--o{ ARTWORK_COLLECTION : "collection_name"
    CUSTOMER ||--o{ PURCHASE : "customer_id"
    ARTWORK ||--o| PURCHASE : "artwork_id"
    CUSTOMER ||--o{ CUSTOMER_FAV_ARTIST : "customer_id"
    ARTIST ||--o{ CUSTOMER_FAV_ARTIST : "artist_name"
    CUSTOMER ||--o{ CUSTOMER_FAV_COLLECTION : "customer_id"
    COLLECTION ||--o{ CUSTOMER_FAV_COLLECTION : "collection_name"
```

## 3.4 Описание таблиц

**ART_STYLE**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| style_id | + | + | + | |
| name | | + | + | |

**ARTIST**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| stage_name | + | + | + | |
| birth_place | | + | | |
| birth_year | | + | | |
| style_id | | + | | ART_STYLE(style_id) |

**WORK_KIND**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| kind_id | + | + | + | |
| name | | + | + | |

**KIND_PARAM**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| param_id | + | + | + | |
| kind_id | | + | | WORK_KIND(kind_id) |
| name | | + | | |
| unit | | | | |

*Составное ограничение уникальности: `UNIQUE (kind_id, name)`.*

**ARTWORK**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| artwork_id | + | + | + | |
| title | | + | | |
| creation_year | | + | | |
| price | | + | | |
| kind_id | | + | | WORK_KIND(kind_id) |
| artist_name | | + | | ARTIST(stage_name) |

*Ограничение:* `price >= 0`.

**ARTWORK_PARAM_VALUE** (составной ПК `(artwork_id, param_id)`)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| artwork_id | + | + | | ARTWORK(artwork_id) |
| param_id | + | + | | KIND_PARAM(param_id) |
| value | | + | | |

*Параметр должен относиться к виду данного произведения (`KIND_PARAM.kind_id = ARTWORK.kind_id`) — контролируется триггером.*

**EXPERTISE**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| expertise_id | + | + | + | |
| artwork_id | | + | + | ARTWORK(artwork_id) |
| expertise_date | | + | | |
| conclusion_scan | | + | | |

**RESTORATION**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| restoration_id | + | + | + | |
| artwork_id | | + | | ARTWORK(artwork_id) |
| restoration_date | | + | | |
| description | | + | | |

**COLLECTION**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| name | + | + | + | |

**ARTWORK_COLLECTION** (составной ПК)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| artwork_id | + | + | | ARTWORK(artwork_id) |
| collection_name | + | + | | COLLECTION(name) |

**CUSTOMER**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| customer_id | + | + | + | |
| full_name | | + | | |
| address | | + | | |
| total_spent | | + | | |

*`total_spent DEFAULT 0`, `total_spent >= 0`; значение = сумма `PURCHASE.amount` покупателя.*

**PURCHASE**

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| purchase_id | + | + | + | |
| customer_id | | + | | CUSTOMER(customer_id) |
| artwork_id | | + | + | ARTWORK(artwork_id) |
| purchase_date | | + | | |
| amount | | + | | |

**CUSTOMER_FAV_ARTIST** (составной ПК)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| customer_id | + | + | | CUSTOMER(customer_id) |
| artist_name | + | + | | ARTIST(stage_name) |

**CUSTOMER_FAV_COLLECTION** (составной ПК)

| Атрибут | ПК | Обяз. | Уник. | Вн. ключ |
|---|---|---|---|---|
| customer_id | + | + | | CUSTOMER(customer_id) |
| collection_name | + | + | | COLLECTION(name) |