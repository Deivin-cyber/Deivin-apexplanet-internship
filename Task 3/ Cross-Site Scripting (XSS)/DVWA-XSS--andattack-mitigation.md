# DVWA — XSS Attack Scenarios + Mitigation Notes

**Purpose:** Demonstrate Stored and Reflected Cross‑Site Scripting (XSS) on DVWA (low security), include the payloads used, reproduction steps, and focused mitigations: **Input Validation** and **Content Security Policy (CSP)**. Provide ready-to-use examples (PHP and headers) and GitHub commit instructions.

---

## Table of Contents

1. Overview
2. Environment
3. Attack scenarios

   * Stored XSS
   * Reflected XSS
4. Payloads used (real examples)
5. Reproduction steps
6. Mitigation strategies

   * Input validation & sanitization
   * Output encoding
   * Content Security Policy (CSP)
   * Additional defenses
7. Code examples

   * PHP input validation & output encoding
   * CSP header examples (Apache / Nginx / Meta tag)
   * Node.js (helmet) example
8. How to add this file to GitHub
9. Appendix & references

---

## 1. Overview

This document documents two common XSS types on DVWA (low security): **Stored XSS** and **Reflected XSS**. The goal is to reproduce the attacks in a controlled lab and demonstrate practical mitigations focusing on server-side input validation and a strict Content Security Policy (CSP).

> **Warning:** Only run these attacks in a controlled environment (local VM / lab). Do not test systems you do not own or have explicit permission to test.

---

## 2. Environment

* Kali Linux (VM)
* DVWA installed and configured (security level: `low`)
* Web server: Apache (localhost / 127.0.0.1)
* Tools used: browser (DevTools), Burp Suite (optional)

---

## 3. Attack scenarios

### A. Stored XSS

**Description:** Malicious script is saved by the application (e.g., in a comment or profile field) and runs when other users view the stored content.

**Example payload used:**

```
<body onload=alert('test')>
```

This payload triggers a browser alert when the stored HTML is rendered directly into a page without encoding.

### B. Reflected XSS

**Description:** Attacker-controlled input is reflected in the server response and executed immediately (e.g., search, error messages, query parameters).

**Example payload used:**

```
<script>alert("XSS Found");</script>
```

If a URL parameter is reflected into the HTML without encoding, the script runs when the URL is visited.

---

## 4. Payloads used (real examples)

* Stored: `<body onload=alert('test')>` — useful for fields where HTML is stored and later rendered.
* Reflected: `<script>alert("XSS Found");</script>` — direct script injection via query parameters.

---

## 5. Reproduction steps

**Stored XSS:**

1. Log in to DVWA and set security to `low`.
2. Find a form that saves input (comments, posts, profile fields).
3. Submit the stored payload (`<body onload=alert('test')>`).
4. Visit the page that displays the saved data — the alert executes.

**Reflected XSS:**

1. Identify a URL parameter or form that echoes input (e.g., `?q=` or an error page).
2. Append the reflected payload to the parameter and load the URL.
3. The script executes immediately if the parameter is inserted into the response without encoding.

---

## 6. Mitigation strategies

### Input validation & sanitization (server-side)

* Treat all input as untrusted.
* Use allow-lists where possible (e.g., only allow certain characters or strict patterns for usernames, IDs, numeric fields).
* For free-form text (comments), sanitize on output rather than aggressively stripping on input — prefer encoding.

### Output encoding (contextual)

* Encode data based on where it is placed: HTML-encode for HTML body, attribute-encode for attributes, JavaScript-encode for inline JS contexts, URL-encode for query strings.
* In PHP use `htmlspecialchars($data, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` before echoing into HTML.

### Content Security Policy (CSP)

A CSP reduces the impact of XSS by restricting what scripts and resources a page may load or execute.

**CSP goals:**

* Disallow `unsafe-inline` scripts/styles.
* Restrict script sources to your origin and trusted CDNs.
* Consider using nonces or hashes for any inline scripts that must exist.

**Example strict CSP header (good starting point):**

```
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'; upgrade-insecure-requests;
```

**If you must allow specific inline scripts, use nonces:**

* Generate a random nonce per response and include it both on the server `Content-Security-Policy: script-src 'self' 'nonce-<random>'` and on inline `<script nonce="<random>">...`.

**Reporting:**

* Add `report-uri` or `report-to` to collect CSP violations during rollout (helpful to identify missed resources).

### Additional defenses

* Use modern templating frameworks that auto-escape (e.g., Twig, Blade, React JSX).
* Set `HttpOnly` on session cookies to make them inaccessible to JavaScript.
* Use `SameSite` and `Secure` attributes on cookies.
* Apply input size limits and rate limits to reduce attack surface.

---

## 7. Code examples

### A. PHP — Input validation & output encoding

```php
<?php
// Example: safely rendering a user-submitted comment
$comment = isset($_POST['comment']) ? $_POST['comment'] : '';
// Minimal server-side normalization (trim + length limit)
$comment = substr(trim($comment), 0, 2000); // limit length

// When rendering into HTML, always encode
echo htmlspecialchars($comment, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
?>
```

**Notes:**

* Prefer output encoding. Avoid trying to remove every possible tag at input-time — it's error-prone.

### B. CSP header examples

**Apache (in .htaccess or virtual host):**

```
Header set Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'; upgrade-insecure-requests;"
```

**Nginx (in server block):**

```
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'; upgrade-insecure-requests;" always;
```

**HTML meta tag (only for testing / older setups — headers are preferred):**

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self';">
```

### C. Using nonces (example flow)

1. Generate a random nonce per request on server-side (e.g., 16+ bytes, base64).
2. Add it to the CSP header: `script-src 'self' 'nonce-<nonce-value>'`.
3. Add the same nonce attribute to any inline `<script nonce="<nonce-value>">` you intend to allow.

**PHP snippet (simplified):**

```php
<?php
$nonce = base64_encode(random_bytes(16));
header("Content-Security-Policy: script-src 'self' 'nonce-{$nonce}';");
?>
<script nonce="<?= $nonce ?>">console.log('allowed inline');</script>
```

### D. Node.js (Express + helmet) example

```js
const express = require('express');
const helmet = require('helmet');
const app = express();

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'"],
      objectSrc: ["'none'"],
    }
  }
}));

app.listen(3000);
```

