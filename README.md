## Постановка задачи:
**Учет товаров на складах и их потребности на торговых точках (1 вариант)**

## Условие задачи:

Имеются товары (регистрационный номер, наименование, единица измерения, стоимость единицы), склады (номер, ФИО кладовщика) и торговые точки (наименование, адрес). Товары поступают на склады в определенном количестве и в определенную дату. Товары запрашиваются в определенную торговую точку в определенном количестве.

*Процессы:*
Регистрируются запросы на поставку товаров
Учитывается, какие товары хранятся на складах, поставляются от поставщиков и продаются в торговых точках

*Выходные документы:*
Выдать список товаров на каждом складе, отсортированный по наименованиям товаров с подсчетом стоимости каждого товара.
Для заданной торговой точки выдать список запрашиваемых товаров с указанием их количества, упорядоченный по наименованиям товаров и по номерам складов.

## Базовые сущности:

- Товары(артикул, единица измерения, стоимость единицы, код поставщика), ключевой набор - артикул
- Склад(номер, адрес, ФИО кладовщика, код поставщика), ключевой набор - номер 
- Торговая точка(наименование, адрес, телефон, почта, город), ключевой набор - наименование

## Процессы(Отношения)

[Запрос]-N,Required---------------------------N,Optional-[Товары]  
[Товары]-N,Required---------------------------N,Optional-[Склад]  
[Товары]-1,Required---------------------------N,Optional-[Поставщик]  
[Товары]-N,Required---------------------------N,Optional-[Торговая точка]

## Логическая модель

*Сущности*
**Поставщик (Supplier)**
- supplier_id (PK) — уникальный идентификатор поставщика

- full_name — ФИО поставщика

- email — электронная почта

- address — адрес

**Товар (Product)**
- sku (PK) — артикул товара

- name — наименование товара

- unit — единица измерения

- unit_price — стоимость единицы

- supplier_id (FK) → Supplier(supplier_id)

**Склад (Warehouse)**
- warehouse_id (PK) — уникальный идентификатор склада

- address — адрес склада

- warehouse_manager — ФИО кладовщика

- supplier_id (FK) → Supplier(supplier_id)

**Торговая точка (SelectOutlet)**
- name (PK) — наименование торговой точки

- address — адрес

- phone — телефон

- email — электронная почта

- city — город

**Запрос (Request)**
- request_id (PK) — уникальный идентификатор запроса

- date — дата запроса

- order_cost — стоимость заказа

*СВЯЗИ*
- Товары на складах (WarehouseProduct)

warehouse_id (FK) → Warehouse(warehouse_id)

sku (FK) → Product(sku)

quantity — количество товара на складе

Первичный ключ: (warehouse_id, sku)

- Товары в торговых точках (OutletProduct)
- 
outlet_name (FK) → SelectOutlet(name)

sku (FK) → Product(sku)

quantity — количество товара в точке

Первичный ключ: (outlet_name, sku)

- Товары в запросах (RequestProduct)
request_id (FK) → Request(request_id)

sku (FK) → Product(sku)

quantity — запрашиваемое количество

Первичный ключ: (request_id, sku)

## Лабораторная работа №1




