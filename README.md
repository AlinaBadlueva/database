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
-- Создание последовательностей для автоинкремента
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

Нормальная форма	 Выполнено	      Комментарий
1NF	                   ✅	          Все атрибуты атомарны, нет повторяющихся групп

2NF	                   ✅	          Все неключевые атрибуты полностью зависят от первичного ключа

3NF	                   ✅	          Нет транзитивных зависимостей между неключевыми атрибутами

BCNF	               ✅	          Все детерминанты являются потенциальными ключами



### ER - диаграмма 

<img width="1223" height="490" alt="diagramm" src="https://github.com/user-attachments/assets/b2182140-58bd-4239-8221-8c32f5ce5d33" />








