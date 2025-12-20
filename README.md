<img width="6956" height="3258" alt="deepseek_mermaid_20251220_7e68b2" src="https://github.com/user-attachments/assets/f96011f2-8b3a-4dc2-851e-c28736c45e5a" /># Лабораторная работа №1
## Постановка задачи:
**Учет товаров на складах и их потребности на торговых точках (1 вариант)**

## Условие задачи:

Имеются товары (регистрационный номер, наименование, единица измерения, стоимость единицы), склады (номер, ФИО кладовщика) и торговые точки (наименование, адрес). Товары поступают на склады в определенном количестве и в определенную дату. Товары запрашиваются в определенную торговую точку в определенном количестве.

*Процессы:*
Регистрируются запросы на поставку товаров
Учитывается, какие товары хранятся на складах, поставляются от поставщиков и продаются в торговых точках

*Входные данные:*
Имеются товары (артикул, наименование, единица измерения, стоимость единицы), склады (номер, ФИО кладовщика) и торговые точки (наименование, адрес, телефон, почта, город). Товары поступают на склады в определенном количестве и в определенную дату. Товары запрашиваются в определенную торговую точку в определенном количестве.

*Выходные документы:*
Выдать список товаров на каждом складе, отсортированный по наименованиям товаров с подсчетом стоимости каждого товара.
Для заданной торговой точки выдать список запрашиваемых товаров с указанием их количества, упорядоченный по наименованиям товаров и по номерам складов.

## Базовые сущности:

- Товары (артикул, единица измерения, стоимость единицы, код поставщика), ключевой набор - артикул
- Склад (номер, адрес, ФИО кладовщика, код поставщика), ключевой набор - номер
- Торговая точка (наименование, адрес, телефон, почта, город), ключевой набор - наименование

## Отношения

[Запрос]-N,Required---------------------------N,Optional-[Товары]  
[Товары]-N,Required---------------------------N,Optional-[Склад]  
[Товары]-1,Required---------------------------N,Optional-[Поставщик]  
[Товары]-N,Required---------------------------N,Optional-[Торговая точка]

## Логическая модель
### Сущности
**1. Поставщик (Supplier)**

- supplier_id (PK, Primary Key) - Уникальный идентификатор поставщика

- full_name (VARCHAR) - ФИО поставщика

- email (VARCHAR) - Электронная почта

- address (VARCHAR) - Адрес поставщика

**2. Товар (Product)**

- sku (PK) - Артикул товара (уникальный идентификатор)

- name (VARCHAR) - Наименование товара

- unit (VARCHAR) - Единица измерения

- unit_price (DECIMAL) - Стоимость единицы товара

- supplier_id (FK) - Код поставщика (ссылка на Supplier)

**3. Склад (Warehouse)**

- warehouse_id (PK) - Уникальный идентификатор склада

- address (VARCHAR) - Адрес склада

- warehouse_manager (VARCHAR) - ФИО кладовщика

- supplier_id (FK) - Код поставщика (ссылка на Supplier)

**4. Торговая точка (SelectOutlet)**

- name (PK) - Наименование торговой точки

- address (VARCHAR) - Адрес

- phone (VARCHAR) - Телефон

- email (VARCHAR) - Электронная почта

- city (VARCHAR) - Город

**5. Запрос (Request)**

- request_id (PK) - Уникальный идентификатор запроса

- date (DATE) - Дата запроса

- order_cost (DECIMAL) - Стоимость заказа

*СВЯЗИ*

Поставщик (1) -----< Товар (N)

- Один поставщик может поставлять много товаров

- Один товар принадлежит только одному поставщику

Товар (N) -----< Товары на складах (M)

- Один товар может храниться на нескольких складах

- На одном складе может храниться несколько товаров

Товар (N) -----< Товары в торговых точках (M)

- Один товар может продаваться в нескольких торговых точках

- В одной торговой точке может продаваться несколько товаров

Товар (N) -----< Товары в запросах (M)

- Один товар может входить в несколько запросов

- В одном запросе может быть несколько товаров

## Физическая модель
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
## Проверка нормальных форм

1NF — Все атрибуты атомарны ✅ Нет повторяющихся групп. Каждое поле содержит одно значение (например, unit_price — число, name — строка). Промежуточные таблицы (RequestProduct, WarehouseProduct, OutletProduct) разбивают N:N связи.

