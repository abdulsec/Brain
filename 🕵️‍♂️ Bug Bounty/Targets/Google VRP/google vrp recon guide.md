
# Google VRP Bug Hunting Reconnaissance Guide

## Program Overview & Scope

### In-Scope Domains
- *.google.com
- *.youtube.com
- *.blogger.com
- *.gstatic.com
- *.googleapis.com
- *.google.com.* (ccTLDs)
- *.google.org
- *.android.com
- *.chrome.com
- *.chromium.org
- *.ggpht.com
- *.googlevideo.com
- *.googleusercontent.com
- *.ytimg.com
- *.appspot.com
- *.google.net
- *.certificate-transparency.org
- *.web.dev

### Key Focus Areas
1. First Party Web Apps
2. Cloud Platform Services
3. Open Source Projects
4. Android Apps & System
5. Chrome Browser & OS
6. Hardware/IoT Devices

## 1. ASN Enumeration

Google ASNs to investigate:
- AS15169 - Google LLC
- AS36040 - Google LLC
- AS36385 - Google LLC
- AS43515 - Google Ireland Limited
- AS45566 - Google LLC
- AS41264 - Google Ireland Limited

Commands:
```bash
# Using ASNLookup
asn -i AS15169 > google_asn_ranges.txt

# Using whois
whois -h whois.radb.net -- '-i origin AS15169' | grep -Eo "([0-9.]+){4}/[0-9]+" >> google_ranges.txt
```

## 2. Acquisition Research

Notable Google acquisitions to include in scope:
- YouTube (youtube.com)
- Blogger (blogger.com)
- Firebase (firebase.com)
- Nest (nest.com)
- Waze (waze.com)
- FitBit (fitbit.com)
- DeepMind (deepmind.com)
- Mandiant (mandiant.com)

## 3. Subdomain Enumeration

### Using Amass
```bash
# For each major domain
amass enum -d google.com -o google_amass.txt
amass enum -d youtube.com -o youtube_amass.txt
amass enum -d blogger.com -o blogger_amass.txt
```

### Using Subfinder
```bash
subfinder -d google.com -o google_subfinder.txt -all
```

### Using BBOT
```bash
bbot -t google.com -f subdomain-enum --scope-report
```

## 4. GitHub Reconnaissance

### Key GitHub Dorks
```
org:google inurl:auth
org:google password
org:google api_key
org:google "google.com"
org:google "internal"
org:google "dev"
org:google "staging"
org:google "prod"
```

### Using github-subdomains
```bash
github-subdomains -d google.com -t YOUR_GITHUB_TOKEN
```

## 5. Cloud Infrastructure Analysis

### GCP Resources
```bash
# Using GCP Asset Inventory (if available)
gcloud asset search-all-resources --scope="organizations/ORGID"

# Looking for exposed storage buckets
gsutil ls gs://google-*
```

## 6. Special Considerations

### High-Value Targets
1. Authentication Systems
   - SSO endpoints
   - OAuth implementations
   - Account management

2. Cloud Platform
   - GCP Console
   - Cloud APIs
   - Cloud Storage
   - App Engine apps

3. Business Systems
   - Google Workspace
   - Google Cloud
   - Enterprise tools
   - Admin consoles

4. Developer Tools
   - Cloud Console
   - Firebase Console
   - API endpoints
   - Developer documentation

### Out of Scope
- Social engineering
- DDoS
- Physical security
- Recently acquired companies (first 6 months)
- Cloud customer applications
- Third-party apps using Google OAuth

## 7. Vulnerability Categories & Rewards

### Priority Areas ($31,337+)
- RCE
- SSRF leading to sensitive data access
- XXE
- SQL Injection
- Authentication bypasses

### High Impact ($13,337+)
- XSS in sensitive applications
- CSRF in sensitive applications
- Information disclosure
- Access control issues

### Medium Impact ($1,337+)
- Limited XSS
- Limited CSRF
- URL redirection
- Minor information leaks

## 8. Recon Automation

Using ReconFTW:
```bash
./reconftw.sh -d google.com -r --deep
```

Using rengine:
```bash
python3 rengine.py -d google.com --deep
```

## 9. Testing Methodology

1. Infrastructure Mapping
   - Map all subdomains
   - Identify technology stacks
   - Look for non-standard ports
   - Identify cloud resources

2. Application Analysis
   - Authentication flows
   - Authorization mechanisms
   - API endpoints
   - File upload functionality
   - User input handling

3. Special Focus Areas
   - OAuth implementations
   - SSO systems
   - API security
   - Cloud misconfigurations
   - Developer tools

## 10. Best Practices

1. Always check if target is in scope
2. Use Google's test accounts/domains
3. Never impact production systems
4. Follow responsible disclosure
5. Document everything thoroughly
6. Use their issue tracker
7. Be patient with responses
8. Follow up appropriately

## 11. Reporting Guidelines

1. Clear description
2. Proof of concept
3. Impact assessment
4. Steps to reproduce
5. Suggested fix
6. Video/screenshots if needed

## Useful Tools

1. Recon
   - Amass
   - Subfinder
   - BBOT
   - Shodan
   - Censys

2. Testing
   - Burp Suite Pro
   - OWASP ZAP
   - Custom scripts
   - Postman

3. Documentation
   - Screenshots
   - Screen recordings
   - HTTP logs
   - Network captures

## Resources

1. Official Documentation
   - VRP Guidelines
   - Submission Portal
   - Technical Guidelines

2. Community Resources
   - Google VRP Writeups
   - Bug Bounty Forums
   - Research Papers

Remember:
- Quality over quantity
- Follow the rules strictly
- Report responsibly
- Be patient
- Document thoroughly