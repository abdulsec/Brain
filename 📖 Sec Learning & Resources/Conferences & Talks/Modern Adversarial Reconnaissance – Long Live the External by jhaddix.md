
tags: #recon #jhaddix #tools #asn 


<iframe width="560" height="315" src="https://www.youtube.com/embed/N_Qyx836Y9s?si=aUvAe7t1iqPyDvGO" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


he start with company asn 

asn will ip address of the owned server
these ip will lead to apex domain  website and servers

tools to use for it  

https://bgp.he.net/
https://whois.arin.net/ui/query.do

asn to port scan

he prefer naabu over other   like nmap or masscan

basic command for naabu to scan asn


```
echo "AS8346" | naabu
```

smap https://github.com/s0md3v/Smap to scan passively

the next step is to use Python script to enumerate valid Microsoft 365 domains, retrieve tenant name, and check for a Microsoft Defender for Identity (MDI) instance.

https://github.com/expl0itabl3/check_mdi


he use also cloudrange to scan the entire cloud ip range 

https://github.com/g0ldencybersec/CloudRecon


he also found that this https://github.com/blacklanternsecurity/bbot  5 percent  found more domain than amass


Subdomain Scraping (using BBOT, working with BBOT output)
The output is a file at "/root/.bbot/scans/{scan_name}/" You can clean up the output to just found subdomains per line like so:
```
cat /root/.bbot/scans/{scan_name}/output.txt | grep -F "[DNS_NAME]' | awk '{print $2}'
Jason Haddix/BuddoBot
```