### Физическая модель
```sql
-- Создание последовательностей для автоинкремента (если необходимо)
CREATE SEQUENCE supplier_seq START WITH 1 INCREMENT BY 1;
CREATE SEQUENCE product_seq START WITH 1 INCREMENT BY 1;
CREATE SEQUENCE warehouse_seq START WITH 1 INCREMENT BY 1;
CREATE SEQUENCE request_seq START WITH 1 INCREMENT BY 1;

-- Таблица поставщиков
CREATE TABLE Supplier (
    supplier_id INTEGER PRIMARY KEY,
    full_name VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    address VARCHAR(500),
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица товаров
CREATE TABLE Product (
    sku INTEGER PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    unit VARCHAR(50) NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL CHECK (unit_price >= 0),
    supplier_id INTEGER NOT NULL,
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (supplier_id) REFERENCES Supplier(supplier_id) ON DELETE RESTRICT
);

-- Таблица складов
CREATE TABLE Warehouse (
    warehouse_id INTEGER PRIMARY KEY,
    address VARCHAR(500) NOT NULL,
    warehouse_manager VARCHAR(255) NOT NULL,
    supplier_id INTEGER,
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (supplier_id) REFERENCES Supplier(supplier_id) ON DELETE SET NULL
);

-- Таблица торговых точек
CREATE TABLE SelectOutlet (
    name VARCHAR(255) PRIMARY KEY,
    address VARCHAR(500) NOT NULL,
    phone VARCHAR(50),
    email VARCHAR(255),
    city VARCHAR(100) NOT NULL,
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица запросов
CREATE TABLE Request (
    request_id INTEGER PRIMARY KEY,
    date DATE NOT NULL,
    order_cost DECIMAL(10,2) NOT NULL CHECK (order_cost >= 0),
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Связующая таблица: Товары на складах
CREATE TABLE WarehouseProduct (
    warehouse_id INTEGER,
    sku INTEGER,
    quantity INTEGER NOT NULL CHECK (quantity >= 0),
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (warehouse_id, sku),
    FOREIGN KEY (warehouse_id) REFERENCES Warehouse(warehouse_id) ON DELETE CASCADE,
    FOREIGN KEY (sku) REFERENCES Product(sku) ON DELETE CASCADE
);

-- Связующая таблица: Товары в торговых точках
CREATE TABLE OutletProduct (
    outlet_name VARCHAR(255),
    sku INTEGER,
    quantity INTEGER NOT NULL CHECK (quantity >= 0),
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (outlet_name, sku),
    FOREIGN KEY (outlet_name) REFERENCES SelectOutlet(name) ON DELETE CASCADE,
    FOREIGN KEY (sku) REFERENCES Product(sku) ON DELETE CASCADE
);

-- Связующая таблица: Товары в запросах
CREATE TABLE RequestProduct (
    request_id INTEGER,
    sku INTEGER,
    quantity INTEGER NOT NULL CHECK (quantity >= 0),
    PRIMARY KEY (request_id, sku),
    FOREIGN KEY (request_id) REFERENCES Request(request_id) ON DELETE CASCADE,
    FOREIGN KEY (sku) REFERENCES Product(sku) ON DELETE CASCADE
);
```
### Проверка нормальных форм

**1NF — Все атрибуты атомарны ✅**
Нет повторяющихся групп.
Каждое поле содержит одно значение (например, unit_price — число, name — строка).
Промежуточные таблицы (RequestProduct, WarehouseProduct, OutletProduct) разбивают N:N связи.

**2NF — Нет частичных зависимостей ✅**
Все неключевые атрибуты зависят от всего первичного ключа.
Например, в Product: name, unit, unit_price зависят от sku, а не от части ключа.
В промежуточных таблицах — только ключи и атрибуты, связанные с ними (например, quantity зависит от пары (request_id, sku)).

**3NF — Нет транзитивных зависимостей ✅**
Нет ситуаций, когда A → B → C, где A — ключ, B — не ключ, C — не ключ.
Пример: Product.supplier_id → Supplier.full_name — это допустимо, потому что Supplier — отдельная сущность, а supplier_id — внешний ключ. Это не транзитивная зависимость, а связь между сущностями.

**BCNF — Все зависимости определяются ключами ✅**
Любая нетривиальная функциональная зависимость X → Y должна иметь X как суперключ.
В нашей модели:
Product.sku → name, unit, unit_price, supplier_id — sku — ключ → OK.
Supplier.supplier_id → full_name, email, address — ключ → OK.
В промежуточных таблицах — только составные ключи → OK.
**Нет нарушений BCNF.**


### ER - диаграмма 

<img width="1223" height="490" alt="diagramm" src="https://github.com/user-attachments/assets/b2182140-58bd-4239-8221-8c32f5ce5d33" />








