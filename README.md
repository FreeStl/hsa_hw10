## Postgres test:

Create table:
```CREATE TABLE "persons" (
"name" character varying(36) NOT NULL,
"age" integer NOT NULL
);
```
Insert some values:
```
INSERT INTO persons(name, age)
VALUES ('Max', 26),
('A', 20),
('D', 21),
('C', 22),
('V', 23),
('B', 24),
('M', 25),
('N', 26),
('D', 27),
('G', 28),
('F', 29)
```


### Test Dirty reads:

First transaction:
```
BEGIN;
SELECT name FROM persons WHERE age = 29;
```
Second transaction:
```
BEGIN;
UPDATE persons SET name = 'U' WHERE age = 29;
```


First transaction:
```
BEGIN;
SELECT name FROM persons WHERE age = 29;
COMMIT;

-- READ UNCOMMITTED retrieves F (same as READ COMMITTED)
-- READ COMMITTED retrieves F (dirty read has been avoided)
-- REPEATABLE READ retrieves F (dirty read has been avoided)
-- SERIALIZABLE retrieves F (dirty read has been avoided)
```

### Test Non-repeatable reads

first transaction:
```
BEGIN;
SELECT age FROM users WHERE id = 1;
```
second transaction:
```
BEGIN;
UPDATE persons SET name = 'MMM' WHERE age = 26;
COMMIT;
```
first transaction:
```
SELECT name FROM persons WHERE age = 26;
COMMIT;

-- READ UNCOMMITTED retrieves MMM (non-repeatable read)
-- READ COMMITTED retrieves MMM (non-repeatable read)
-- REPEATABLE READ retrieves N (non-repeatable read has been avoided)
-- SERIALIZABLE retrieves N (non-repeatable read has been avoided)
```

### Test Phantom reads:

first transaction:
```
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE ;
SELECT name FROM persons WHERE age > 33;
```

second transaction:
```
BEGIN;
INSERT INTO persons VALUES ('II', 37);
COMMIT;
```

first transaction:
```
SELECT name FROM persons WHERE age > 33;
COMMIT;
-- READ UNCOMMITTED retrieves T, u , II (phantom read)
-- READ COMMITTED retrieves T, u , II (phantom read)
-- REPEATABLE READ retrieves T, u , II (phantom read)
-- SERIALIZABLE retrieves T, u (phantom read has been avoided)
```

## Percona test:

percona shows similar result, exept there is a difference between READ UNCOMMITTED and READ COMMITTED. In PosgreSQL there is no difference
