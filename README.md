# Лабораторная работа №1
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

# Лабораторная работа №2
## Заполнение таблиц:

<img width="1165" height="766" alt="2025-12-20_20-11-28" src="https://github.com/user-attachments/assets/020e73d3-1ba5-4632-8ce8-b79806f9d054" />

<img width="1099" height="836" alt="2025-12-20_20-12-14" src="https://github.com/user-attachments/assets/6eb33bc9-8f63-43e5-bf19-0fab1cd880f9" />

<img width="1404" height="665" alt="2025-12-20_20-12-59" src="https://github.com/user-attachments/assets/722e80d4-a45a-4e65-80e4-26e39b0117e0" />

<img width="952" height="855" alt="2025-12-20_20-13-41" src="https://github.com/user-attachments/assets/07cbc41d-8ee1-4d9d-8401-ac23eb0c93de" />

<img width="978" height="755" alt="2025-12-20_20-14-00" src="https://github.com/user-attachments/assets/162cad6a-2a98-4450-98e0-7d61f02f0b5e" />

<img width="831" height="839" alt="2025-12-20_20-14-24" src="https://github.com/user-attachments/assets/743537f7-9dcc-44e1-ab51-c19b132af0e5" />

## Выходные документы. 1-ый запрос(по заданию). Выдать список товаров на каждом складе, отсортированный по наименованиям товаров с подсчетом стоимости каждого товара

<img width="1573" height="899" alt="2025-12-20_20-19-53" src="https://github.com/user-attachments/assets/7a6d29a5-2a6d-4d0d-aa69-5d609311067f" />

<img width="1524" height="868" alt="2025-12-20_20-20-04" src="https://github.com/user-attachments/assets/e6c373d1-cee1-4629-be35-40626d37e228" />

## Выходные документы. 2-ой запрос(по заданию).  Для заданной торговой точки выдать список запрашиваемых товаров с указанием их количества, упорядоченный по наименованиям товаров и по номерам складов

<img width="1463" height="747" alt="2025-12-20_20-29-37" src="https://github.com/user-attachments/assets/37868d4f-9b76-4b6b-8682-73570fa29c7f" />

## 3-ий запрос. Анализ поставщиков.

<img width="936" height="643" alt="2025-12-20_20-41-07" src="https://github.com/user-attachments/assets/291b09ae-9c8e-4861-ab85-e091f6e6c0d8" />

## 4-ый запрос. Активность торговых точек

<img width="933" height="755" alt="2025-12-20_20-42-43" src="https://github.com/user-attachments/assets/30cac320-7b63-4b68-b78a-17143d85ac8d" />

# Лабораторная работа №3
## Цель: Освоение механизмов абстракции данных и программных модулей.
*Задачи:*
- Создание представлений для выходных документов
  
- Разработка хранимых процедур с параметрами
  
- Оптимизация запросов через представления

## Создание представлений
## Представление для Выходного документа 1. Список товаров на каждом складе с подсчетом стоимости.
<img width="745" height="585" alt="2025-12-20_21-00-30" src="https://github.com/user-attachments/assets/76200e44-bda7-42ef-bb52-a8d1be6b771a" />
<img width="1557" height="372" alt="2025-12-20_21-01-46" src="https://github.com/user-attachments/assets/9138511c-61d5-4b3f-b5db-d6bba7195bd5" />

## Представление для Выходного документа 2. Запросы для торговой точки
<img width="907" height="748" alt="2025-12-20_21-07-35" src="https://github.com/user-attachments/assets/cea8ec31-ca6f-4363-af5e-3d9a0c36baeb" />
<img width="1559" height="314" alt="2025-12-20_21-08-24" src="https://github.com/user-attachments/assets/0c62c026-4dd1-499a-8714-3000ef9cad1d" />

## Представление: Статистика поставщиков
<img width="813" height="490" alt="2025-12-20_21-19-00" src="https://github.com/user-attachments/assets/74c10b5f-892d-4365-a628-e0208c97e6f6" />
<img width="1485" height="248" alt="2025-12-20_21-21-39" src="https://github.com/user-attachments/assets/9113cfaa-2bf0-48f2-bb95-e95fa9f54b5c" />

