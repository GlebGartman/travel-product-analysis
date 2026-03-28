# 📊 Travel Product Analysis (Sputnik8)

## Описание проекта
Анализ пользовательского поведения на travel-платформе Sputnik8 с фокусом на взаимодействие с фильтрами и продуктовыми метриками.

В рамках проекта:

- рассчитана доля пользователей, использующих фильтры  
- проанализировано использование фильтров по городам (top/bottom сегменты)  
- исследована структура взаимодействия с интерфейсом (filters, sorting, categories)  
- оценена популярность ценовых фильтров  
- рассчитаны DAU, MAU и средний DAU  
- проведён базовый анализ конверсии  

Цель — выявление узких мест в пользовательском пути и точек роста продукта

## 🛠 Реализация

Код проекта представлен в следующих файлах:

- `travel_city.ipynb` — анализ пользовательского поведения и использования фильтров  
- `travel_orders.ipynb` — анализ заказов и продуктовых метрик (DAU, MAU, конверсия)

## 📂 Описание данных

Датасет содержит информацию о пользовательских действиях на платформе и включает следующие поля:

- `event_category` — категория события  
- `event_action` — тип действия пользователя  
- `event_label` — дополнительная информация о событии (например, город или фильтр)  
- `total_events` — общее количество событий  
- `unique_events` — количество уникальных пользователей, совершивших действие
-  `order_id` — уникальный идентификатор заказа  
- `user_id` — идентификатор пользователя  
- `order_date` — дата оформления заказа  
- `product_id` — идентификатор продукта (экскурсии)  
- `category` — категория продукта  
- `price` — стоимость продукта  
- `quantity` — количество единиц в заказе  
- `region` — регион пользователя или заказа  

## 🗄 Работа с базой данных

Для анализа используется `sqlite3` (in-memory база данных).

```python
import sqlite3
import pandas as pd

conn = sqlite3.connect(":memory:")
df.to_sql("travels", conn, index=False, if_exists="replace")
```

