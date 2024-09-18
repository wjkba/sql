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
