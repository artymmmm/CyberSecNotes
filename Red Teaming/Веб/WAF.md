## Проверка наличия WAF
- `nmap -p80 --script=http-waf-detect.nse <host>`
- `nmap -p80 --script=http-waf-fingerprint.nse <host>`
- `wafw00f <host>`
## Обход WAF
- Unicode Normalization: [Unicode Normalization Vulnerabilities & the Special K Polyglot](https://appcheck-ng.com/unicode-normalization-vulnerabilities-the-special-k-polyglot/)