2NF — Нет частичных зависимостей ✅ Все неключевые атрибуты зависят от всего первичного ключа. Например, в Product: name, unit, unit_price зависят от sku, а не от части ключа. В промежуточных таблицах — только ключи и атрибуты, связанные с ними (например, quantity зависит от пары (request_id, sku)).

3NF — Нет транзитивных зависимостей ✅ Нет ситуаций, когда A → B → C, где A — ключ, B — не ключ, C — не ключ. Пример: Product.supplier_id → Supplier.full_name — это допустимо, потому что Supplier — отдельная сущность, а supplier_id — внешний ключ. Это не транзитивная зависимость, а связь между сущностями.

BCNF — Все зависимости определяются ключами ✅ Любая нетривиальная функциональная зависимость X → Y должна иметь X как суперключ. В нашей модели: Product.sku → name, unit, unit_price, supplier_id — sku — ключ → OK. Supplier.supplier_id → full_name, email, address — ключ → OK. В промежуточных таблицах — только составные ключи → OK. Нет нарушений BCNF.

## Примеры запросов для "Выходных документов"
Подготовка тестовых данных

```sql
-- Заполняем таблицы тестовыми данными

-- Поставщики
INSERT INTO Supplier (supplier_id, full_name, email, address) VALUES
(1, 'ООО "Молочные продукты"', 'milk@mail.ru', 'г. Москва, ул. Молочная, 15'),
(2, 'ИП Пекарь', 'baker@yandex.ru', 'г. Санкт-Петербург, пр. Хлебный, 20'),
(3, 'ЗАО "Бакалея"', 'grocery@gmail.com', 'г. Казань, ул. Продовольственная, 30'),
(4, 'ООО "Напитки"', 'drinks@mail.ru', 'г. Екатеринбург, ул. Водная, 10');

-- Товары
INSERT INTO Product (sku, name, unit, unit_price, supplier_id) VALUES
(1001, 'Молоко "Простоквашино" 3,2%', 'литр', 89.50, 1),
(1002, 'Хлеб "Бородинский" 500г', 'штука', 55.00, 2),
(1003, 'Яйца куриные С0 10шт', 'упаковка', 135.00, 1),
(1004, 'Сахар-песок 1кг', 'пакет', 75.00, 3),
(1005, 'Соль поваренная "Экстра" 1кг', 'пакет', 32.00, 3),
(1006, 'Масло сливочное 82,5% 200г', 'упаковка', 185.00, 1),
(1007, 'Колбаса "Докторская" 500г', 'штука', 320.00, 4),
(1008, 'Сыр "Российский" 300г', 'упаковка', 280.00, 1);

-- Склады
INSERT INTO Warehouse (warehouse_id, address, warehouse_manager, supplier_id) VALUES
(1, 'г. Москва, складской комплекс №1, ул. Складская, 1', 'Иванов Иван Иванович', 1),
(2, 'г. Санкт-Петербург, логистический центр, пр. Складов, 25', 'Петров Петр Петрович', 2),
(3, 'г. Казань, товарный склад №3, ул. Заводская, 50', 'Сидорова Мария Сергеевна', 3),
(4, 'г. Екатеринбург, складской терминал, ул. Промышленная, 15', 'Кузнецов Алексей Владимирович', 4);

-- Товары на складах
INSERT INTO WarehouseProduct (warehouse_id, sku, quantity) VALUES
(1, 1001, 250),
(1, 1003, 120),
(1, 1006, 80),
(1, 1008, 60),
(2, 1002, 300),
(2, 1001, 150),
(2, 1007, 100),
(3, 1004, 400),
(3, 1005, 350),
(3, 1002, 200),
(4, 1006, 90),
(4, 1007, 110),
(4, 1008, 70);

-- Торговые точки
INSERT INTO SelectOutlet (name, address, phone, email, city) VALUES
('Супермаркет "Пятерочка" №101', 'г. Москва, ул. Ленина, 10', '+7-495-111-11-11', 'shop101@pyaterochka.ru', 'Москва'),
('Магазин "У дома" №205', 'г. Санкт-Петербург, пр. Победы, 25', '+7-812-222-22-22', 'shop205@udoma.ru', 'Санкт-Петербург'),
('Торговый центр "Мега" Казань', 'г. Казань, ул. Советская, 50', '+7-843-333-33-33', 'kazan@mega.ru', 'Казань'),
('Супермаркет "Магнит" №47', 'г. Екатеринбург, ул. Магнитная, 47', '+7-343-444-44-44', 'shop47@magnit.ru', 'Екатеринбург');

-- Запросы
INSERT INTO Request (request_id, date, order_cost) VALUES
(1, '2024-01-10', 15000.00),
(2, '2024-01-12', 22500.00),
(3, '2024-01-15', 18000.00),
(4, '2024-01-18', 12000.00);

-- Товары в запросах
INSERT INTO RequestProduct (request_id, sku, quantity) VALUES
(1, 1001, 100),   -- Молоко
(1, 1002, 50),    -- Хлеб
(1, 1003, 30),    -- Яйца
(2, 1004, 200),   -- Сахар
(2, 1005, 150),   -- Соль
(2, 1006, 40),    -- Масло
(3, 1007, 60),    -- Колбаса
(3, 1008, 45),    -- Сыр
(3, 1001, 80),    -- Молоко
(4, 1002, 70),    -- Хлеб
(4, 1003, 25),    -- Яйца
(4, 1004, 100);   -- Сахар
```
## Запрос 1: Для заданной торговой точки выдать список запрашиваемых товаров с указанием их количества, упорядоченный по наименованиям товаров и по номерам складов

