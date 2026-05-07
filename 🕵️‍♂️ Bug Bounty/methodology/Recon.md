recon

Ssl.cert.subject.CN:"facebook.com" 200

Nmap -sV 

https://crt.sh/?q=Facebook+Inc.



https://discord.com/api/webhooks/1060594921440034896/sp-HVcI6gwfEIZjTTvHXkzBaKwL1HXt7HYlbXd6DtDjanSYNkNIJqSzkuORoYLyVmpI4


org:YOUR_TARGET http.favicon.hash:116323821
Use this query on Shodan to find Spring Boot servers. Then check for exposed actuators. If /env is available you can probably achieve RCE. If /heapdump is accessible you may find private keys and tokens.



home/abdulsec/wordlist/best-dns-wordlist.txt
puredns bruteforce /home/abdulsec/wordlist/best-dns-wordlist.txt domain.com -r ~/resolverst.tx


BASIC RECON & TOOLS


• Subdomain Enumerations
https://crt.sh/?q=%25.target.com
https://securitytrails.com/list/apex_domain/target.com
https://www.shodan.io/search?query=Ssl.cert.subject.CN%3A%22t
arget.com%22
Single Domain:
amass enum -passive -norecursive -noalts -d dell.com -o amass-dell.txt

ssl:"Fidelity National Information Services Inc." 200


naabu -list sub-list.txt -top-ports 1000 -exclude-ports 80,443,21,22,25 -o ports.txt
naabu -list sub-list.txt -p - -exclude-ports 80,443,21,22,25 -o ports.txt



puredns bruteforce /home/abdulsec/wordlist/best-dns-wordlist.txt indeed.com -r ~/resolvers.txt | anew indeed.txt

`sudo masscan -iL ip.range  -p80,443,8000,8082,8081,8083,8084 --rate 100000 --output-format json --output-filename scan-iprange-results.json`

cat scan-results.json | sed -e ‘/^\[/d’ -e ‘/^\]/d’ -e ‘s/,$//’ | jq -r ‘[.ip, .ports[0].port]' | sed ‘s/\t/:/’ | sort -u > alive-hosts


hakcron -f "daily" -c  "rm resolvers.txt & cd ~ && wget https://raw.githubusercontent.com/BonJarber/fresh-resolvers/main/resolvers.txt "

create words from the  subdomains

tr "." "\n"


 ssl.cert.subject.CN:"*.target.com"+200 

blind xss:  https://discord.com/api/webhooks/1066153682334404619/70eoa3OlN8ndYL42z56FwBWS4nQ4LoBn1OIzHB-TDSDQY76utWLScKiqWQ84ot_cWGfl
            https://discord.com/api/webhooks/1066153682334404619/70eoa3OlN8ndYL42z56FwBWS4nQ4LoBn1OIzHB-TDSDQY76utWLScKiqWQ84ot_cWGfl


cat main.9f55d7d5.js |  nestle -regex '(query|mutation)\s+[a-zA-Z]+[0-9]*[a-zA-Z]+(\([^(\(|\))]+\))*\s*[{:nested:}]' | tee 9f55d7d5.graphql



```
cat dell.com.lst | dnsx -a -resp-only -silent | anew dell.com.ip

```

md5v1zmxrdwgmxmzeu9mmj4n35z3b4f7




```hakcron -f "daily" -c  "rm /home/abdulsec/.config/puredns/resolvers.txt & cd /home/abdulsec/.config/puredns && wget https://raw.githubusercontent.com/trickest/resolvers/main/resolvers.txt " ```

md5v1zmxrdwgmxmzeu9mmj4n35z3b4f7



to deduplicate  a  list of subdomains


awk -F"[" '!seen[$2, $3, $4, $5]++'


regex to find all url an replace it in auto repeater

https?://(www.)?[-a-zA-Z0–9@:%.+~#=]{1,256}.[a-zA-Z0–9()]{1,6}\b([-a-zA-Z0–9()@:%+.~#?&//=]*)




#Scan Subdomains and remove https string from httpx results

