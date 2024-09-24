# SQL
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

# Wstawianie, aktualizacja i usuwanie danych

## Wstawianie pojedynczego rekordu

`INSERT INTO nazwa_tabeli` służy do wstawiania nowych rekordów do tabeli.

w `VALUES()` wpisujemy wartości, które chcemy wstawić

metoda 1, tutaj trzeba zachować kolejnośc kolumn tak jak w bazie danych

```sql
INSERT INTO customers
VALUES (
        DEFAULT,
        'JOHN',
        'SMITH',
        '1990-01-01',
        NULL,
        'address',
        'city',
        'CA',
        DEFAULT
       )
```

metoda 2, tutaj sami wybieramy kolejność wstawianych wartości

```sql
INSERT INTO customers (last_name,
                       first_name,
                       birth_date,
                       address,
                       city,
                       state)
VALUES (
        'SMITH',
        'JOHN',
        '1990-01-01',
        'address',
        'city',
        'CA'
       )
```

## Wstawianie wielu rekordów

```sql
INSERT INTO shippers (name)
VALUES ('Shipper1'),
       ('Shipper2'),
       ('Shipper3')
```

## Wstawianie rekordów do wielu tabel z hierarchią

W tym przypadku chcemy dodać nowy zamówienie do tabeli `orders` i zaraz po nowym zamówieniu dodajemy przedmioty do tabeli `order_items` 

Żeby dodać nowe przedmioty do `order_items` tabeli potrzebujemy `order_id` nowego zamówienia, id możemy dostać za pomocą funkcji `LAST_INSERT_ID()` , ktorej używamy jako wartości kiedy dodajemy nowy rekord przedmiotu do tabeli `order_items` 

```sql
INSERT INTO orders (customer_id, order_date, status)
VALUES(1, '2019-01-02', 1);

INSERT INTO order_items
VALUES(LAST_INSERT_ID(), 1, 1, 2.95),
      (LAST_INSERT_ID(), 2, 1, 3.95)
```

## Kopiowanie tabeli

Przy kopiowaniu tabeli w taki sposób tracimy niektóre atrybuty kolumn, np. primary key nie będzie obecny w nowej kopii tabeli

```sql
CREATE TABLE orders_archived AS
SELECT * FROM orders
```

Niżej przykład używania statmentu `SELECT` jako subquery w `INSERT` statement

```sql
INSERT INTO orders_archived
SELECT *
FROM orders
WHERE order_date < '2019-01-01'
```

```sql
CREATE TABLE invoices_archived AS
SELECT invoice_id, number, c.name, invoice_total, i.payment_total, invoice_date, due_date, payment_date
FROM invoices i
JOIN clients c
	USING(client_id) 
	WHERE i.payment_date IS NOT NULL
ORDER BY i.invoice_id
```

## Aktualizacja pojedynczego rekordu

`UPDATE` służy do modyfikowania istniejących rekordów w tabeli

```sql
UPDATE invoices
SET payment_total = 10, payment_date = '2024-09-20'
WHERE invoice_id = 1
```

Można aktualizować dane na podstawie aktualnych wartości innych kolumn

```sql
UPDATE invoices
SET payment_total = invoice_total * 0.5, payment_date = due_date
WHERE invoice_id = 3
```

## Aktualizacja wielu rekordów

Aktualizacja rekordów dla klientów o id 3 i 4

```sql
UPDATE invoices
SET payment_total = 999,
    payment_date = '2024-09-20'
WHERE client_id IN (3,4) 
```

## Używanie subqueries w aktualizacji

W tym przykładzie znajdujemy `client_id` na podstawie wartości `name` , która otrzymujemy przy pomocy `SELECT`

```sql
UPDATE invoices
SET
    payment_total = 999,
    payment_date = '2024-09-20'
WHERE client_id = (SELECT client_id FROM clients WHERE name = 'Myworks')
```

```sql
-- add comment to orders for customers who have more than 3000 points
UPDATE orders
    SET comments = 'GOLD'
WHERE customer_id IN (SELECT customer_id FROM customers WHERE points > 3000)
```

## Usuwanie rekordów

`DELETE from nazwa_tabeli` służy do usuwania istniejących rekordów w tabeli.

```sql
DELETE from invoices
WHERE client_id = (SELECT client_id FROM clients WHERE name = 'Myworks')
```

# Podsumowywanie danych

## Funkcje agregujące

Funkcje agregujące działają tylko na wartościach, które nie są `NULL` . Rekordy z wartościami `NULL` są pomijane w egzekucji funkcji.

```sql
SELECT MAX(invoice_total) AS highest,
       MIN(invoice_total) AS lowest,
       AVG(invoice_total) AS average,
       SUM(invoice_total) AS total,
       COUNT(invoice_total) AS number_of_invoices,
       COUNT(payment_date) AS count_of_payments, -- wartości null będą pominięte
       COUNT(*) AS total_records
FROM invoices
```

`DISTINCT` zwraca tylko unikalne wartości, w przykładzie niżej zapewnia to, że `client_id` nie będzie powtórzone drugi raz

