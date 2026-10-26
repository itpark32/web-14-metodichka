# MySQL и SQL

<p class="reading-time">Чтение: 14 минут</p>

Реляционная база хранит данные в таблицах, связывает строки ключами и обеспечивает ограничения целостности.

!!! abstract "Фокус"
    - **Нужно знать:** таблица, тип, primary key, foreign key, CRUD и JOIN.
    - **Часто в работе:** группировка, агрегаты, индексы, транзакции и нормализация.
    - **Достаточно узнавать:** сложные планы выполнения и глубокую оптимизацию СУБД.

## Модель данных

```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE
);
CREATE TABLE orders (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT NOT NULL,
  status VARCHAR(30) NOT NULL,
  total DECIMAL(10, 2) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

Первичный ключ идентифицирует строку. Внешний ключ связывает таблицы. `NOT NULL`, `UNIQUE` и `CHECK` защищают инварианты ближе к данным.

## CRUD

```sql
INSERT INTO products (title, price) VALUES ('Чай', 320);

SELECT id, title, price
FROM products
WHERE price >= 300
ORDER BY price DESC
LIMIT 20;

UPDATE products SET price = 350 WHERE id = 7;
DELETE FROM products WHERE id = 7;
```

`UPDATE` и `DELETE` без `WHERE` затрагивают все строки.

## JOIN и группировка

```sql
SELECT u.name, COUNT(o.id) AS orders_count
FROM users AS u
LEFT JOIN orders AS o ON o.user_id = u.id
GROUP BY u.id, u.name
HAVING COUNT(o.id) >= 3;
```

`INNER JOIN` оставляет совпавшие строки. `LEFT JOIN` сохраняет все строки слева. `WHERE` фильтрует до группировки, `HAVING` после.

### Связный запрос по заказам

```sql
SELECT o.id, u.name, o.total
FROM orders AS o
JOIN users AS u ON u.id = o.user_id
ORDER BY o.id DESC;

SELECT user_id, COUNT(*) AS orders_count, SUM(total) AS total_spent
FROM orders
GROUP BY user_id;
```

Первый запрос объединяет каждый заказ с его покупателем. Второй возвращает одну строку на пользователя с количеством и суммой заказов; пользователь без заказов в нем отсутствует. Чтобы сохранить и таких пользователей, начните с users и LEFT JOIN, как в примере выше.

Здесь связь one-to-many: один пользователь имеет много заказов, внешний ключ хранится в orders. Связь many-to-many заказов и товаров потребует таблицы order_items с количеством и ценой на момент покупки. `VARCHAR` хранит текст, `BIGINT` — целое, `DECIMAL` — точное десятичное значение, `TIMESTAMP` — время. При передаче денег в API согласуйте единицы: копейки integer или десятичная строка, а не случайное смешение форматов.

## Подзапросы и функции

Подзапрос передает результат одного запроса другому. Часто ту же задачу можно выразить JOIN или CTE, поэтому выбирайте понятный вариант и проверяйте план.

```sql
SELECT title, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

Агрегатные функции `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` работают с набором строк. Скалярные функции преобразуют значение одной строки.

## Индексы

Индекс ускоряет поиск и сортировку, но занимает место и замедляет запись. Составной индекс эффективен с учетом порядка колонок. Проверяйте план через `EXPLAIN`.

## Транзакции

Транзакция объединяет операции в одно целое. Для перевода средств нужны `BEGIN`, изменения обеих записей и `COMMIT`; при ошибке выполняется `ROLLBACK`.

## Нормализация

Не дублируйте независимые факты в каждой строке. Разделяйте сущности и связывайте их ключами, но не дробите модель без пользы для запросов и целостности.

!!! warning "Частая ошибка"
    Конкатенация пользовательского ввода в SQL создает SQL injection. Всегда используйте параметры подготовленного запроса.

## Как ответить на интервью

> SQL описывает операции над данными, а схема задает типы, ключи и ограничения. JOIN связывает строки таблиц, GROUP BY формирует группы для агрегатов, индекс ускоряет чтение ценой места и записи, транзакция объединяет изменения по принципу все или ничего.

## Мини-задача

Спроектируйте таблицы магазина: users, products, orders, order_items. Напишите запрос суммы заказа и пяти самых продаваемых товаров.

## Проверьте себя

1. В какой таблице находится внешний ключ one-to-many?
2. Почему DECIMAL подходит денежным значениям?
3. Чем WHERE отличается от HAVING?
4. Когда LEFT JOIN сохранит строку без совпадения?
5. Почему нельзя создавать индекс на каждую колонку без оценки запросов?

[Перейти к Laravel](laravel.md){ .md-button }
