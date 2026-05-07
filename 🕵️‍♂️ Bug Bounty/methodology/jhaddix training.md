# TBHM Live - Course Info

I am thrilled to introduce you to The Bug Hunter's Methodology LIVE, my masterclass designed for aspiring and seasoned offensive security professionals, including web application security testers, red teamers, and bug bounty hunters.

The Bug Hunter's Methodology (TBHM) is a two-day, paid, virtual training that aims to equip you with the latest tools, techniques, and strategies, plus provide a data-driven methodology on how and where to search for vulnerabilities that are currently common in the wild.

Unlike other courses, TBHM Live is not an A-Z or beginner-oriented course. True to the spirit of my public TBHM talks, my emphasis is on expert tips, time-saving tricks, practical Q&As, automation strategies, vetted resources, and engagement via the dedicated community on Discord.

**Here are the details for the upcoming masterclasses:**

  

Class 1 for US Timezone: March 2nd-3rd , 10-6 MST

Class 2 for EU Timezone: March 9th-10th , 5am-1pm MST

  

Each module will be driven live, using real-time targets where possible. You'll have access to all source material to refer back to after the training. Plus, a video recording of the class will be available for all participants shortly after the course concludes.

TBHM Live is also much more than just a course. I am dedicated to fostering a vibrant and supportive community for our learners. In keeping with this commitment, I will maintain a Discord channel for ongoing support, including resume guidance and job placement assistance.

Join us for TBHM Live and get ready to supercharge your skills, refine your strategies, and join an active community of like-minded professionals.

I look forward to seeing you in the class!  
  

**Full syllabus:**

  

# Day 1 - Recon

  

**Recon Part 1: Recon Concepts**

- Introduction to Recon
    

  

**Recon Part 2: Acquisitions and Domains**

- Scope
    
- Shodan
    
- ASN Analysis
    
- Crunchbase ++
    
- ReconGTP
    
- Reverse WHOIS
    
- Certificate Analysis
    
- Add and Analytics Relationships
    
- Supply chain investigation and SaaS
    
- Google-fu (trademark & Priv Pol)
    
- TLDs Scanning
    
- 0365 Enumeration for Apex Domains
    

  

**Recon Part 3: Subdomain Enumeration**

- Subdomain Scraping (all the best sources and why to use them)
    
- Security Trails + Netlas
    
- Brute force
    
- Wildcards
    
- Permutation Scanning
    
- Linked Discovery
    
- Wordlists
    
- Advantageous Subs (WAF bypass - Origins)
    
- Favicon analysis
    
- Sub sub domains
    
- Esoteric techniques
    
- Dnssec / nsec / nsec3 walking
    

  

**Recon Part 4: Server & App Level Analysis**

- Port Scanning
    
- Service Bruteforce
    
- Tech Stack
    
- Screenshotting
    

  

**Recon Part 5: Profiling People for Social Engineering**

- Linkedin (people, tech)
    
- Hunter.io
    
- Hiring Sites
    

  

**Recon Part 6: Recon Adjacent Vulnerability Analysis**

- CVE scanners vs Dynamic Analysis
    
- Subtakover
    
- S3 buckets
    
- Quick Hits (swagger, .git, configs, panel analysis)
    

  

**Recon Part 7: Recon Frameworks and Helpers**

- Frameworks
    
- Understanding your framework
    
- Tips for success (keys)
    
- Distribution and Stealth
    

  

  

# Day 2 - Application Analysis

  

**Application Analysis Part 1: Analysis Concepts**

- Indented usage (not holistic, contextual)
    
- Analysis Layers
    
- Application Layers as related to success.
    
- Tech profiling
    
- The Big Questions
    
- Change monitoring
    

  

**Application Analysis Part 2: Vulnerability Automation**

- More on CVE and Dynamic Scanners
    
- Dependencies
    
- Early running so you can focus on manual.
    
- Secrets of automation kings
    

  

**Application Analysis Part 3: Content Discovery**