```sql
SELECT COUNT(DISTINCT client_id) AS total_records
FROM invoices
```

```sql
SELECT
    'First half of 2019' AS date_range,
    SUM(invoice_total) AS total_sales,
    SUM(payment_total) AS total_payments,
    SUM(invoice_total - payment_total) AS what_we_expect
FROM invoices
WHERE invoice_date BETWEEN '2019-01-01' AND '2019-06-30'
UNION
SELECT
    'Second half of 2019' AS date_range,
    SUM(invoice_total) AS total_sales,
    SUM(payment_total) AS total_payments,
    SUM(invoice_total - payment_total) AS what_we_expect
FROM invoices
WHERE invoice_date BETWEEN '2019-07-01' AND '2019-12-31'
UNION
SELECT
    'TOTAL' AS date_range,
    SUM(invoice_total) AS total_sales,
    SUM(payment_total) AS total_payments,
    SUM(invoice_total - payment_total) AS what_we_expect
FROM invoices
WHERE invoice_date BETWEEN '2019-01-01' AND '2019-12-31'

```

## GROUP BY

`GROUP BY`  grupuje rekordy mające te same wartości w określonych kolumnach, umożliwiając wykonywanie funkcji agregujących, takich jak `SUM`, `COUNT` czy `AVG`, na każdej grupie rekordów

`GROUP BY` jest w kolejności za `FROM` i `WHERE` i przed `ORDER BY`

```sql
SELECT
    client_id,
    SUM(invoice_total) AS total_sales
FROM invoices
GROUP BY client_id
ORDER BY total_sales DESC
```

```sql
SELECT
    state,
    city,
    SUM(invoice_total) AS total_sales
FROM invoices
JOIN clients USING(client_id)
GROUP BY state, city
```

```sql
SELECT
    date,
    pm.name AS payment_method,
    SUM(amount) AS total_payments
FROM payments p
 JOIN payment_methods pm
 ON payment_method = payment_method_id
GROUP BY date, payment_method
ORDER BY date
```

## HAVING

`HAVING` istnieje, ponieważ `WHERE` nie może być używane z funkcjami agregującymi

`HAVING` stosujemy kiedy rekordy są już pogrupowane

Kolumny, które używamy muszą być zaznaczone przez 2

```sql
SELECT
    client_id,
    SUM(invoice_total) AS total_sales
FROM invoices
GROUP BY client_id
HAVING total_sales > 500
```

```sql
SELECT c.customer_id,
       c.first_name,
       c.last_name,
       SUM(oi.unit_price*oi.quantity) AS total_sales
FROM customers c
JOIN orders o USING(customer_id)
JOIN order_items oi USING(order_id)
WHERE state = 'VA'
GROUP BY c.customer_id, c.first_name, c.last_name
HAVING total_sales > 100
```

## ROLLUP

Dodając `WITH ROLLUP` na koniec `GROUP BY nazwa_kolumny` otrzymujemy podsumowanie, sumę wszystkich poprzednich kolumn. Działa to tylko na funkcjach, które agregują wartości

```sql
SELECT
    client_id,
    SUM(invoice_total) AS total_sales
FROM invoices
GROUP BY client_id WITH ROLLUP 
```

Rollup nie działa z aliasami, trzbea wpisywać dokładną nazwę kolumny

```sql
SELECT pm.name AS payment_method,
       SUM(amount) as total
FROM payments p
JOIN payment_methods pm
    ON p.payment_method = pm.payment_method_id
GROUP BY pm.name WITH ROLLUP
ORDER BY total
```
# Complex query

## Subqueries

Find products that are more expensive that product with id of 3 

```sql
SELECT *
FROM products
WHERE unit_price > (SELECT unit_price FROM products WHERE product_id = 3)
```

Find employees that earn more than average

```sql
SELECT *
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
```

## IN

Find the products that have never been ordered

Najpierw warto napisać oddzielne query, które zaznacza wszystkie `DISTINCT` , czyli unikalne wartości z `order_items` tabeli a potem wsadzamy to query do innego query

```sql
SELECT *
FROM products p
WHERE p.product_id NOT IN (SELECT DISTINCT product_id FROM order_items)
```

Find clients without invoices

```sql
SELECT *
FROM clients
WHERE client_id NOT IN (SELECT DISTINCT client_id FROM invoices)
```

## Subqueries vs Joins

Powyższy przykład z klientami, którzy nie mają invoice’ów można zapisać użwyając `LEFT JOIN` 

Wynik będzie taki sam chociaż wcześniejszy zapis jest bardziej czytelny

```sql
SELECT *
FROM clients
LEFT JOIN invoices USING(client_id)
WHERE invoice_id IS NULL
```

Find customers who have ordered lettuce

Podejście używając subquery

```sql
SELECT c.customer_id, c.first_name, c.last_name
FROM customers c
WHERE c.customer_id IN(
    SELECT o.customer_id
    FROM order_items oi
    JOIN orders o USING(order_id)
    WHERE product_id = 3
		)
```

