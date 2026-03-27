## Источники информации
- [SPbCTF A&D Кароч: первым делом на вулнбоксе](https://www.youtube.com/watch?v=hxEQdgabKUQ&t=901s&pp=ugUEEgJydQ%3D%3D)
- [SPbCTF A&D: мониторинг атак и патчинг](https://www.youtube.com/live/MfFVUaakUlY?si=R3ZYsTnb5JoRqE6U)
- [SPbCTF A&D: образы с докер-контейнерами](https://www.youtube.com/watch?v=yvvB9KLhggI&list=PLLguubeCGWoZT2myk_-FwBt0afThSqZKu&index=11)
## Первые действия на вулнбоксе
### Подключение к VPN
#### OpenVPN
- Linux: `apt install openvpn`, `openvpn --config config.ovpn`
- Windows: [Openvpn](https://openvpn.net/community/)
- MacOS: `brew install openvpn`, [Tunnelblick](https://tunnelblick.net/downloads.html)
#### WireGuard
- Linux: 
	1. `sudo apt install wireguard wireguard-tools`
	2. `sudo wg-quick up <путь до файла .conf>`
- MacOS:  
	1. `brew install wireguard-tools`
	2. `sudo wg-quick up <путь до файла .conf>`
### Информация о сети
- `ifconfig` - посмотреть, какой IP в VPN  
- `route -n` - посмотреть, докуда можно достучаться через VPN
### Подключение к образу
- Windows: [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)  
- Linux, MacOS: `ssh team18@7.1.1.1`
### Поиск запущенных сервисов
- `ps auxf` — процессы в виде дерева, обратить внимание на пользователя (в A/D часто сервисы под своими пользователями)
- `netstat -tunlp` - слушающие порты, обратить внимание на PID процессов; необходмо различать то, что торчит наружу (сервисы), от того, что доступно локально (вспомогательные демоны). 127.0.0.1/ : : 1 : .... - слушает локально, 0.0.0.0/ : : : .... - слушает со всех IP.
### Файлы процесса  
В папке `/proc` хранится информация о запущенных процессах. Если у нас есть процесс с PID 1, в папке `/proc/1` будут храниться его файлы. Если процесс запущен не под пользователем, под которым сейчас произведен вход, можно зайти под `root`, чтобы посмотреть содержимое папки. Полезные файлы:
- `/proc/.../exe` - бинарный файл, который запущен
- `/proc/.../cwd` - папка, в которой работает процесс
- `/proc/.../fd/**` - открытые файлы сервиса
Есть два варианта, какой процесс будет слушать на порту:  
1. У сервиса есть самописный процесс, он и слушает. Тогда в `/proc/.../exe` будет файл, который нужно изучать.
2. Сервис пользуется сторонним демоном, который слушает за него ([xinetd](http://vault.centos.org/3.7/docs/html/rhel-rg-en-3/s1-tcpwrappers-xinetd-config.html), [apache](https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps)/[nginx](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04), [systemd](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/using_systemd_unit_files_to_customize_and_optimize_your_system/assembly_working-with-systemd-unit-files_working-with-systemd)). Тогда нужно изучать конфиги этого демона. Конфиги обычно лежат в `/etc`.
Запасной вариант как найти сервисы - места автозагрузки в Linux:  
- `/etc/rc.local` - скрипт, который выполняется один раз при загрузке  
- `/etc/init.d/` - каталог с описаниями сервисов для SysV Init (старый стандарт)  
- `/etc/systemd/system/` (вероятней), `/lib/systemd/system/` (менее) - каталоги с сервисами для SystemD (новый стандарт)
### Взаимодействие с сервисами
- `nc -nv 7.1.1.1 16404` - TCP
- `nc -nvu 7.1.1.1 5417` - UDP (по UDP сервис не знает, что соединение "открыто", надо ему что-нибудь послать)
- `openssl s_client -connect 7.1.1.1:8443` - SSL. Если вывело инфу о сертификате, значит успешно установило с сервисом SSL-сессию
 - `curl -v http://7.1.1.1:8100` - HTTP, `-v` нужен, чтобы видеть заголовки  (`curl -vk https://7.1.1.1:8443` - HTTPS)
Стоит пробовать посылать сервисам `help`, `?` и т.д., смотреть, что они отвечают, гуглить ответы (вдруг известный протокол).
### Передача файла с образа
- `scp -r team18@7.1.1.1:/home /tmp/` - копирование по пути
- `sftp team18@7.1.1.1` - открывается шелл с возможностью делать `cd`, `ls`
- [FileZilla](https://filezilla-project.org/download.php?type=client) (GUI)
## Docker
### Провера запущенных контейнеров и образов
Статус, имя образа, на котором основан, проброс портов: `docker ps -a`  
Список образов: `docker image ls`
### Исходники
Поиск в `/`:
- `find / -name Dockerfile`
- `find / -name docker-compose.yml`
### Бэкап контейнера
- `docker export '<имя (или ID) контейнера>' > name.tar`  
### Патчинг контейнера
1. Скопировать файлы из контейнера: `docker cp <container_id>:/app /tmp/backup`
2. Пропатчить файлы
3. Скопировать файлы в контейнер: `docker cp patched_file <container_id>:/app/file`
4. Перезапустить контейнер: `docker restart <container_id>`  
5. Добавить изменения в образ: `docker commit <container_id> <new_image>`  
### Полезное в CTF
Запуск дополнительного процесса в контейнере (`bash`) и присоединение к нему с максимальными привилегиями: `docker exec -ti --privileged '<имя (или ID) контейнера>' /bin/bash`  
Копирование файлов между хостовой машиной и контейнером:
- `docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-`
- `docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH`  
Запуск/остановка контейнера: `docker start/stop '<имя (или ID) контейнера'>`  
Сохранение изменений (в tag помечается версия): `docker commit 'ID контейнера' 'имя image':'tag'`  
Pull+create+start: `docker run [параметры] 'имя image'`  
Доступ в корень файловой системы контейнера (без общих папок):
1. `ps auxf`: `root      3675 0.0 0.0  4384  816 ?       Ss  Oct25  0:00 |      \_ /app/serv 10556` (PID запущенного в контейнере процесса (а не самого docker-процесса))
2. `ls /proc/3675/root`  
Скачивание образа с https://hub.docker.com:   `docker pull 'имя image'  `
Удалить контейнер: `docker rm 'имя (или ID) контейнера'`  
Удалить образ: `docker rmi 'имя image'`  
Присоединиться к stdin/stdout текущего процесса в контейнере: `docker attach 'имя (или ID) контейнера'`  
Отсоединиться от контейнера, не дропнув его:  `ctrl+p`, `ctrl+q`  
### Параметры 'docker run'
- `-d` - фон
- `-ti` - получение ввод/вывод (обязательный набор для /bin/bash)
- `-v` - общая папка, на хост машине по пути: `/var/lib/docker/volumes/`
- `-p` - проброс портов
- `--restart always` - следить, чтоб всегда был в UP  
Эти параметры задаются на стадии `create` и при `start` уже недоступны для изменения
### Пример Dockerfile
```dockerfile
FROM ubuntu
WORKDIR /app
COPY ./files /app
EXPOSE 4125
RUN apt-get update && apt-get install -y python3 python3-pip && pip3 install flask
CMD ["/app/server.py"]
```
### Создание своего образа (для этого требуется Dockerfile)
- `docker build -t <image name> .`
### Docker-compose
Удобный run всего, что прописано в yml файле: 
- `docker-compose up -d`
### Пример docker-compose.yml
```yml
version: '3'
services:
 cryptobulki:
  image: "i_cryptobulki"
  restart: always
  ports:
    - "4125:4125"
 smartbox:
  image: "i_smartbox"
  restart: always
  ports:
    - "10556:10556"
 dungeon:
  image: "i_dungeon:ver_1"
  restart: always
  ports:
    - "666:666"
```
## Защита
### Смена паролей
- Сменить пароли пользователей, админов в сервисах
- Сменить пароли пользователей операционной системы: `passwd <user>`
### Мониторинг атак
#### Логи
- Логи сервиса обычно лежат в папке со всеми файлами сервиса
- Веб-серверы: `/var/log/apache2/`, `/var/log/nginx/`
- Конфигурационные файлы веб-серверов, где указываются пути логов:  `/etc/apache2/`, `/etc/nginx/`
- При отсутствии логирования в сервисе можно добавить его. В идеале логирование необходимо помещать до шифрования/расшифрования ответов/запросов. Пример для PHP (в файл будет помещаться тело POST-запросов): `@file_put_contents("/tmp/service_logs", print_r($_POST, true), FILE_APPEND);`
#### Запись трафика
- Запись трафика в файл: `tcpdump -s 0 -i any -f 'port not 22' -C 100 -w dump.pcap`
- Получение отдельных файлов для каждого TCP-стрима: `tcpflow -r dump.pcap -o tcps/` 
- Фильтр для пакетов с флагами: `tcp matches "[A-Z0-9]{31}="`
- Перенос трафика с вулнбокса сразу в Wireshark: `ssh root@6.1.1.1 "tcpdump -s 0 -i any -f 'port not 22' -w -" | wireshark -k -i -` 
- Без буфера: `ssh root@6.1.1.1 "dumpcap -i any -P -f 'port not 22' -w-" | wireshark -k -i -` (для `dumpcap` необходимо поставить `apt -y install tshark`)
### Патчинг уязвимостей
Если файл занят каким-то процессом, то есть невозможно пропатчить, можно сделать следующее:
1. Скопировать файл: `cp <file> <file_copy>`
2. Отредактировать скопированный файл: `vim <file_copy>`
3. Переименовать старый файл и сразу переименовать новый: `mv <file> <file_old>; mv <file_copy> <file>`  
Важный момент: следить за привилегиями и владельцами файлов. Обычно для каждого сервиса имеется собственный пользователь/группа, под которым работает сервис.  
- Cмена владельца файла: `chown <user> <file>`
- Смена группы файла: `chown :<group> <file>`
- Смена владельца и группы файла: `chown <user>:<group> <file>`
- Смена привилегий файла: `chmod <mode: 0750> <file>`
#### Отредактировать код и перезапустить 
- Работают без перезапуска, изменения подхватываются на лету: xinetd, сайты на PHP
- Сервисы (`/etc/init.d/`, `/etc/systemd/`): `service <service_name> restart`
#### Бинарные файлы
Изменить строчку в бинарном файле:
- Консольный hex-редактор: `hexedit` 
- Hex-редактор c GUI: [010 Editor](http://www.sweetscape.com/download/010editor/)  
Изменить инструкцию Assembler:  
- IDA:  Edit→Patch program→Assemble→Apply patches
### Межсетевой экран
- iptables: 
	- `iptables -A INPUT -m string --algo kmp --string "TESTTEST" -j DROP`; 
	- `iptables -A INPUT -m string --algo kmp --hex-string "|4b004b00|" -j DROP
	- Активные правила: `iptables-save`
	- Убрать правило: `iptables -D` (вместо `-A`)
### Переброс трафика
Перебросить весь трафик, который идёт на порт сервиса (`:8443`), на другой порт где висит пропатченный сервис (`:18443`): `iptables -t nat -A PREROUTING -d 6.1.1.1 -p tcp --dport 8443 -j DNAT --to :18443`  
Клиенты будут коннектиться на `8443`, и прозрачно попадать на `18443`. Можно поднять запатченную копию сервиса на другом порту и временно перебросить трафик на неё, чтобы проверить, не поплохеет ли чекеру.  
Вместо `:18443` можно написать `6.6.17.102:18443` - прозрачно пробросить на другой хост.
### Просмотр расшифрованного трафика
Вклиниться в SSL и смотреть расшифрованный трафик в консоли: `socat openssl-listen:18443,fork,reuseaddr,cert=1.crt,key=1.key,verify=0 system:'tee /dev/stderr | socat - openssl\:127.0.0.1\:8443\,verify=0 | tee /dev/stderr'`  
Перед этим нужно сгенерировать ключ и сертификат: `openssl genrsa 2048 > 1.key`, `openssl req -new -x509 -out 1.crt -key 1.key`  
Что происходит:
1. Первый `socat` слушает порт `18443` с шифрованием
2. На каждый коннект он запускает команду, которая в кавычках
3. Первая `tee` отправляет данные пришедшие на порт, на консоль (stderr) и дальше во второй `socat`
4. Второй `socat` подключается к `127.0.0.1:8443` с шифрованием и обменивается данными с stdin/stdout (первый параметр "_-_")
5. Вторая `tee` отправляет данные, выведенные `socat`, на консоль и на вывод, где их подхватывает первый `socat`
### Патчинг сетевых пакетов на живую
Можно патчить сетевые пакеты на живую, меняя в них текст по правилу:
`socat tcp-l:16667,fork,reuseaddr system:'stdbuf -i0 -o0 sed "s/hello/preved/g" | socat - tcp\:127.0.0.1\:6667 | stdbuf -i0 -o0 sed "s/[A-Z0-9]\\\\{31\\\\}=/CENSORED/g"'`  
Слушает `16667` без шифрования, конектится на `127.0.0.1:6667`. Меняет от клиента к серверу `hello`→`preved`, а от сервера к клиенту - любые флаги по регулярке на `CENSORED`. На что обратить внимание:  
- этот пример можно совместить с предыдущим с SSL
- `sed` работает построчно, поэтому будет работать только с протоколами, у которых любые данные заканчиваются переводом строки. Иначе `sed` не отдаст данные, пока не встретит перевод строки. Можно заменить `sed` своим скриптом, который не будет заточен под строчки
- `\` для `sed` пришлось превратить в `\\` из-за экранирования в `socat`. Можно заменить всю команду на вызов внешнего скрипта, в котором уже будет этот пайплайн: `exec:/path/to/script.sh`
- убирать флаги по регулярке - сломает чекер. Он же их тоже проверяет
## Чекер
Чекер каждый раунд делает следующие действия:
- Check: проверка функциональности сервиса
- Put: чекер кладет новый флаг в сервис
- Get: чекер достает один из старых флагов  
Флаги протухают после нескольких раундов (обычно 3-4 раунда). Один раунд  - 1-5 минут.
