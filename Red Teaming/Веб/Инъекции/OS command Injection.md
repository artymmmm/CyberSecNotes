Инъекции команд ОС позволяют атакующему выполнять команды ОС на сервере, на котором размещено веб-приложение
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

