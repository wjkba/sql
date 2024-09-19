# sql
Baza danych to kolekcja danych zapisanych w formacie, który ułatwia dostęp do danych

**DBMS** - Database Managment System

SQL służy do komunikacji z relacyjną bazą danych

Tabelka składa się z rekordów i kolumn

# Pobieranie danych z pojedynczej tabeli

## SELECT

`USE nazwa_database`  zaznacza database

`SELECT nazwa_kolumny`  zaznacza kolumne lub  `*` (wszystkie kolumny)

`FROM tabela`  z tabeli do której wysyłamy zapytanie

```jsx
USE sql_store
```

Poniższy kod zaznaczy wszystkie kolumny z tabeli customers

```jsx
SELECT * FROM customers
```

`SELECT` może zaznaczać kilka kolumn na raz, używać można wyrażeń arytmetycznych

`AS nazwa_kolumny` zmienia nazwę kolumny, którą dostajemy z powrotem, może być ze spacjami `"nazwa kolumny"` 

```jsx
SELECT 
	last_name, 
	first_name, 
  points, 
  (points + 10) * 100 AS discount_factor
FROM customers

```

`SELECT DISTINCT kolumna` zaznacza wartości w kolumnie bez duplikatów

```jsx
SELECT DISTINCT state
FROM customers
```

## WHERE

Klauzula `WHERE` służy do filtrowania rekordów.

Służy do pobierania tylko tych rekordów, które spełniają określony warunek.

Poniższy kod zaznaczy customers, którzy mają więcej punktów niż 2000

```sql
SELECT *
FROM customers
WHERE points > 2000
```

Można zaznaczyć rekordy z konkretną wartością

```sql
WHERE state = "VA"
```

```sql
WHERE state != "VA"
```

Daty można porównywać, ale zapisujemy je za w quotation mark

```sql
WHERE birth_date > "1990-01-01"
```

## AND, OR, NOT

do `WHERE` możemy odawać operatory `AND` , `OR` i `NOT` 

`AND` jest wyżej w kolejności działań niż `OR` , kolejność można zmienić przy użyciu nawiasów tak jak w matmie

Poniższy kod zaznaczy wszystkie rekordy spełniające warunki

```sql
SELECT *
FROM customers
WHERE birth_date > "1990-01-01" OR (points > 1000 AND state = "VA")
```

```sql
SELECT *
FROM customers
WHERE NOT (birth_date > "1990-01-01" OR points > 1000)
-- to samo zapisane inaczej
-- WHERE (birth_date <= "1990-01-01" AND points <= 1000)
```

## IN

Operator `IN` umożliwia określenie wielu wartości w klauzuli `WHERE`

Operator `IN` jest skrótem dla wielu warunków `OR`

Zamiast  

```sql
WHERE state = "VA" OR state = "FL" OR state = "GA"
```

Można użyć `IN`

```sql
WHERE state IN ("VA", "FL", "GA")
```

`IN` można łączyć z `NOT` 

```sql
WHERE state NOT IN ("VA", "FL", "GA")
```

## BETWEEN

Operator `BETWEEN` wybiera wartości w danym zakresie. Wartości mogą być liczbami, tekstem lub datami

```sql
SELECT * FROM customers
-- WHERE points >= 1000 AND points <= 3000
-- to samo
WHERE points BETWEEN 1000 AND 3000
```

## LIKE

Operator `LIKE` jest używany w klauzuli `WHERE` do wyszukiwania określonego wzorca w kolumnie

Poniższy kod korzysta z patternu, który zaznacza wszystkie wartości zaczynające się na b

```sql
SELECT * FROM customers
WHERE last_name LIKE "b%"
```

## REGEXP

Można używać regular expressions

`^` początek stringa

`$` końec stringa

`|` logical or

`[abcd]` matchowanie w środku brackets

`[a-f]` zakres 

```sql
SELECT * FROM customers
WHERE last_name REGEXP "b[ru]|ey$|^my"
```

## IS NULL

Dostęp do rekordów, które mają puste wartości (null)

```sql
SELECT * FROM customers
WHERE phone IS NULL
```

## ORDER BY

`ORDER BY` służy do sortowania zbioru wyników w porządku rosnącym lub malejącym

```sql
SELECT * FROM customers
ORDER BY state DESC, first_name
```

w przykładzie niżej na samym początku zdefiniowałem alias, który moge użyć dalej

```sql
SELECT *, quantity * unit_price AS total_price
FROM order_items
WHERE  order_id = 2
ORDER BY total_price DESC
```

## LIMIT

`LIMIT liczba` zwraca liczbę rekordów

Przydatne jest przy stronach, paginacji

```sql
SELECT * FROM customers
LIMIT 3
```

`LIMIT offset, n` offset daje znać ile rekordów pominąć, po pominięciu wyświetla `n` liczbe rekordów

```sql
SELECT * FROM customers
LIMIT 6, 3
```
# Pobieranie danych z wielu tabel

## Inner join

Klauzula `JOIN` służy do łączenia wierszy z dwóch lub więcej tabel na podstawie powiązanej kolumny między nimi

Poniższy kod łączy wewnętrznie tabelę orders z tabelą customers na podstawie `customer_id`. Zapytanie patrzy na `customer_id` tabeli orders i znajduje odpowiedni rekord z tym samym id w tabeli customers.

`o` i `c` obok orders i customers to aliasy, które sprawiają, że kod jest bardziej czytelny

```sql
SELECT order_id, o.customer_id, first_name, last_name
FROM orders o
JOIN customers c
	ON o.customer_id = c.customer_id
```