## Представление: Ежедневные остатки
<img width="1090" height="672" alt="2025-12-20_21-24-31" src="https://github.com/user-attachments/assets/544f8deb-3f94-4fbc-acd1-08f063e0f19d" />

## Представление: Топ товаров по спросу
<img width="1359" height="820" alt="2025-12-20_21-30-39" src="https://github.com/user-attachments/assets/0081e583-6fc6-45c8-9940-0385b5c54d70" />

## Создание хранимых процедур
### Процедура для получения статистики за период
<img width="651" height="412" alt="2025-12-20_22-15-41" src="https://github.com/user-attachments/assets/e291a8d5-db3d-4425-984b-cefb1c90c227" />
<img width="679" height="585" alt="2025-12-20_22-15-53" src="https://github.com/user-attachments/assets/adcb7530-5eb8-4a7a-a643-57fe05b420cf" />

### Процедура добавления нового запроса
<img width="633" height="545" alt="2025-12-20_22-18-05" src="https://github.com/user-attachments/assets/4ba6e4a3-90c3-42b6-bb83-60e384e70326" />

### Процедура добавления товара на склад
<img width="616" height="513" alt="2025-12-20_22-19-44" src="https://github.com/user-attachments/assets/5fb346dc-84e0-470b-abef-25e3cee632ac" />

## Оптимизация запросов через представления
<img width="1034" height="670" alt="2025-12-20_22-11-19" src="https://github.com/user-attachments/assets/ef5a502c-b8a9-4d01-be22-4222c51ae177" />

# Лабораторная работа №4
## Создание генератора данных (20 000 записей в каждой таблице)
### Генератор складов
<img width="871" height="547" alt="2025-12-21_16-55-02" src="https://github.com/user-attachments/assets/37925bf4-ae4d-4c2d-ba57-30dc30a218ba" />
<img width="579" height="234" alt="2025-12-21_16-56-26" src="https://github.com/user-attachments/assets/f74c6ccd-5317-4126-a2ba-6d5b3da56f48" />

### Генератор торговых точек
<img width="804" height="653" alt="2025-12-21_17-33-27" src="https://github.com/user-attachments/assets/7ea1bdd9-8784-4c6c-aa00-5ce4b357237c" />
<img width="555" height="345" alt="2025-12-21_17-34-53" src="https://github.com/user-attachments/assets/a4690912-649a-42b4-8b87-43b1b86a8f2a" />

### Генератор запросов
<img width="795" height="574" alt="2025-12-21_17-42-16" src="https://github.com/user-attachments/assets/1adfadfc-4e3c-4964-9947-8332c09ce22c" />
<img width="762" height="413" alt="2025-12-21_17-45-43" src="https://github.com/user-attachments/assets/0f16056f-adf9-4fa0-9896-2434b49f1a7a" />

### Генератор поставщиков
<img width="903" height="621" alt="2025-12-21_16-50-23" src="https://github.com/user-attachments/assets/b0bef99b-e12b-486a-8eae-d1b9e3b4e8e4" />
<img width="616" height="144" alt="2025-12-21_16-51-48" src="https://github.com/user-attachments/assets/b32d89cf-b133-4169-9ee7-d00f992761a5" />

### Генератор товаров
<img width="957" height="520" alt="2025-12-21_17-15-39" src="https://github.com/user-attachments/assets/19c67416-5b68-4aaa-a76a-5c1f09b237a2" />
<img width="531" height="319" alt="2025-12-21_17-16-02" src="https://github.com/user-attachments/assets/bb84f348-6a6e-4ddf-bda4-65390d441d6f" />

### Генератор связей товаров со складами
<img width="882" height="578" alt="2025-12-21_17-20-00" src="https://github.com/user-attachments/assets/0891a1d0-1de6-4a93-b536-7908b84003a0" />
<img width="724" height="558" alt="2025-12-21_17-20-13" src="https://github.com/user-attachments/assets/8273574f-970b-4f99-a251-4471cc9952c0" />