Podejście używając `JOIN`. W tym przypadku join jest bardziej czytelny, intuicyjny

```sql
SELECT DISTINCT customer_id, first_name, last_name
FROM customers c
JOIN orders o USING(customer_id)
JOIN order_items oi USING(order_id)
WHERE oi.product_id = 3
```

## ALL

Select invoices larger than all invoices of client 3

Można to rozwiązać na dwa sposoby, pierwszy używa wybiera maksymalną wartość invoice dla klienta i przyrównuje z innymi. Subquery tutaj zwraca jedną wartość

```sql
SELECT *
FROM invoices
WHERE invoice_total > (
    SELECT MAX(invoice_total)
    FROM invoices
    WHERE client_id = 3
);
```

Drugi sposób używa keyworda `ALL` , tutaj dla każdej wartości invoice klienta porównujemy wszystkie zaznaczone wartości w środku `ALL` .  Subquery tutaj zwraca listę wartości, dzięki `ALL` można porównać liste wartości z pojedynczą wartością. Wynik jest taki sam jak wyżej

```sql
SELECT *
FROM invoices
WHERE invoice_total > ALL(
    SELECT invoice_total
    FROM invoices
    WHERE client_id = 3
)
```

## ANY

Select clients with at lest two invocies

Dwa sposoby, dla subquery możemu użyć `IN` , który sprawdzi czy `client_id` należy do listy wartości z wyniku subquery

```sql
SELECT *
FROM clients
WHERE client_id IN (
    SELECT client_id
    FROM invoices
    GROUP BY client_id
    HAVING COUNT(*) >= 2
);
```

Zamiast `IN` można użyć `= ANY()` , wynik bedzie taki sam

```sql
SELECT *
FROM clients
WHERE client_id = ANY(
    SELECT client_id
    FROM invoices
    GROUP BY client_id
    HAVING COUNT(*) >= 2
)

```

## Corelated subqueries

Select employees whose salary is above the average in their office

Niżej subquery będzie uruchamiane dla każdego `employee` , będzie obliczało average salary dla pracowników w tym samym `office_id` . Czynność ta jest powtarzana dla każdego employee i nazywamy to corelated subquery, w subquery jest korelacja między zewnętrznym query

```sql
SELECT *
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE office_id = e.office_id
)
```

```sql
SELECT *
FROM invoices i
WHERE invoice_total > (
    SELECT AVG(invoice_total)
    FROM invoices
    WHERE i.client_id = client_id
)
```

## Exists

Select client that have an invoice

Niżej w subquery zaznaczamy unikalne `client_id` i zwracana jest lista pasujących wyników, która przyrównujemy do `client_id` w zewnętrznym query używając `WHERE client_id IN()`

```sql
SELECT *
FROM clients
WHERE client_id IN(SELECT DISTINCT client_id FROM invoices);
// WHERE client_id IN(1, 2, 3, 5)
```

Inaczej działa keyword `EXISTS` , który tylko daje znać czy dany warunek jest spełniony. W momencie kiedy znajdzie pasujący rekord to zwraca `TRUE`. Jeśli mamy dużo rekordów to `EXISTS` jest bardziej wydajnym podejściem, nie generuje długiej listy do której trzeba przyrównywać wartości tak jak w przypadku zwykłego `IN` i subquery

```sql
SELECT *
FROM clients c
WHERE EXISTS(
    SELECT client_id
    FROM invoices
    WHERE client_id = c.client_id
)
```

Find the products that have never been ordered

```sql
SELECT *
FROM products p
WHERE NOT EXISTS(
    SELECT *
    FROM order_items oi
    WHERE p.product_id = oi.product_id
)
```

## Subqueries in SELECT

Subqueries można używać w `SELECT` 

Obliczając różnicę nie mogłem zapisać `invoice_total - invoice_average` , ponieważ `invoice_average` to alias dlatego potrzebny jest przed nim `SELECT`

```sql
SELECT
    invoice_id,
    invoice_total,
    (SELECT AVG(invoice_total)
     FROM invoices) AS invoice_average,
    (invoice_total - (SELECT invoice_average)) AS difference
FROM invoices
```

```sql
SELECT client_id,
       name,
       (SELECT SUM(invoice_total)
        FROM invoices i
        WHERE(i.client_id = c.client_id)) AS total_sales,
        (SELECT AVG(invoice_total) from invoices) AS average,
        (SELECT total_sales - average) AS difference
FROM clients c
```

## Subqueries in FROM

W `FROM` można używać subquery, lepiej ograniczać się do prostych subquery. Da się też stworzyć takiego potwora jak niżej, ale do tego lepiej już nadaje się view.

```sql
SELECT *
FROM (
    SELECT client_id,
       name,
       (SELECT SUM(invoice_total)
        FROM invoices i
        WHERE(i.client_id = c.client_id)) AS total_sales,
        (SELECT AVG(invoice_total) from invoices) AS average,
        (SELECT total_sales - average) AS difference
    FROM clients c
     ) AS sales_summary
WHERE total_sales IS NOT NULL
```