## Łączenie między bazami danych

Kiedy chcemy uzyskać dostęp do tabeli z innej bazy danych niż aktualna musimy dodać prefix

W tym przypadku chcę mieć dostęp do tabeli `products` , która jest w bazie `sql_inventory`

`sql_inventory.products`

```sql
SELECT *
FROM order_items oi
JOIN sql_inventory.products p
 ON oi.product_id = p.product_id
```

## Self joins

Łączenie tabeli z samą sobą

Jedyna różnica w łączeniu ze sobą tej samej tabeli jest taka, że musimy mieć dwa aliasy i pamiętać o prefiksach kiedy łączymy ze sobą dane

Niżej mamy baze danych, która zawiera informacje o pracownikach, każdy pracownik ma przypisane id managera a sam manager też jest pracownikiem

```sql
USE sql_hr;
SELECT e.employee_id, e.first_name, m.first_name AS manager_name
FROM employees e
JOIN employees m
    ON e.reports_to = m.employee_id
```

## Łączenie wielu tabel

W przypadku więcej niż jednej tabeli postępujemy tak jak wcześniej i po prostu używamy znowu klauzuli `JOIN` , trzeba pamiętać o prefixach i aliasach

```sql
SELECT order_id,order_date,c.first_name, c.last_name, os.name AS status
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id
JOIN order_statuses os
    ON o.status = os.order_status_id
```

## Compound Join Conditions

W poniższym przykładzie łączymy tabele na podstawie dwóch warunków

```sql
SELECT *
FROM order_items oi
JOIN order_item_notes oin
    ON oi.order_id = oin.order_Id
    AND oi.product_id = oin.product_id
```

## Outer joins

Outer join’y zwracają dopasowane i niedopasowane wartości z jednej lub obu tabel

`LEFT` i `RIGHT` joiny automatycznie są outer, nie trzeba używać `LEFT/RIGHT OUTER`

`LEFT JOIN` zwraca wartości z lewej tabeli

`RIGHT JOIN` zwraca wartości z prawej tabeli

Niżej mamy dwie tabele `products` i `order_items` , stosujemy lewego outer joina i wynikiem będą wybrane kolumny na podstawie `product_id`  - te, które spełniają warunek i te, które nie spełeniają warunku. Outer joiny jest z lewej strony czyli jeśli product z tabeli `products` nie ma zamówienia w `order_items` to wynikiem `quantity` będzie `null`

```sql
SELECT p.product_id, p.name, oi.quantity
FROM products p
LEFT JOIN order_items oi 
	ON p.product_id = oi.product_id
```

## Outer join między wieloma tabelami

Przy wielu tabelach najlepszą praktyką jest używanie `LEFT JOIN` , lepiej nie mieszać left i right bo kod jest mniej czytelny

```sql
SELECT c.customer_id, c.first_name, o.order_id, sh.name AS shipper
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
LEFT JOIN shippers sh
    ON o.shipper_id = sh.shipper_id
ORDER BY c.customer_id
```

## Self Outer Joins

W tym przykładzie każdy pracownik ma przypisane id managera, w zwykłym self join’ie wynikiem będą tylko pracownicy, którzy mają managera. W tym przypadku dostaniemy również rekord samego managera, który też jest pracownikiem a managerem tego managera jest `null`

```sql
SELECT e.employee_id, e.first_name, m.first_name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.reports_to = m.employee_id
```

## USING

Jeśli kolumny między tabelami mają takie same nazwy nie trzeba pisać warunku `ON` wartość jednej kolumny równa się druga. Wystarzcy użyć `ON (nazwa_wspolnej_kolumny)`

```sql
SELECT o.order_id, c.first_name
FROM orders o
JOIN customers c
    -- ON o.customer_id = c.customer_id
    USING (customer_id)
```

```sql
SELECT *
FROM order_items oi
JOIN order_item_notes oin
    USING(order_id, product_id)
#     ON oi.order_id = oin.order_Id AND
#        oi.product_id = oin.product_id
```

## Natural joins

Natural join łączy tabele na podstawie wspólnych kolumn

⚠️ Nie mamy kontroli nad tym co wybieramy, ten join nie jest rekomendowany

```sql
SELECT o.order_id, c.first_name
FROM orders o
NATURAL JOIN customers c
```

## Cross joins

Cross joiny łączą każdy rekord z pierwszej tabeli z każdym rekordem z drugiej tabeli bez warunku.

Wszystko ze wszystkim

Explicit:

```sql
SELECT sh.name, p.name
FROM shippers sh
CROSS JOIN products p
ORDER BY shipper_id
```

Implicit (ten sam wynik):

```sql
SELECT sh.name, p.name
FROM shippers sh, products p
ORDER BY shipper_id
```

## Unions

Operator `UNION` służy do łączenia zestawu wyników dwóch lub więcej instrukcji `SELECT`

```sql
SELECT order_id, order_date,
       'Active' AS status
FROM orders
WHERE order_date >= '2019-01-01'
UNION
SELECT order_id, order_date,
       'Archived' AS status
FROM orders
WHERE order_date < '2019-01-01'
```

```sql
SELECT customer_id, first_name, points,
       'Bronze' AS type
FROM customers
    WHERE points < 2000
UNION
SELECT customer_id, first_name, points,
       'Silver' AS type
FROM customers
    WHERE points BETWEEN 2000 AND 3000
UNION
SELECT customer_id, first_name, points,
       'Gold' AS type
FROM customers
    WHERE points > 3000
ORDER BY first_name
```
