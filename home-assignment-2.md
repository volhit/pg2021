- запустить везде psql из под пользователя postgres **[✓]**
- выключить auto commit **[✓]**
- сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit; **[✓]**
- посмотреть текущий уровень изоляции: show transaction isolation level
```
postgres=*# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
```
- начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции **[✓]**
- в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
```
postgres=*# insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
```
- сделать select * from persons во второй сессии
```
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
```
- видите ли вы новую запись и если да то почему?

**нет, потому что в первой сессии транзакция не закоммичена, а тип изоляции транзакции - `read commited`, в которой не допускается "грязное чтение".**

- завершить первую транзакцию - commit; **[✓]**
- сделать select * from persons во второй сессии
```
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
- видите ли вы новую запись и если да то почему?

**да, потому что транзакция в первой сессии завершена, а тип изоляции транзакции во второй сессии - `read commited`, в которой разрешено неповторяющееся чтение**

- завершите транзакцию во второй сессии **[✓]**
- начать новые но уже repeatable read транзации - set transaction isolation level repeatable read; **[✓]**
- в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
- сделать select * from persons во второй сессии
```
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
- видите ли вы новую запись и если да то почему?

**нет, потому что во второй сессии транзакция не закоммичена, а тип изоляции транзакции - `repeatable read`**

- завершить первую транзакцию - commit; **[✓]**
- сделать select * from persons во второй сессии
```
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 rows)
```
- видите ли вы новую запись и если да то почему?

**нет, по той же причине - во второй сессии транзакция не закоммичена, а тип изоляции транзакции - `repeatable read`**

- завершить вторую транзакцию **[✓]**
- сделать select * from persons во второй сессии
```
postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
(4 rows)
```
- видите ли вы новую запись и если да то почему?

**да, потому что завершили транзакцию и в первой, и во второй сессии.**

- остановите виртуальную машину но не удаляйте ее **[✓]**
