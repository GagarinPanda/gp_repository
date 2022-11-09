1) Какую сумму в среднем в месяц тратит:
- пользователи в возрастном диапазоне от 18 до 25 лет включительно
- пользователи в возрастном диапазоне от 26 до 35 лет включительно

WITH
x AS (SELEct userid
      FROM Users
  	  Where age >= 18 AND age <= 25),

y AS (SELEct userid
      FROM Users
  	  Where age >= 26 AND age <= 35)
      
SELECT SUM(x.price) / COUNT(EXTRACT(month FROM x.date)) AS avg_revenue_18_25,
       SUM(y.price) / COUNT(EXTRACT(month FROM y.date)) AS avg_revenue_26_35
FROM (x LEFT JOIN Purchases AS pur ON x.userId = pur.userId Left JOIN Items as i ON pur.itemId = i.itemId) AS x,
	 (y LEFT JOIN Purchases AS pur ON y.userId = pur.userId Left JOIN Items as i ON pur.itemId = i.itemId) AS y;

2) В каком месяце года выручка от пользователей в возрастном диапазоне 35+ самая большая
     
SELECT EXTRACT(month FROM date) AS month_number,
	   SUM(price) AS total_revenue
FROM Purchases AS pur LEFT JOIN Users AS u ON pur.userId = u.userId LEFT JOIN Items AS i ON pur.itemId = i.itemId
WHERE age >= 35
GROUP BY EXTRACT(month FROM date)
ORDER BY total_revenue DESC
LIMIT 1;

3) Какой товар обеспечивает дает наибольший вклад в выручку за последний год

SELECT pur.itemId,
	   SUM(price) AS revenue
FROM Purchases AS pur LEFT JOIN Items AS i ON pur.itemId = i.itemId
WHERE EXTRACT(YEAR FROM date) = (select max(EXTRACT(YEAR FROM date))
                                FROM Purchases)
GROUP BY pur.itemId
ORDER BY revenue DESC
LIMIT 1;

4) Топ-3 товаров по выручке и их доля в общей выручке за любой год

WITH
x AS (SELEct SUM(price) AS total_revenue
      FROM Purchases AS pur LEFT JOIN Items AS i ON pur.itemId = i.itemId
  	  WHERE EXTRACT(YEAR FROM date) = (select max(EXTRACT(YEAR FROM date))
                                FROM Purchases)),
                                
y AS (select pur.itemId,
	   SUM(price) AS revenue
FROM Purchases AS pur LEFT JOIN Items AS i ON pur.itemId = i.itemId, x
WHERE EXTRACT(YEAR FROM date) = (select max(EXTRACT(YEAR FROM date))
                                FROM Purchases)
GROUP BY pur.itemId
ORDER BY revenue DESC
LIMIT 3)

select itemId,
	   revenue,
	   ROUND((revenue::NUMERIC / total_revenue::NUMERIC) * 100, 2) AS rate
FROM x, y;