- Intro to CD (walking, brute/fuzz, historical, JS, spider, mobile, params)
    
- Importance of walking the app
    
- Bruteforce Tooling
    
- Bruteforce Tooling Lists: based on tech
    
- Bruteforce Tooling Lists: make your own (from-install, dockerhub, trials, from word analysis)
    
- Bruteforce Tooling Lists: generic/big
    
- Bruteforce Tooling Lists: quick configs
    
- Bruteforce Tooling Lists: API
    
- Bruteforce Tooling Tips: Recursion
    
- Bruteforce Tooling Tips: sub as path
    
- Bruteforce Tooling Tips: 403 bypass
    
- Historical Content Discovery
    
- Newschool JavaScript Analysis
    
- Spidering
    
- Mobile Content Discovery
    
- Parameter Content Discovery
    

  

**Application Analysis Part 4: The Big Questions**

- How does the app pass data?
    
- How/where does the app talk about users?
    
- Does the site have multi-tenancy or user levels?
    
- Does the site have a unique threat model?
    
- Abuse Primitives
    
- Has there been past security research & vulns?
    
- How does the app handle common vuln classes?
    
- Where does the app store data?
    

  

**Application Analysis Part 5: Application Heat Mapping**

- Common Issue Place: Upload functions
    
- Common Issue Place: Content type multipart-form
    
- Common Issue Place: Content type XML / JSON
    
- Common Issue Place: Account section and integrations
    
- Common Issue Place: Errors
    
- Common Issue Place: Paths/URLs passed in parameters
    
- Common Issues Place: chatbots  
    

  

**Application Analysis Part 6: Web Fuzzing & Analyzing Fuzzing Results**

- Parameters and Paths (generic fuzzing)
    
- Reducing Similar URLs
    
- Dynamic only fuzzing
    
- Fuzzing resources SSWLR - "Sensitive Secrets Were Leaked Recently”
    
- Backslash powered Scanner
    

  

**Application Analysis Part 7: Introduction to Vulnerability Types**

- Indented usage (not holistic. Tips and Contextual)
    
- Covered vulns and why
    

  

**Application Analysis Part 8: XSS Tips and Tricks**

- Stored and Reflected
    
- Polyglots
    
- Blind
    
- DOM
    
- Common Parameters
    
- Automation and Tools
    

  

**Application Analysis Part 9: IDOR Tips and Tricks**

- IDOR, Access, Authorization, MLAC, Direct browsing Business logic, parameter manipulation
    
- Numeric IDOR
    
- Identifying user tokens GUID IDOR
    
- Common Parameters
    

  

**Application Analysis Part 10: SSRF Tips and Tricks**

- SSRF intro
    
- schemas
    
- Alternate IP encoding
    
- Common Parameters
    

  

  

**Application Analysis Part 11: XXE**

- Common areas of exploitation
    
- Payloads
    
- Common Parameters
    

  

  

**Application Analysis Part 12: File Upload Vulnerabilities Tips and Tricks**

- Common bypasses
    
- Common Parameters
    

  

  

**Application Analysis Part 13: SQL Injection Tips and Tricks**

- Manual Identification
    
- SQLmap tamper
    
- Common Parameters
    

  

  

**Application Analysis Part 14: Command Injection Tips and Tricks**

- Common Parameters
    

  

  

**Application Analysis Part 15: COTS and Framework Scanning**

- Default Creds
    
- CMS's WordPress + Adobe Experience Manager
    
- Others  
      
    

**Application Analysis Part 16: Bypass of security controls**

- Subdomains where controls are not applied
    
- Origins
    
- TLDs (.jp, .uk, .xx)  
    

  

# Red Team Analysis

  

**Red Teaming Analysis Part 1: Initial Access Primer**

- Phishing Tips and Tricks
    
- Threat Intel + Levels
    
- Credential Stuffing
    
- Open discussion of C2
    
- SaaS
    
- Cloud
    

  

**Red Teaming Analysis Part 2: Post Initial Access**

- Open Discussion of common internal methods to succeed