1. `git clone https://github.com/OpenAttackDefenseTools/tulip.git`
2. `cd tulip`
3. `mkdir services/pcaps`
4. `cp .env.example .env`
5. `vim .env`:
```
##############################
# Tulip config
##############################

# Timescale connection
TIMESCALE="postgres://tulip@timescale:5432/tulip"

# ПОМЕНЯТЬ!
# The location of your pcaps as seen by the host
TRAFFIC_DIR_HOST="./services/pcap"

# The location of your pcaps (and eve.json), as seen by the container
TRAFFIC_DIR_DOCKER="/traffic"

# Set BPF filter expression (see https://www.tcpdump.org/manpages/pcap-filter.7.html)
#BPF="port 8080"

# Visualizer
#VISUALIZER_URL="http://scraper.example.com"

##############################
# Game config
##############################

# ПОМЕНЯТЬ!
# Start time of the CTF (or network open if you prefer)
TICK_START="2026-03-31T13:00:00Z"

# ПОМЕНЯТЬ!
# Tick length in ms
TICK_LENGTH=60000

# ПОМЕНЯТЬ!
# The flag format in regex
FLAG_REGEX="\w{31}="

# ПОМЕНЯТЬ!
# VM IP (inside gamenet)
# Currently ignored unless FLAGID_SCRAPE is set
VM_IP="10.10.3.1"
TEAM_ID="3"

##############################
# PCAP_OVER_IP CONFIGS
##############################

# Enable pcap-over-ip and choose server endpoint
# Empty value = disabled
PCAP_OVER_IP=
#PCAP_OVER_IP="host.docker.internal:1337"
# For multiple PCAP_OVER_IP you can comma separate
#PCAP_OVER_IP="host.docker.internal:1337,otherhost.com:5050"

##############################
# DUMP_PCAPS CONFIGS
##############################

# Enable pcap dumping and select target location
# Empty value = disabled
DUMP_PCAPS=
#DUMP_PCAPS="/traffic"

# Dumping options
# Ignored unless DUMP_PCAPS is set
DUMP_PCAPS_INTERVAL="1m"
DUMP_PCAPS_FILENAME="2006-01-02_15-04-05.pcap"

##############################
# FLAGID CONFIGS
##############################

# Enable flagid scrapping
# Empty value = disabled
FLAGID_SCRAPE=
#FLAGID_SCRAPE=1

# Enable flagid scanning - Tags flag ids in traffic
# Empty value = disabled
# Does nothing unless FLAGID_SCRAPE is set
FLAGID_SCAN=
#FLAGID_SCAN=1

# ПОМЕНЯТЬ!
# Flag lifetime in ticks
# Empty value = Fallback to TICK_LENGTH
# -1 = No check, pls don't use outside testing
FLAG_LIFETIME=10
#FLAG_LIFETIME=-1
#FLAG_LIFETIME=5

# Flagid endpoint
# Default value is a test container in docker-compose-test.yml, change this for production
# Ignored unless FLAGID_SCRAPE is set
FLAGID_ENDPOINT="http://flagidendpoint:8000/flagids.json"

##############################
# FLAG_VALIDATOR CONFIGS
##############################

# Enables flag validation / fake flag feature. Must be one of: faust, enowars, eno, itad
# Empty value = disabled
FLAG_VALIDATOR_TYPE=

# Some flag validators can make use of (our) team number/ID
# Ignored unless FLAG_VALIDATOR_TYPE is set
FLAG_VALIDATOR_TEAM=42

##############################
# SURICATA CONFIGS
##############################

# Directory for Suricata files (see suricata/etc, suricata/lib/rules, suricata/logs)
# (they should be generated on first run)
#SURICATA_DIR_HOST="./suricata"
```
6. `mkdir -p ~/.ssh`
7. `chmod 700 ~/.ssh`
8. `ssh-keygen -t ed25519 -a 100 -f ~/.ssh/vulnbox_pcap -C "pcap-sync"`
9. `sudo useradd -m -s /bin/bash pcapsync` (на вулнбоксе)
10. `sudo mkdir -p /pcaps` (на вулнбоксе)
11. `sudo chown root:pcapsync /home/pcapsync/pcaps` (на вулнбоксе)
12. `sudo chmod 750 /home/pcapsync/pcaps` (на вулнбоксе)
13. `sudo usermod -aG pcapsync pcapsync` (на вулнбоксе)
14. `sudo -u pcapsync mkdir -p /home/pcapsync/.ssh` (на вулнбоксе)
15. `sudo -u pcapsync chmod 700 /home/pcapsync/.ssh` (на вулнбоксе)
16. `ssh-copy-id -i ~/.ssh/vulnbox_pcap.pub pcapsync@<VULNBOX_IP>`
17. `ssh -i ~/.ssh/vulnbox_pcap pcapsync@<VULNBOX_IP>` (тест логина без пароля)
18. `sudo tcpdump -i eth0 -n -s 0 -G 180 -Z root -w '/home/pcapsync/pcaps/traffic_%Y-%m-%d_%H-%M-%S.pcap'`
19. `vim ~/.ssh/config` (заменить IP в конфиге!):
```
Host vulnbox-pcap
    HostName 10.0.0.2
    User pcapsync
    IdentityFile ~/.ssh/vulnbox_pcap
    IdentitiesOnly yes
    ServerAliveInterval 30
    ServerAliveCountMax 3
    StrictHostKeyChecking accept-new
```
20. `rsync -av --ignore-existing --include='traffic_*.pcap' --exclude='*'  vulnbox-pcap:/home/pcapsync/pcaps/ /home/user/ctf/tulip/services/pcap/` (ручной тест)
21. `sudo vim /usr/local/bin/pull-vulnbox-pcaps.sh`:
```bash
#!/usr/bin/env bash
set -euo pipefail

DEST="/home/user/ctf/tulip/services/pcap/`"
HOST="vulnbox-pcap"

mkdir -p "$DEST"

rsync -av --ignore-existing \
  --include='traffic_*.pcap' --exclude='*' \
  "${HOST}:/home/pcapsync/pcaps/" "$DEST/"
```
22. `sudo chmod +x /usr/local/bin/pull-vulnbox-pcaps.sh`
23. `sudo vim /etc/systemd/system/pull-vulnbox-pcaps.service` (поменять имя пользователя локального в `User`):
```
[Unit]
Description=Pull rotated pcap files from vulnbox for Tulip
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
User=user
ExecStart=/usr/local/bin/pull-vulnbox-pcaps.sh
```
22. `sudo vim /etc/systemd/system/pull-vulnbox-pcaps.timer`
```
[Unit]
Description=Run pcap pull every minute

[Timer]
OnBootSec=30s
OnUnitActiveSec=60s
AccuracySec=10s
Unit=pull-vulnbox-pcaps.service

[Install]
WantedBy=timers.target
```
23. `sudo systemctl daemon-reload`
24. `sudo systemctl enable --now pull-vulnbox-pcaps.timer`
25. `systemctl list-timers | grep pull-vulnbox-pcaps`
26. `journalctl -u pull-vulnbox-pcaps.service` (проверка логов)
27. `vim /home/user/ctf/tulip/services/api/configurations.py`:
```python
services = [
    {"ip": vm_ip, "port": <port1>, "name": "<name1>"},
    {"ip": vm_ip, "port": <port2>, "name": "<name2>"},
    {"ip": vm_ip, "port": <port3>, "name": "<name3>"},
    {"ip": vm_ip, "port": <port4>, "name": "<name4>"},
    {"ip": vm_ip, "port": -1, "name": "other"},
]
```
28. `docker compose up -d --build`