
# 📚 דף עזר למבחן SQL (SQLite & PostgreSQL) — בעברית
*פתוח-חומר, מרוכז, עם דוגמאות קצרות. בלי פייתון/סטרימליט.*

> **מבנה כללי של שאילתה**
```sql
SELECT <עמודות | *> 
FROM <טבלאות וצירופים>
[WHERE תנאי-שורות]
[GROUP BY עמודות-קיבוץ]
[HAVING תנאי-על-קבוצות]
[ORDER BY עמודות [ASC|DESC]]
[LIMIT N [OFFSET K]]
```

---

## 1) יצירת/מחיקת/שינוי טבלאות (DDL)
### יצירה בסיסית + טיפוסי נתונים
```sql
-- SQLite
CREATE TABLE table_name (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  amount INTEGER DEFAULT 0,
  price REAL,
  created_at TEXT
);

-- PostgreSQL
CREATE TABLE table_name (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  amount INTEGER DEFAULT 0,
  price NUMERIC(10,2),
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### מחיקה ושינוי
```sql
DROP TABLE table_name;

ALTER TABLE shopping RENAME TO shopp;
ALTER TABLE shopp RENAME TO shopping;

ALTER TABLE shopping ADD COLUMN supplier TEXT;
ALTER TABLE shopping RENAME COLUMN supplier TO vendor;
ALTER TABLE shopping ALTER COLUMN amount TYPE INTEGER;
```

### מגבלות
```sql
CREATE TABLE products (
  product_id INTEGER PRIMARY KEY,
  sku TEXT UNIQUE,
  name TEXT NOT NULL,
  price REAL CHECK (price >= 0),
  category_id INTEGER,
  FOREIGN KEY (category_id) REFERENCES category(category_id)
);

CREATE TABLE category (category_id INTEGER PRIMARY KEY, name TEXT);
CREATE TABLE products (
  product_id INTEGER PRIMARY KEY,
  name TEXT,
  category_id INTEGER,
  FOREIGN KEY (category_id) REFERENCES category(category_id) ON DELETE CASCADE
);
```

---

## 2) הכנסה/עדכון/מחיקה (DML)
```sql
INSERT INTO shopping (id, name, amount) VALUES
(1, 'Avokado', 5),
(2, 'Milk', 2);

UPDATE shopping SET name='Bisli' WHERE name='Bamba';
UPDATE shopping SET amount=1 WHERE name='Milk';

DELETE FROM shopping WHERE name='Orange';
```

---

## 3) שליפות בסיסיות + סינון
```sql
SELECT * FROM shopping;
SELECT id, name FROM shopping;

SELECT * FROM shopping WHERE amount > 5;
SELECT * FROM shopping WHERE name LIKE 'Bam%';
SELECT * FROM shopping WHERE amount BETWEEN 3 AND 5;
SELECT * FROM shopping WHERE maavar IS NULL;
SELECT * FROM shopping WHERE name IN ('Milk','Bread');
```

מיון והגבלה:
```sql
SELECT * FROM shopping ORDER BY maavar;
SELECT * FROM shopping ORDER BY maavar DESC;
SELECT * FROM shopping ORDER BY amount DESC LIMIT 3 OFFSET 2;
```

---

## 4) פונקציות אגרגטיביות, GROUP BY ו-HAVING
```sql
SELECT COUNT(*) FROM shopping;
SELECT MAX(amount) FROM shopping;
SELECT MIN(amount) FROM shopping;
SELECT AVG(amount) FROM shopping;
SELECT SUM(amount) FROM shopping;

SELECT maavar, COUNT(*) 
FROM shopping 
GROUP BY maavar;

SELECT maavar, COUNT(*) 
FROM shopping 
GROUP BY maavar
HAVING COUNT(*) > 1;
```

---

## 5) צירופים (JOINs)
```sql
SELECT s.id, s.name, s.amount, p.price
FROM shopping s
JOIN prices p ON s.id = p.id;

SELECT s.id, s.name, p.price
FROM shopping s
LEFT JOIN prices p ON s.id = p.id;
```

---

## 6) קשרים בין טבלאות
- **1:1** — `passwords(user_id UNIQUE)` ↔ `users(user_id)`  
- **1:N** — `products(category_id)` ↔ `category(category_id)`  
- **M:N** — טבלת צירוף עם PK מורכב

---

## 7) UNION
```sql
SELECT name FROM patients
UNION
SELECT name FROM nurses;

SELECT name FROM patients
UNION ALL
SELECT name FROM nurses;
```

---

## 8) VIEW
```sql
CREATE VIEW expensive AS
SELECT name, amount
FROM shopping
WHERE amount > 5;

SELECT * FROM expensive;
```

---

## 9) TRIGGER
```sql
CREATE TRIGGER update_stats_after_insert
AFTER INSERT ON grades
BEGIN
  UPDATE grade_stats
  SET grade_count = (SELECT COUNT(*) FROM grades)
  WHERE id = 1;
END;
```

---

## 10) UPSERT (PostgreSQL)
```sql
INSERT INTO books (book_id, title, author, year_published)
VALUES (2, 'Nineteen Eighty-Four', 'George Orwell', 1949)
ON CONFLICT (book_id)
DO UPDATE SET
  title = EXCLUDED.title,
  author = EXCLUDED.author,
  year_published = EXCLUDED.year_published;
```

---

## 11) טרנזאקציות
```sql
BEGIN;
UPDATE accounts SET bal = bal-100 WHERE id=1;
UPDATE accounts SET bal = bal+100 WHERE id=2;
COMMIT;

ROLLBACK;
```

---

## 12) טעויות נפוצות
- `WHERE` לפני GROUP BY, `HAVING` אחרי
- `COUNT(*)` סופר הכל, `COUNT(col)` לא סופר NULL
- `NULL` → לבדוק עם IS NULL / IS NOT NULL
- ב-JOIN → PK ↔ FK
- UNION מחייב מספר וסדר עמודות זהה
