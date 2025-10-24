## SQL Analysis

Анализ выполнен в **PostgreSQL** на основе датасета [*Online Retail Dataset (Kaggle)*](https://www.kaggle.com/datasets/carrie1/ecommerce-data?resource=download).

---

## Загрузка и подготовка данных

### 1. Импорт данных в PostgreSQL

Создаём новую базу данных:

```sql
CREATE DATABASE ecommerce_analysis;
```

Подключаемся к ней:

```sql
\c ecommerce_analysis;
```

Создаём таблицу для данных:

```sql
CREATE TABLE orders (
    InvoiceNo VARCHAR,
    StockCode VARCHAR,
    Description TEXT,
    Quantity INT,
    InvoiceDate TEXT,
    UnitPrice NUMERIC,
    CustomerID INT,
    Country TEXT
);
```

---

### 2. Импорт CSV-файла

```sql
\copy orders FROM 'C:\Users\User\Downloads\data.csv' DELIMITER ',' CSV HEADER;
```

---

### 3. Преобразование даты

После импорта добавляем новый столбец с корректным форматом даты:

```sql
ALTER TABLE orders ADD COLUMN InvoiceDate_parsed TIMESTAMP;

UPDATE orders
SET InvoiceDate_parsed = TO_TIMESTAMP(InvoiceDate, 'MM/DD/YYYY HH24:MI');
```

---

### 4. Проверка корректности загрузки

```sql
SELECT * FROM orders LIMIT 5;
```

---

### Общая информация о данных

```sql
SELECT COUNT(*) AS total_rows,
       COUNT(DISTINCT InvoiceNo) AS unique_invoices,
       COUNT(DISTINCT CustomerID) AS unique_customers,
       COUNT(DISTINCT Country) AS countries
FROM orders;
```

|  total_rows | unique_invoices | unique_customers | countries |
| :---------: | :-------------: | :--------------: | :-------: |
| **541,909** |    **25,900**   |     **4,372**    |   **38**  |

В датасете почти 542 тыс. транзакций, более 25 тыс. заказов и 4 тыс. клиентов из 38 стран.

---

### Добавление выручки

```sql
ALTER TABLE orders
ADD COLUMN Revenue NUMERIC;

UPDATE orders
SET Revenue = Quantity * UnitPrice;
```

**Назначение:** создаёт поле для расчёта суммы заказа.

---

### Топ-10 стран по общей выручке

```sql
SELECT Country,
       ROUND(SUM(Revenue), 2) AS Total_Revenue
FROM orders
GROUP BY Country
ORDER BY Total_Revenue DESC
LIMIT 10;
```


| №  | Country        | Total Revenue |
| -- | -------------- | ------------- |
| 1  | United Kingdom | 8,187,806.36  |
| 2  | Netherlands    | 284,661.54    |
| 3  | EIRE           | 263,276.82    |
| 4  | Germany        | 221,698.21    |
| 5  | France         | 197,403.90    |
| 6  | Australia      | 137,077.27    |
| 7  | Switzerland    | 56,385.35     |
| 8  | Spain          | 54,774.58     |
| 9  | Belgium        | 40,910.96     |
| 10 | Sweden         | 36,595.91     |

Великобритания — основной рынок (около 90% выручки). Остальные страны дают отностительно мало вклада.

---

### Средний чек по странам

```sql
SELECT Country,
       ROUND(AVG(Revenue), 2) AS Avg_Revenue_Per_Invoice
FROM orders
GROUP BY Country
ORDER BY Avg_Revenue_Per_Invoice DESC
LIMIT 10;
```

| №  | Country              | Avg Revenue per Invoice |
| -- | -------------------- | ----------------------- |
| 1  | Netherlands          | 120.06                  |
| 2  | Australia            | 108.88                  |
| 3  | Japan                | 98.72                   |
| 4  | Sweden               | 79.21                   |
| 5  | Denmark              | 48.25                   |
| 6  | Lithuania            | 47.46                   |
| 7  | Singapore            | 39.83                   |
| 8  | Lebanon              | 37.64                   |
| 9  | Brazil               | 35.74                   |
| 10 | Hong Kong            | 35.13                   |

Из таблицы видно, что наибольший средний доход с одного заказа наблюдается в Нидерландах (120$) и Австралии (109$), это может говорить о крупных заказах или о более дорогих товарах, приобретаемых клиентами из этих стран.
Таким образом, можно предположить, что рынки Нидерландов и Австралии более прибыльны при меньшем числе заказов.

---

### Динамика продаж по месяцам

```sql
SELECT DATE_TRUNC('month', InvoiceDate) AS Month,
       ROUND(SUM(Revenue), 2) AS Total_Revenue
FROM orders
GROUP BY Month
ORDER BY Month;
```

| №  | month   | total_revenue |
| -- | ------- | ------------: |
| 1  | 2010-12 |    748,957.02 |
| 2  | 2011-01 |    560,000.26 |
| 3  | 2011-02 |    498,062.65 |
| 4  | 2011-03 |    683,267.08 |
| 5  | 2011-04 |    493,207.12 |
| 6  | 2011-05 |    723,333.51 |
| 7  | 2011-06 |    691,123.12 |
| 8  | 2011-07 |    681,300.11 |
| 9  | 2011-08 |    682,680.51 |
| 10 | 2011-09 |  1,019,687.62 |
| 11 | 2011-10 |  1,070,704.67 |
| 12 | 2011-11 |  1,461,756.25 |
| 13 | 2011-12 |    433,668.01 |

Выручка демонстрирует явно выраженный сезонный рост к концу 2011 года.
После умеренного уровня продаж в первой половине года (500–700 тыс.), начинается рост с сентября, достигающий пика в ноябре (1.46 млн) вероятно из-за предпраздничного спроса.
В декабре наблюдается резкое падение.
Таким образом, можно сделать вывод, что вторая половина года значительно прибыльнее, особенно в предновогодний сезон.

---

### Топ-10 популярных товаров

```sql
SELECT Description,
       SUM(Quantity) AS Total_Quantity
FROM orders
GROUP BY Description
ORDER BY Total_Quantity DESC
LIMIT 10;
```

| №  | description                        | total_quantity    |
| -- | ---------------------------------- | ----------------: |
| 1  | WORLD WAR 2 GLIDERS ASSTD DESIGNS  |            53,847 |
| 2  | JUMBO BAG RED RETROSPOT            |            47,363 |
| 3  | ASSORTED COLOUR BIRD ORNAMENT      |            36,381 |
| 4  | POPCORN HOLDER                     |            36,334 |
| 5  | PACK OF 72 RETROSPOT CAKE CASES    |            36,039 |
| 6  | WHITE HANGING HEART T-LIGHT HOLDER |            35,317 |
| 7  | RABBIT NIGHT LIGHT                 |            30,680 |
| 8  | MINI PAINT SET VINTAGE             |            26,437 |
| 9  | PACK OF 12 LONDON TISSUES          |            26,315 |
| 10 | PACK OF 60 PINK PAISLEY CAKE CASES |            24,753 |

Наибольшие объёмы продаж приходятся на недорогие подарочные и декоративные товары.
Особенно выделяются позиции вроде “WORLD WAR 2 GLIDERS ASSTD DESIGNS” и “JUMBO BAG RED RETROSPOT” - вероятно, лидеры по популярности в сегменте сувениров и аксессуаров для дома.
Можно предположить, что основная аудитория — покупатели небольших подарков и товаров для декора, а не крупные компании.

---

### Топ-10 покупателей по выручке

```sql
SELECT CustomerID,
       ROUND(SUM(Revenue), 2) AS Total_Spent
FROM orders
WHERE CustomerID IS NOT NULL
GROUP BY CustomerID
ORDER BY Total_Spent DESC
LIMIT 10;
```

| №  | customerid  | total_spent         |
| -- | ----------- | ------------------: |
| 1  | 14646       |          279,489.02 |
| 2  | 18102       |          256,438.49 |
| 3  | 17450       |          187,482.17 |
| 4  | 14911       |          132,572.62 |
| 5  | 12415       |          123,725.45 |
| 6  | 14156       |          113,384.14 |
| 7  | 17511       |           88,125.38 |
| 8  | 16684       |           65,892.08 |
| 9  | 13694       |           62,653.10 |
| 10 | 15311       |           59,419.34 |

Эти данные показывают, что основная доля выручки концентрируется у небольшого числа клиентов.
Клиенты с ID 14646 и 18102 ключевые для бизнеса: их заказы превышают 250 тысяяч.
Такой анализ помогает выявить наиболее ценных клиентов для удержания.

---

**Выводы:**

* Основной объём продаж приходится на **Великобританию**.
* Наблюдаются сезонные пики активности (сентябрь–ноябрь).
* Наиболее прибыльные категории — **аксессуары для дома и праздничный декор**.

---


```python

```
