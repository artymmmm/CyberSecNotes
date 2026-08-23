## Проверка наличия WAF
- `nmap -p80 --script=http-waf-detect.nse <host>`
- `nmap -p80 --script=http-waf-fingerprint.nse <host>`
- `wafw00f <host>`