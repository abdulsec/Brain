shodan dork for better bugbounty hunting


### Find API Keys in HTML Body

```
http.html:"xoxb-" // Slack
http.html:"AKIA"  // AWS
http.html:"AIza"  // Google Maps
```
### Find all Grafana Dashboards using the website's title

`http.title:"Grafana"`

### Open and Login Successful FTP Ports

`org:orgName port:21 "230 Login Successful"`
### FTP Anonymous Login

`230 'anonymous@' login ok org:organization-name`
### Find targets using ASN (Autonomous System Number)

`asn:AS63293`

### **Search using favicon hashes**

`http.favicon.hash:81586312`
### **Increase attack surface with SSL certificate**

`ssl.cert.subject.cn:a[pple.com](http://pple.com/?ref=rashahacks.com)`  
`ssl.cert.issuer.cn:a[pple.com](http://pple.com/?ref=rashahacks.com)`

### **Find the WordPress wp-config.php configuration file**

`http.html:"The wp-config.php creation script uses this file"`

### **Find a particular backend component**

`http.component:AngularJS org:organization-name`

### **Public Directory Listing**

```
http.title:"Index Of /"
http.title:"Index Of /admin"
http.title:"Directory Listing" org:organization-name
```

**find devices based on an IP address or /x CIDR.**

`net:210.214.0.0/16`

To find copyright of the target website
`html:"eBay Inc. All Right Reserved"`


#### Shodan Port Scan & CIDR

```
shodan.io

net:CIDR/24,CIDR/24

Example

net:109.70.100.50/24,89.67.54.0/22

You could also search via Organistations Name with
org:Paypal

Of course these techniques will only return assets on your targets OWN external network but what about resources hosted with thirdparty cloud providers such as AWS or Azure. One Techn to find these is to search with SSL

ssl:paypal
```