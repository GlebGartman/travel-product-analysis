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

## 🗄 Работа с базой данных

Для анализа используется `sqlite3` (in-memory база данных).

```python
import sqlite3

conn = sqlite3.connect(":memory:")
df.to_sql("travels", conn, index=False, if_exists="replace")
```

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