### Генератор связей товаров с запросами
<img width="847" height="556" alt="2025-12-21_17-48-53" src="https://github.com/user-attachments/assets/f87efa10-3624-4ac5-a252-e058187094f5" />
<img width="654" height="406" alt="2025-12-21_17-49-09" src="https://github.com/user-attachments/assets/2a89006e-8884-4bf8-b248-746b40adc4de" />

## Оптимизация БД через индексы
## Анализ
<img width="776" height="541" alt="2025-12-21_19-06-29" src="https://github.com/user-attachments/assets/d3934a07-13a1-4f07-bd35-bb8b5b04a3f0" />

## Без индекса
<img width="978" height="694" alt="2025-12-21_19-06-47" src="https://github.com/user-attachments/assets/4c689c13-8588-4300-b0dc-3c11696d79b0" />

## Создаем индекс
<img width="757" height="407" alt="2025-12-21_19-07-12" src="https://github.com/user-attachments/assets/cb6dd211-4cb5-4802-b9c1-21f53b97ec12" />

## С индексом
<img width="788" height="481" alt="2025-12-21_19-07-34" src="https://github.com/user-attachments/assets/318bcb9e-d25a-4a94-ac37-70f4e8feb3ec" />

С индексами производительность быстрее

# Лабораторная работа №5
### ТРИГГЕР КАСКАДНОГО УДАЛЕНИЯ ТОВАРОВ НА СКЛАДАХ ПРИ УДАЛЕНИИ ТОВАРА
```sql
CREATE OR REPLACE FUNCTION badlueva_2262.delete_warehouse_product_cascade_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    RAISE NOTICE 'Триггер запущен! Удаляем товары на складах для товара SKU %', OLD.sku;
    DELETE FROM badlueva_2262.WarehouseProduct WHERE sku = OLD.sku;
    RETURN OLD;
END;
$$;

CREATE OR REPLACE TRIGGER delete_warehouse_product_cascade_trigger
    BEFORE DELETE ON badlueva_2262.Product
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.delete_warehouse_product_cascade_function();
```
### ТРИГГЕР КАСКАДНОГО УДАЛЕНИЯ ТОВАРОВ В ЗАПРОСАХ ПРИ УДАЛЕНИИ ТОВАРА
```sql
CREATE OR REPLACE FUNCTION badlueva_2262.delete_request_product_cascade_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    RAISE NOTICE 'Триггер запущен! Удаляем товары в запросах для товара SKU %', OLD.sku;
    DELETE FROM badlueva_2262.RequestProduct WHERE sku = OLD.sku;
    RETURN OLD;
END;
$$;

CREATE OR REPLACE TRIGGER delete_request_product_cascade_trigger
    BEFORE DELETE ON badlueva_2262.Product
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.delete_request_product_cascade_function();
```
### ТРИГГЕР КАСКАДНОГО УДАЛЕНИЯ ТОВАРОВ ПРИ УДАЛЕНИИ ПОСТАВЩИКА
```sql
CREATE OR REPLACE FUNCTION badlueva_2262.delete_product_cascade_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    RAISE NOTICE 'Триггер запущен! Удаляем товары поставщика ID %', OLD.supplier_id;
    DELETE FROM badlueva_2262.Product WHERE supplier_id = OLD.supplier_id;
    RETURN OLD;
END;
$$;

CREATE OR REPLACE TRIGGER delete_product_cascade_trigger
    BEFORE DELETE ON badlueva_2262.Supplier
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.delete_product_cascade_function();
```
### ТРИГГЕР КАСКАДНОГО УДАЛЕНИЯ ТОВАРОВ НА СКЛАДЕ ПРИ УДАЛЕНИИ СКЛАДА
```sql
CREATE OR REPLACE FUNCTION badlueva_2262.delete_warehouse_products_cascade_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    RAISE NOTICE 'Триггер запущен! Удаляем товары со склада ID %', OLD.warehouse_id;
    DELETE FROM badlueva_2262.WarehouseProduct WHERE warehouse_id = OLD.warehouse_id;
    RETURN OLD;
END;
$$;

CREATE OR REPLACE TRIGGER delete_warehouse_products_cascade_trigger
    BEFORE DELETE ON badlueva_2262.Warehouse
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.delete_warehouse_products_cascade_function();
```
### ТРИГГЕР КАСКАДНОГО УДАЛЕНИЯ ТОВАРОВ В ЗАПРОСАХ ПРИ УДАЛЕНИИ ЗАПРОСА
```sql
CREATE OR REPLACE FUNCTION badlueva_2262.delete_request_products_cascade_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    RAISE NOTICE 'Триггер запущен! Удаляем товары из запроса ID %', OLD.request_id;
    DELETE FROM badlueva_2262.RequestProduct WHERE request_id = OLD.request_id;
    RETURN OLD;
END;
$$;

CREATE OR REPLACE TRIGGER delete_request_products_cascade_trigger
    BEFORE DELETE ON badlueva_2262.Request
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.delete_request_products_cascade_function();
```
### ТРИГГЕР КАСКАДНОГО УДАЛЕНИЯ ЗАПРОСОВ ПРИ УДАЛЕНИИ ТОРГОВОЙ ТОЧКИ
```sql
CREATE OR REPLACE FUNCTION badlueva_2262.delete_requests_cascade_function()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    RAISE NOTICE 'Триггер запущен! Удаляем запросы для торговой точки %', OLD.name;
    DELETE FROM badlueva_2262.Request WHERE outlet_name = OLD.name;
    RETURN OLD;
END;
$$;

CREATE OR REPLACE TRIGGER delete_requests_cascade_trigger
    BEFORE DELETE ON badlueva_2262.SelectOutlet
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.delete_requests_cascade_function();
```
### ТРИГГЕР АУДИТА ИЗМЕНЕНИЙ ДЛЯ ВСЕХ ТАБЛИЦ
```sql
CREATE OR REPLACE FUNCTION badlueva_2262.триггер_аудит_изменений()
RETURNS TRIGGER AS $$
BEGIN
    IF (TG_OP = 'DELETE') THEN
        INSERT INTO badlueva_2262."Журнал_Изменений" (
            имя_таблицы, 
            тип_операции, 
            id_объекта, 
            данные_до
        )
        VALUES (
            TG_TABLE_NAME, 
            'DELETE', 
            CASE 
                WHEN TG_TABLE_NAME = 'SelectOutlet' THEN OLD.name::INTEGER
                ELSE OLD.id 
            END,
            row_to_json(OLD)::JSONB
        );
        RETURN OLD;
        ELSIF (TG_OP = 'UPDATE') THEN
        INSERT INTO badlueva_2262."Журнал_Изменений" (
            имя_таблицы, 
            тип_операции, 
            id_объекта, 
            данные_до, 
            данные_после
        )
        VALUES (
            TG_TABLE_NAME, 
            'UPDATE', 
            CASE 
                WHEN TG_TABLE_NAME = 'SelectOutlet' THEN NEW.name::INTEGER
                ELSE NEW.id 
            END,
            row_to_json(OLD)::JSONB, 
            row_to_json(NEW)::JSONB
        );
        RETURN NEW;
        ELSIF (TG_OP = 'INSERT') THEN
        INSERT INTO badlueva_2262."Журнал_Изменений" (
            имя_таблицы, 
            тип_операции, 
            id_объекта, 
            данные_после
        )
        VALUES (
            TG_TABLE_NAME, 
            'INSERT', 
            CASE 
                WHEN TG_TABLE_NAME = 'SelectOutlet' THEN NEW.name::INTEGER
                ELSE NEW.id 
            END,
            row_to_json(NEW)::JSONB
        );
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```
### СОЗДАНИЕ ТРИГГЕРОВ АУДИТА ДЛЯ КАЖДОЙ ТАБЛИЦЫ

