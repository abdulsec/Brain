#### Reverse WHOIS Search

tools
https://tools.whoisxmlapi.com/login
https://www.whoxy.com/

## Acquisition Enumeration

- [Crunchbase](https://www.crunchbase.com/)
- Wikipedia & Open Google Search
-

## Reverse Lookup

This helps you to find out more related domains and often by manually enumerating through those related domains, you can uncover domains owned by your target organization, resulting in increased scope and attack surface.

https://viewdns.info/reverseip/
https://hackertarget.com/ip-tools/

## Discovering the IP space

**ASN**(Autonomous System Number) is a unique identifier of certain IP prefixes. Very large organizations such as Apple, Github, Tesla have their own significant IP space. To find an ASN of an organization [https://bgp.he.net/](https://bgp.he.net/) is a useful website where we can query.


Now that we have found out the ASN number, next we need to figure out IP ranges within that ASN. For this, we will use **whois** tool.

 `whois -h whois.radb.net  -- '-i origin AS714' | grep -Eo "([0-9.]+){4}/[0-9]+" | uniq` ``


#### Reverse DNS Lookups on List of IP’s



```
#https://github.com/hakluke/hakrevdns

Sometimes you may have a IP list of targets instead of domains, perhaps from ASN lookup. Here we can use a quick little tool called hakrevdns to carry out numerous reverse DNS lookups.

hakluke~$ prips 173.0.84.0/24 | hakrevdns 
173.0.84.110	he.paypal.com.
173.0.84.109	twofasapi.paypal.com.
173.0.84.114	www-carrier.paypal.com.
173.0.84.77	twofasapi.paypal.com.
173.0.84.102	pointofsale.paypal.com.
173.0.84.104	slc-a-origin-pointofsale.paypal.com.
173.0.84.111	smsapi.paypal.com.
173.0.84.203	m.paypal.com.
173.0.84.105	prm.paypal.com.
173.0.84.113	mpltapi.paypal.com.
173.0.84.8	ipnpb.paypal.com.
173.0.84.2	active-www.paypal.com.
173.0.84.4	securepayments.paypal.com.

