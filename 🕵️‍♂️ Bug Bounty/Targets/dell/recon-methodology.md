# Comprehensive Recon Map Following Hussein Daher's Methodology

## 1. ASN Mapping
```bash
# Step 1: Find ASN Information
URL: https://bgp.he.net/search
Search Terms: 
- Dell Technologies
- Dell Inc.
- Dell Computer Corporation

# Step 2: Extract IP Ranges
- Save all IP ranges from ASN results
- Convert CIDR ranges to individual IPs using prips
prips 192.168.1.0/24 > dell_ips.txt
```

## 2. Domain Discovery
```bash
# Step 1: Primary Domain Enumeration
amass enum -passive -d dell.com -o dell_domains.txt
sublist3r -d dell.com -o dell_sublist.txt
bbot -t dell.com -f subdomain -o dell_bbot.txt

# Step 2: Recursive Subdomain Pattern
{dev,staging,test,preprod,uat,qa}.*.dell.com
*.{dev,staging,test,preprod,uat,qa}.dell.com
```

## 3. VHOST Discovery
```bash
# Step 1: Prepare Host Headers List
cat dell_domains.txt | sort -u > vhost_wordlist.txt

# Step 2: BurpSuite Intruder Setup
Target: https://dell.com
Position: Host header
Payload: vhost_wordlist.txt

# Step 3: Analysis
- Group by response length
- Check for unique status codes
- Look for different content in responses
- Document non-standard server headers
```

## 4. Web Content Discovery

### 4.1 URL Gathering
```bash
# Step 1: Active Crawling
katana -u "https://dell.com" -jc -o dell_katana.txt
katana -list dell_domains.txt -jc -o dell_all_katana.txt

# Step 2: Historical Data
gau --subs dell.com > dell_gau.txt
waybackurls dell.com > dell_wayback.txt

# Step 3: Combine Results
cat dell_katana.txt dell_gau.txt dell_wayback.txt | sort -u > dell_all_urls.txt
```

### 4.2 Custom Wordlist Creation
```bash
# Step 1: Extract Endpoints
cat dell_all_urls.txt | LinkFinder > dell_endpoints.txt

# Step 2: Create Custom Wordlist
cat dell_all_urls.txt dell_endpoints.txt | unfurl paths | sort -u > dell_wordlist.txt
```

### 4.3 Fuzzing Strategy
```bash
# Directory Enumeration
ffuf -w dell_wordlist.txt -u https://DOMAIN/FUZZ -H "Host: DOMAIN"

# Parameter Fuzzing
ffuf -w dell_wordlist.txt -u https://DOMAIN/path?FUZZ=value

# Recursive Fuzzing
for dir in $(cat discovered_dirs.txt); do
    ffuf -w dell_wordlist.txt -u https://DOMAIN/$dir/FUZZ
done
```

## 5. Advanced Dorking

### 5.1 Google Dorks
```plaintext
# Basic Patterns
site:*.dell.com
site:*dell* 
site:dell.* inurl:admin
site:dell.* ext:php
site:dell.* intext:password
site:dell.* intitle:"index of"

# Advanced Patterns
site:*<dell.*>*
site:dell>*
inurl:dell filetype:conf
```

### 5.2 Bing IP Search
```plaintext
# For each IP from ASN mapping
ip:x.x.x.x
ip:"x.x.x.x"
```

## 6. Content Analysis Pipeline

```bash
# Step 1: Initial Processing
cat all_discovered_urls.txt | grep -iE "api|json|xml|admin|test|dev|stag|lab|swagger"

# Step 2: Parameter Extraction
cat all_discovered_urls.txt | unfurl format %q > parameters.txt

# Step 3: Technology Detection
# Look for:
- JavaScript frameworks
- API endpoints
- Development endpoints
- Test environments
- Documentation paths
```

## 7. Recursive Discovery Loop

```bash
# Step 1: For each discovered subdomain
for domain in $(cat dell_domains.txt); do
    # Run Katana
    katana -u "https://$domain" -jc -o "$domain_urls.txt"
    
    # Extract endpoints with LinkFinder
    cat "$domain_urls.txt" | LinkFinder > "$domain_endpoints.txt"
    
    # Fuzzing with custom wordlist
    ffuf -w dell_wordlist.txt -u "https://$domain/FUZZ"
done
```

## 8. Final Analysis

### 8.1 Pattern Recognition
- Look for similar endpoint structures
- Identify common parameters
- Map technology stacks
- Document authentication methods
- Note error handling patterns

### 8.2 Documentation Format
```markdown
## Target Information
- Domain: 
- IP Range:
- Technologies:
- Interesting Endpoints:

## Notable Findings
- Authentication Methods:
- API Patterns:
- Development Traces:
- Sensitive Files:
```

## 9. Continuous Monitoring
```bash
# Add discovered domains to continuous monitoring
for domain in $(cat dell_domains.txt); do
    # Monitor for new subdomains
    # Monitor for new endpoints
    # Monitor for technology changes
done
```

## Workflow Notes:
1. Start with broad enumeration
2. Focus on interesting patterns
3. Deep dive into promising areas
4. Document everything systematically
5. Create custom tools for repetitive tasks
6. Maintain separate wordlists for different contexts
7. Keep track of successful patterns
