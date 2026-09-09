# MySQL и SQL

<p class="reading-time">Чтение: 14 минут</p>

Реляционная база хранит данные в таблицах, связывает строки ключами и обеспечивает ограничения целостности.

## Модель данных

```sql
CREATE TABLE orders (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT NOT NULL,
  status VARCHAR(30) NOT NULL,
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

## Индексы

Индекс ускоряет поиск и сортировку, но занимает место и замедляет запись. Составной индекс эффективен с учетом порядка колонок. Проверяйте план через `EXPLAIN`.

## Транзакции

Транзакция объединяет операции в одно целое. Для перевода средств нужны `BEGIN`, изменения обеих записей и `COMMIT`; при ошибке выполняется `ROLLBACK`.

## Нормализация

Не дублируйте независимые факты в каждой строке. Разделяйте сущности и связывайте их ключами, но не дробите модель без пользы для запросов и целостности.

!!! warning "Частая ошибка"
    Конкатенация пользовательского ввода в SQL создает SQL injection. Всегда используйте параметры подготовленного запроса.

## Мини-задача

Спроектируйте таблицы магазина: users, products, orders, order_items. Напишите запрос суммы заказа и пяти самых продаваемых товаров.

[Перейти к Laravel](laravel.md){ .md-button }