```sql
SET @outlet_name = 'Супермаркет "Пятерочка" №101';

SELECT 
    p.name AS "Наименование товара",
    SUM(rp.quantity) AS "Запрашиваемое количество",
    p.unit AS "Единица измерения",
    w.warehouse_id AS "Номер склада",
    w.address AS "Адрес склада",
    w.warehouse_manager AS "Кладовщик",
    wp.quantity AS "Доступно на складе",
    p.unit_price AS "Цена за единицу",
    ROUND(SUM(rp.quantity) * p.unit_price, 2) AS "Общая стоимость запроса"
FROM RequestProduct rp
JOIN Request r ON rp.request_id = r.request_id
JOIN Product p ON rp.sku = p.sku
JOIN WarehouseProduct wp ON p.sku = wp.sku
JOIN Warehouse w ON wp.warehouse_id = w.warehouse_id
WHERE EXISTS (
    SELECT 1 FROM SelectOutlet so 
    WHERE so.name = @outlet_name
)
GROUP BY p.sku, p.name, p.unit, w.warehouse_id, w.address, w.warehouse_manager, wp.quantity, p.unit_price
ORDER BY p.name, w.warehouse_id;
```

## Запрос 2: Выдать список товаров на каждом складе, отсортированный по наименованиям товаров с подсчетом стоимости каждого товара
```sql
SELECT 
    w.warehouse_id AS "Номер склада",
    w.address AS "Адрес склада",
    w.warehouse_manager AS "Кладовщик",
    p.sku AS "Артикул",
    p.name AS "Наименование товара",
    p.unit AS "Единица измерения",
    wp.quantity AS "Количество",
    p.unit_price AS "Цена за единицу",
    ROUND(wp.quantity * p.unit_price, 2) AS "Общая стоимость",
    s.full_name AS "Поставщик",
    s.email AS "Email поставщика"
FROM WarehouseProduct wp
JOIN Warehouse w ON wp.warehouse_id = w.warehouse_id
JOIN Product p ON wp.sku = p.sku
JOIN Supplier s ON p.supplier_id = s.supplier_id
ORDER BY p.name, w.warehouse_id;
```

## Полученные диаграммы
### ER - диаграмма 

<img width="1223" height="490" alt="diagramm" src="https://github.com/user-attachments/assets/b2182140-58bd-4239-8221-8c32f5ce5d33" />

## Логическая модель в виде Диаграммы классов

<img width="4588" height="2835" alt="deepseek_mermaid_20251220_fe2359" src="https://github.com/user-attachments/assets/f1207f11-7d5e-4d66-b39c-01b689b36542" />

## Физическая модель БД

<img width="6956" height="3258" alt="deepseek_mermaid_20251220_7e68b2" src="https://github.com/user-attachments/assets/9f1aa2ba-a2ec-44e5-92f8-4370de7b8c57" />







