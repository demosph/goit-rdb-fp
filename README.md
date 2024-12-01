# Фінальний проєкт

### Завдання 1

Завантажте дані:

- Створіть схему `pandemic` у базі даних за допомогою SQL-команди.
- Оберіть її як схему за замовчуванням за допомогою SQL-команди.
- Імпортуйте дані за допомогою Import wizard так, як ви вже робили це у темі 3.
- Продивіться дані, щоб бути у контексті.

! 💡 Як бачите, атрибути `Entity` та `Code` постійно повторюються. Позбудьтеся цього за допомогою нормалізації даних.

```sql
CREATE SCHEMA pandemic;

USE pandemic;

-- Import Data from CSV file

SELECT * FROM infectious_cases;
SELECT COUNT(*) AS records FROM infectious_cases; -- 10521
```

### Завдання 2

Нормалізуйте таблицю `infectious_cases` до 3ї нормальної форми. Збережіть у цій же схемі дві таблиці з нормалізованими даними.

```sql
-- Create the entities table
CREATE TABLE entities (
    entity_id INT PRIMARY KEY AUTO_INCREMENT,
    entity_name VARCHAR(255) NOT NULL,
    entity_code VARCHAR(10),
    UNIQUE KEY unique_entity (entity_name, entity_code)
);

-- Add a foreign key to the infectious_cases table
ALTER TABLE infectious_cases
ADD COLUMN entity_id INT AFTER Entity,
ADD FOREIGN KEY (entity_id) REFERENCES entities(entity_id);

-- Populate the entities table with unique values
INSERT INTO entities (entity_name, entity_code)
SELECT DISTINCT Entity, Code
FROM infectious_cases
WHERE Entity IS NOT NULL;

SET SQL_SAFE_UPDATES = 0;

-- Update foreign keys in the infectious_cases table
UPDATE infectious_cases ic
JOIN entities e ON ic.Entity = e.entity_name and ic.Code = e.entity_code
SET ic.entity_id = e.entity_id;

SET SQL_SAFE_UPDATES = 1;

-- Drop old columns after successful update
ALTER TABLE infectious_cases
DROP COLUMN Entity,
DROP COLUMN Code;
```

### Завдання 3

Проаналізуйте дані:

- Для кожної унікальної комбінації `Entity` та `Code` або їх `id` порахуйте середнє, мінімальне, максимальне значення та суму для атрибута `Number_rabies`.

! 💡 Врахуйте, що атрибут Number_rabies може містити порожні значення `‘’` — вам попередньо необхідно їх відфільтрувати.

- Результат відсортуйте за порахованим середнім значенням у порядку спадання.
- Оберіть тільки 10 рядків для виведення на екран.

```sql
SELECT
    e.entity_name,
    e.entity_code,
    AVG(ic.number_rabies) AS avg_rabies,
    MIN(ic.number_rabies) AS min_rabies,
    MAX(ic.number_rabies) AS max_rabies,
    SUM(ic.number_rabies) AS sum_rabies
FROM
    infectious_cases ic
        JOIN
    entities e ON ic.entity_id = e.entity_id
WHERE
    ic.number_rabies NOT IN ('')
GROUP BY e.entity_name , e.entity_code
LIMIT 10;
```

### Завдання 4

Побудуйте колонку різниці в роках.

Для оригінальної або нормованої таблиці для колонки `Year` побудуйте з використанням вбудованих SQL-функцій:

- атрибут, що створює дату першого січня відповідного року,

! 💡 Наприклад, якщо атрибут містить значення ’1996’, то значення нового атрибута має бути ‘1996-01-01’.

- атрибут, що дорівнює поточній даті,
- атрибут, що дорівнює різниці в роках двох вищезгаданих колонок.

! 💡 Перераховувати всі інші атрибути, такі як `Number_malaria`, не потрібно.

```sql
SELECT
    e.entity_name,
    e.entity_code,
    ic.Year,
    DATE(CONCAT(ic.Year, '-01-01')) AS year_date,
    CURDATE() AS today_date,
    TIMESTAMPDIFF(YEAR,
        DATE(CONCAT(ic.Year, '-01-01')),
        CURDATE()) AS years_diff
FROM
    infectious_cases ic
        JOIN
    entities e ON ic.entity_id = e.entity_id
LIMIT 10;
```

### Завдання 5

Побудуйте власну функцію.

Створіть і використайте функцію, що будує такий же атрибут, як і в попередньому завданні: функція має приймати на вхід значення року, а повертати різницю в роках між поточною датою та датою, створеною з атрибута року (1996 рік → ‘1996-01-01’).

```sql
DROP FUNCTION IF EXISTS diff_years;

DELIMITER //

CREATE FUNCTION diff_years(input_year YEAR)
RETURNS INT
DETERMINISTIC
BEGIN
   RETURN TIMESTAMPDIFF(
       YEAR,
       DATE(CONCAT(input_year, '-01-01')),
       CURDATE()
   );
END //

DELIMITER ;

SELECT
   e.entity_name,
   e.entity_code,
   ic.Year,
   DATE(CONCAT(ic.Year, '-01-01')) as year_start_date,
   CURDATE() as today_date,
   diff_years(ic.Year) as years_diff
FROM infectious_cases ic
JOIN entities e ON ic.entity_id = e.entity_id
LIMIT 10;
```