*Аудит для таблицы Product*
```sql
DROP TRIGGER IF EXISTS audit_product_trigger ON badlueva_2262.Product;
CREATE TRIGGER audit_product_trigger
    AFTER INSERT OR UPDATE OR DELETE ON badlueva_2262.Product
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.триггер_аудит_изменений();
```
*Аудит для таблицы Supplier*
```sql
DROP TRIGGER IF EXISTS audit_supplier_trigger ON badlueva_2262.Supplier;
CREATE TRIGGER audit_supplier_trigger
    AFTER INSERT OR UPDATE OR DELETE ON badlueva_2262.Supplier
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.триггер_аудит_изменений();
```
*Аудит для таблицы Warehouse*
```sql
DROP TRIGGER IF EXISTS audit_warehouse_trigger ON badlueva_2262.Warehouse;
CREATE TRIGGER audit_warehouse_trigger
    AFTER INSERT OR UPDATE OR DELETE ON badlueva_2262.Warehouse
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.триггер_аудит_изменений();
```
*Аудит для таблицы SelectOutlet*
```sql
DROP TRIGGER IF EXISTS audit_outlet_trigger ON badlueva_2262.SelectOutlet;
CREATE TRIGGER audit_outlet_trigger
    AFTER INSERT OR UPDATE OR DELETE ON badlueva_2262.SelectOutlet
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.триггер_аудит_изменений();
```
*Аудит для таблицы Request*
```sql
DROP TRIGGER IF EXISTS audit_request_trigger ON badlueva_2262.Request;
CREATE TRIGGER audit_request_trigger
    AFTER INSERT OR UPDATE OR DELETE ON badlueva_2262.Request
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.триггер_аудит_изменений();
```
*Аудит для таблицы WarehouseProduct*
```sql
DROP TRIGGER IF EXISTS audit_warehouse_product_trigger ON badlueva_2262.WarehouseProduct;
CREATE TRIGGER audit_warehouse_product_trigger
    AFTER INSERT OR UPDATE OR DELETE ON badlueva_2262.WarehouseProduct
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.триггер_аудит_изменений();
```
*Аудит для таблицы RequestProduct*
```sql
DROP TRIGGER IF EXISTS audit_request_product_trigger ON badlueva_2262.RequestProduct;
CREATE TRIGGER audit_request_product_trigger
    AFTER INSERT OR UPDATE OR DELETE ON badlueva_2262.RequestProduct
    FOR EACH ROW
    EXECUTE FUNCTION badlueva_2262.триггер_аудит_изменений();

### ДОПОЛНИТЕЛЬНЫЕ ФУНКЦИИ ДЛЯ РАБОТЫ С ЖУРНАЛОМ ИЗМЕНЕНИЙ
```
*Функция для получения истории изменений по таблице*
```sql
CREATE OR REPLACE FUNCTION badlueva_2262.получить_историю_изменений(
    p_таблица TEXT DEFAULT NULL,
    p_id_объекта INTEGER DEFAULT NULL,
    p_тип_операции TEXT DEFAULT NULL
)
RETURNS TABLE (
    id_samvcu BIGINT,
    имя_таблицы TEXT,
    тип_операции TEXT,
    id_объекта INTEGER,
    данные_до JSONB,
    данные_после JSONB,
    пользователь TEXT,
    время_изменения TIMESTAMP
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        ж.id_samvcu,
        ж.имя_таблицы,
        ж.тип_операции,
        ж.id_объекта,
        ж.данные_до,
        ж.данные_после,
        ж.пользователь,
        ж.время_изменения
    FROM badlueva_2262."Журнал_Изменений" ж
    WHERE (p_таблица IS NULL OR ж.имя_таблицы = p_таблица)
        AND (p_id_объекта IS NULL OR ж.id_объекта = p_id_объекта)
        AND (p_тип_операции IS NULL OR ж.тип_операции = p_тип_операции)
    ORDER BY ж.время_изменения DESC;
END;
$$ LANGUAGE plpgsql;
```
*Функция для получения последних изменений*
```sql
CREATE OR REPLACE FUNCTION badlueva_2262.последние_изменения(p_лимит INTEGER DEFAULT 10)
RETURNS TABLE (
    id_samvcu BIGINT,
    таблица TEXT,
    операция TEXT,
    id_объекта INTEGER,
    пользователь TEXT,
    время TIMESTAMP,
    изменения JSONB
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        ж.id_samvcu,
        ж.имя_таблицы as таблица,
        ж.тип_операции as операция,
        ж.id_объекта,
        ж.пользователь,
        ж.время_изменения as время,
        COALESCE(ж.данные_после, ж.данные_до) as изменения
    FROM badlueva_2262."Журнал_Изменений" ж
    ORDER BY ж.время_изменения DESC
    LIMIT p_лимит;
END;
$$ LANGUAGE plpgsql;
```
*Функция для очистки старых записей журнала*
```sql
CREATE OR REPLACE FUNCTION badlueva_2262.очистить_журнал_старше_дней(p_дней INTEGER DEFAULT 90)
RETURNS INTEGER AS $$
DECLARE
    v_удалено INTEGER;
BEGIN
    DELETE FROM badlueva_2262."Журнал_Изменений" 
    WHERE время_изменения < CURRENT_TIMESTAMP - (p_дней || ' days')::INTERVAL;
    GET DIAGNOSTICS v_удалено = ROW_COUNT;
    RETURN v_удалено;
END;
$$ LANGUAGE plpgsql;
```
### ТЕСТИРОВАНИЕ ТРИГГЕРОВ И АУДИТА
```sql
SELECT 'Записей в журнале до тестов: ' || COUNT(*)::TEXT 
FROM badlueva_2262."Журнал_Изменений";
```
*Тест INSERT (создаем тестовую запись)*
```sql
DO $$
DECLARE
    v_test_sku INTEGER;
BEGIN
    RAISE NOTICE '=== ТЕСТ INSERT ===';
    INSERT INTO badlueva_2262.Product (sku, name, unit, unit_price, supplier_id)
    VALUES (99999, 'Тестовый товар для аудита', 'шт', 150.50, 1)
    RETURNING sku INTO v_test_sku;
    RAISE NOTICE 'Создан тестовый товар SKU: %', v_test_sku;
    RAISE NOTICE 'Проверка журнала после INSERT:';
END;
$$;
```
*Тест UPDATE (изменяем тестовую запись)*
```sql
DO $$
DECLARE
    v_test_sku INTEGER := 99999;
BEGIN
    RAISE NOTICE '=== ТЕСТ UPDATE ===';
    UPDATE badlueva_2262.Product 
    SET unit_price = 200.00, name = 'Тестовый товар (обновлен)'
    WHERE sku = v_test_sku;
    RAISE NOTICE 'Обновлен товар SKU: %', v_test_sku;
    RAISE NOTICE 'Проверка журнала после UPDATE:';
END;
$$;
```
*Тест DELETE (удаляем тестовую запись)*
```sql
DO $$
DECLARE
    v_test_sku INTEGER := 99999;
    v_count_before INTEGER;
    v_count_after INTEGER;
BEGIN
    RAISE NOTICE '=== ТЕСТ DELETE (с каскадом) ===';
    SELECT COUNT(*) INTO v_count_before 
    FROM badlueva_2262.WarehouseProduct 
    WHERE sku = v_test_sku;
    RAISE NOTICE 'Товаров на складах до удаления: %', v_count_before;
    DELETE FROM badlueva_2262.Product WHERE sku = v_test_sku;
    RAISE NOTICE 'Удален товар SKU: %', v_test_sku;
    SELECT COUNT(*) INTO v_count_after 
    FROM badlueva_2262.WarehouseProduct 
    WHERE sku = v_test_sku;
    RAISE NOTICE 'Товаров на складах после удаления: %', v_count_after;
END;
$$;
```
*Проверка журнала изменений после тестов*
```sql
SELECT 'Записей в журнале после тестов: ' || COUNT(*)::TEXT 
FROM badlueva_2262."Журнал_Изменений";
```
### ПРИМЕРЫ ИСПОЛЬЗОВАНИЯ ФУНКЦИЙ
```sql
SELECT * FROM badlueva_2262.получить_историю_изменений('Product', 12345);
```