![Города разведка](https://drive.google.com/uc?export=view&id=11e5RdNIKY4D4y7WtBuglWLL1X5zhu7c_)
![Заказы разведка](https://drive.google.com/uc?export=view&id=1wJlRjvmOUvvRt_W5UapoUkvi0D-onr_C)


## 🔎 Доля пользователей, использующих фильтры

```python
filters = """
WITH cities as (
SELECT event_category,
       event_action,
       UPPER(CASE 
            WHEN INSTR(event_label, '/') > 0
            THEN TRIM(SUBSTR(event_label, 1, INSTR(event_label, '/') - 1))
            ELSE TRIM(event_label) END) AS city,	
       total_events,
       unique_events
FROM travels
),

kolvo as (
SELECT SUM(unique_events) FILTER(WHERE event_action = 'Page Visit') as unique_pages_visit,
       SUM(unique_events) FILTER(WHERE event_action != 'Page Visit') as unique_filters_visit
FROM cities
)

SELECT *, ROUND(100.0 * unique_filters_visit / unique_pages_visit, 2) as filter_percent 
FROM kolvo
"""

result1 = pd.read_sql(filters, conn)
display(result1)
```
![Доля пользователей](https://drive.google.com/uc?export=view&id=1sP9NB2m-GWGEVHyHTtRpdGkxNS3f6-aM)

## 🌍 Использование фильтров по городам

```python
cities = """
WITH cities as (
SELECT event_category,
       event_action,
       UPPER(CASE 
            WHEN INSTR(event_label, '/') > 0
            THEN TRIM(SUBSTR(event_label, 1, INSTR(event_label, '/') - 1))
            ELSE TRIM(event_label) END) AS city,	
       total_events,
       unique_events
FROM travels
),
city_kolvo AS (
SELECT city,
       SUM(CASE WHEN event_action = 'Page Visit' THEN unique_events ELSE 0 END) AS page_users,
       SUM(CASE WHEN event_action != 'Page Visit' THEN unique_events ELSE 0 END) AS filter_users
FROM cities
GROUP BY 1
)

SELECT *, ROUND(CAST(filter_users AS FLOAT) / page_users, 2) AS filter_rate 
FROM city_kolvo
WHERE page_users > 0
"""

result2 = pd.read_sql(cities, conn).set_index('city')

# Топ 10 по использованию фильтров
top_cities = result2.sort_values('filter_rate', ascending=False).head(10)


# Топ 10 с наименьшим использованием
bottom_cities = result2.sort_values('filter_rate').head(10)

print('Топ 10 городов по использованию фильтров')
display(top_cities)

print()

print('Топ 10 городов с наименьшим использованием фильтров')
display(bottom_cities)
```
### 🔝 Топ 10 городов по использованию фильтров
![Топ города](https://drive.google.com/uc?export=view&id=1qnBzhjpi-EjWndYw4Ep7NpylFcXJx_1d)

---

### 🔻 Топ 10 городов с наименьшим использованием фильтров
![Аутсайдеры города](https://drive.google.com/uc?export=view&id=1w177ira2UjBhvWIdDX4bb48e617ARtMg)

## 🧩 Анализ взаимодействия с элементами интерфейса

```python
section = """
WITH base as (
SELECT event_action, unique_events,
CASE 
WHEN event_action IN (
    'ticket-type_checkbox',
    'pay-type_checkbox',
    'dates_filter_mobile',
    'clear_filter_mobile',
    'start_date_click',
    'end_date_click'
) THEN 'filters'   
WHEN event_action = 'filters-categories_click' THEN 'sorting'
WHEN event_action = 'search-tools-button_open' THEN 'categories'
ELSE 'other'
END AS section
FROM travels
)

SELECT section, SUM(unique_events) AS unique_events 
FROM base
WHERE event_action != 'Page Visit'
GROUP BY 1
ORDER BY 2 DESC
"""

result3 = pd.read_sql(section, conn).set_index('section')
display(result3)
```
![Наименьшие города](https://drive.google.com/uc?export=view&id=1w177ira2UjBhvWIdDX4bb48e617ARtMg)

## 💰 Использование ценовых фильтров

```python
price = """

WITH price_kolvo as (
SELECT SUM(CASE WHEN event_action IN (
    'price_first',
    'price_second',
    'price_third',
    'price_button_submit'
) THEN unique_events ELSE 0 END) AS price_usage,
SUM(unique_events) FILTER(WHERE event_action != 'Page Visit') as unique_filters_visit
FROM travels
)

SELECT price_usage, 
       ROUND(100.0 * price_usage / unique_filters_visit, 2) AS price_percent 
FROM price_kolvo 

"""

result4 = pd.read_sql(price, conn)
display(result4)
```
![Ценовой фильтр](https://drive.google.com/uc?export=view&id=1P8Sxef3Dhz2rw8rn88lAYlFgv8K0U2HE)

## 🧪 A/B-тестирование фильтра по времени начала экскурсии

**Главная цель** внедрения фильтра по времени начала (утро / день / вечер) —  упростить и ускорить поиск подходящей экскурсии

Фильтр помогает пользователям быстрее убрать неподходящие варианты  и сфокусироваться на релевантных

Если фильтр работает правильно, то пользователь:

- быстрее находит нужный вариант,
- тратит меньше усилий на поиск,
- чаще доходит до бронирования
---

**Основная метрика:** `Конверсия в бронирование (CR)`

**Вспомогательные метрики**: 

- Конверсия в клик по экскурсии
- Время на странице
- Количество возвратов к списку
- Средний чек (AOV)
- Выручка на пользователя (ARPU)
- Доля отмен
- Доля пользователей, применивших фильтр

---

Расчитаем `конверсию в бронирование для каждой группы`:

**Группа А**: 

- 5000 пользователей  
- 300 бронирований  

**Конверсия**:

CR_A = 300 / 5000 = 6%

---

**Группа B (с фильтром)**

- 5000 пользователей  
- 450 бронирований  

**Конверсия**:

CR_B = 450 / 5000 = 9%

---

**Вывод:**

Конверсия выросла: `с 6% до 9%`

`Абсолютный прирост:`

9% − 6% = 3 процентных пункта

`Относительный рост:`

(9% − 6%) / 6% = 50%

`Прирост составил` **+150 Бронирований**

---

Новый фильтр увеличил конверсию `с 6% до 9%`, что соответствует росту на `3` процентных пункта или на `50%` относительно контрольной группы. Это существенный прирост и дополнительно `150` бронирований на 5000 пользователей.  

При подтверждении статистической значимости с помощью `z-теста для пропорций` и отсутствии негативного влияния на выручку и другие ключевые метрики фильтр целесообразно внедрять

---

**Проверим статистикой:**

Группа A  
n₁ = 5000  
x₁ = 300  
p₁ = 300 / 5000 = 0.06  

Группа B  
n₂ = 5000  
x₂ = 450  
p₂ = 450 / 5000 = 0.09  

Разница пропорций  

Δp = p₂ − p₁ = 0.09 − 0.06 = 0.03  

---

`Z-тест для пропорций`

**Нулевая гипотеза**

H₀: p₁н = p₂н  
H₁: p₁н ≠ p₂н  

---

**Оценка стандартной ошибки**

ESE = √[ (p1(1 − p1) / n1) + (p2(1 - p2)/ n₂) ]

Подробнее про формулу расчета z-статистики для сравнения пропорций можно узнать здесь:  
https://practicum.yandex.ru/trainer/statistics-basic/lesson/3f893114-03cc-4581-9140-42ede5b8984b/

p1(1 − p1) / n1 = 0.06 × 0.94 / 5000 = 0.0564 / 5000 = 0.00001128

p2(1 − p2) / n1 = 0.09 × 0.91 / 5000 = 0.0819 / 5000 = 0.00001638

ESE = √[0.00001128 + 0.00001638] = 0.0052516664

---

**Z статистика**

Z = Δp / ESE  = 0.03 / 0.0052516664 = 5.7124725211

Наше значение не попадает в интервал `от −1.96 до 1.96`, то есть выходит за пределы допустимой области. Это означает, что результат является достаточно экстремальным, и у нас есть основания отвергнуть нулевую гипотезу

---

**P-value**

- Z = 5.71 
- p value < 0.05  

Разница статистически значима

## 📊 Расчёт DAU (Daily Active Users)

```python
dau = """ 
SELECT order_date as date, COUNT(DISTINCT user_id) as DAU 
FROM orders
GROUP BY order_date
ORDER BY date	
"""

result1 = pd.read_sql(dau, conn).set_index('date')
display(result1)
```
![DAU](https://drive.google.com/uc?export=view&id=1PNS1fAz1ECE-Brkhs67hEqBEnsA9736e)

## 📊 Средний DAU по месяцам

```python
avg_dau = """
WITH dau as (
SELECT order_date as date, COUNT(DISTINCT user_id) as DAU 
FROM orders
GROUP BY order_date
ORDER BY date	
)

SELECT strftime('%Y-%m', date) as month, 
       ROUND(AVG(DAU), 2) as AVG_DAU 
FROM dau
GROUP BY 1
ORDER BY month
"""

result3 = pd.read_sql(avg_dau, conn).set_index('month')
display(result3)
```
![Средний DAU](https://drive.google.com/uc?export=view&id=1b0qImzuXgO0mCMmiAeO9_McD6NPRnJ8g)

## 📊 Расчёт MAU (Monthly Active Users)

```python
mau = """ 
SELECT strftime('%Y-%m', order_date) as month, COUNT(DISTINCT user_id) as MAU 
FROM orders 
GROUP BY 1
ORDER BY month
"""

result2 = pd.read_sql(mau, conn).set_index('month')
display(result2)
```
![MAU](https://drive.google.com/uc?export=view&id=1BXvUIUEBxcl1TvqAFCaRbBS15fI9PFHN)

## 💸 Пользователи с тратами выше среднего

```python
plat_avg = """
WITH user_payments as
(
SELECT user_id, SUM(price * quantity) AS total_payment
FROM orders
GROUP BY user_id
)
SELECT user_id, total_payment
FROM user_payments
WHERE total_payment > (SELECT AVG(total_payment) FROM user_payments)
"""

plat = pd.read_sql(plat_avg, conn).set_index('user_id')
display(plat)
```

## 👥 Сегментация пользователей по количеству заказов

```python
orders_user_count = df[['user_id', 'order_id']].groupby('user_id').count()

one_order = (orders_user_count['order_id'] == 1).sum()
more_than_three = (orders_user_count['order_id'] > 3).sum()

print(f'''Кол-во пользователей с одним заказом: {one_order}
Кол-во пользователей с более, чем тремя заказами: {more_than_three}''')

```

## 🏆 Пользователь с максимальной выручкой (GMV)

```python
user_gmv = df[['user_id', 'gmv']].groupby('user_id').sum()
top_user = user_gmv.loc[[user_gmv['gmv'].idxmax()], :]

display(top_user)
```