subfinder -d redacted.com -silent | httpx -silent > alive.txt ;cat alive.txt | sed 's/https\?:\/\///' > scan-ip.txt

#Run on scan-ip.txt

rustscan 'scan-ip.txt' -p --ulimit 5000 -- -n -sV -Pn -oA scan-result



 site:http://site.com ext:xml | ext:conf | ext:cnf | ext:reg | ext:inf | ext:rdp | ext:cfg | ext:txt
| ext:ora | ext:ini


site:docs.google.com/spreadsheets "Orwa Inc."
site:groups.google.com "Orwa Inc" **Amazing Tip


auto repeater gegex for urls

(?i)^(https|http|file)://.*
(?i)^(https|http|file)%3A%2F%2F.*




axiom


First, create the config folder on every droplet:

axiom-exec 'mkdir /home/op/.config/subfinder' 'dirac*'
And then upload the config file:

└─$ axiom-scp subfinder.config.yaml 'dirac*':/home/op/.config/subfinder/config.yaml
Notice that dirac is the prefix of my fleet, you need to change this value accordingly.

axiom-scan domains.txt -m amass --spinup 5 --shutdown-when-done | anew subdomains.txt

axiom-scan domains.txt -m puredns-bruteforce -w /home/op/lists/seclists/Discovery/DNS/dns-Jhaddix.txt


cat main_js-0G1XIoem.js | nestle -regex '(query|mutation)\s+[a-zA-Z]+[0-9]*[a-zA-Z]+(\([^(\(|\))]+\))*\s*[{:nested:}]' | sed 's/\\n/\n/g' | grep query | sort -u  | grep query



ll4yG6THOXbN6r5RkRMBz9A/HS3LwrOZEBs0dx2ksIQ=


regular expression for burp auto repeater

(?i)^(https|http|file)://.*
(?i)^(https|http|file)%3A%2F%2F.*

User
http://169.254.169.254/latest/meta-data/hostname


sqlmap -r sqli-mozilla --level=3 -p invite_code --dbms=postgresql --tables --force-ssl


project discovery key

3cd70189-cc7c-4a5c-889a-66eff4ecdffa


 ssl: "redacted.com" 200





curl 'https://crt.sh/?q=%dell&output=json' | jq -r  '.[].name_value' | sed 's/\"//g' | sed 's/\*\.//g' | sort -u


docker run -t -d  -p 5013:5013 -e WEB_HOST=0.0.0.0 ed2440055a4a



sort httpx title by status code and title


httpx -title -cl -sc | awk -F'\\[|\\]' '{ print $4 " " $0 }' | sort -t ' ' -k1,1 | sed 's/^[^ ]* //'


find subodmain based on the copyright

Intext: "@2016 atst intellectual property.* -att.com


find more subdomain by certicate

brute force wiht a wordlist that server will understand

- **Certificate Search**
    
- **Leveraging IP to Asset Discovery**
    
- **Using CSP Headers**

- **Google Dorks**
    
- **Using Google Analytics id**
    
- **Static and Dynamic DNS Brute force**
    
- **OSINT**

`curl -s "`[`https://crt.sh/?O=Apple%20Inc.&output=json`](https://crt.sh/?O=Apple%20Inc.&output=json)`" | jq -r ".[].common_name" | tr A-Z a-z | unfurl format %r.%t | sort -u | tee apple.cert.txt`



`curl -s "`[`https://crt.sh/?O=Apple%20Inc.&output=json`](https://crt.sh/?O=Apple%20Inc.&output=json)`" | jq -r ".[].common_name" | tr A-Z a-z | unfurl format %r.%t | sort -u | tee apple.cert.txt`



find domain associated with an organizations using brave tracker

https://github.com/duckduckgo/tracker-radar/tree/main/entities

nmap

`nmap -sV -sT -sC`


katana 

```
katana -u vulnweb.com -d 5 -ps -pss waybackarchive,commoncrawl,alienvault -f qurl -jc -xhr -kf -fx -fs dn -ef woff,css,png,svg,jpg,woff2,jpeg,gif,svg
```

