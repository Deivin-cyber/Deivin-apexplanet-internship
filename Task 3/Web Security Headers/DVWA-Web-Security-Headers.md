# DVWA — Web Security Headers (Apache) — Add & Verify


**Purpose:** Save current headers, add recommended security headers to Apache VirtualHost, enable HSTS for SSL hosts, restart Apache, and verify the new headers. Also includes how to scan a test site using securityheaders.com.

---

## Table of Contents

1. Overview
2. Save current headers (before changes)
3. Edit Apache VirtualHost (add headers)
4. Enable headers module & restart Apache
5. Verify headers after change
6. Scan an external test site with securityheaders.com
7. How to add this file to GitHub

---

## 1. Overview

Follow these steps to add common security headers to your Kali Apache instance hosting DVWA and verify the changes. Perform these changes only in a lab environment.

---

## 2. Save current headers (before changes)

Run this from your Kali terminal to capture the existing response headers for DVWA:

```bash
curl -I http://127.0.0.1/dvwa > headers-before.txt
```

This saves the raw HTTP response headers to `headers-before.txt` so you can compare after the change.

---

## 3. Edit Apache VirtualHost (add headers)

Open the default virtual host config (path may vary):

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Inside the `<VirtualHost *:80>` block add the following snippet (require `mod_headers`):

```apacheconf
<IfModule mod_headers.c>
    Header always set X-Frame-Options "SAMEORIGIN"
    Header set X-Content-Type-Options "nosniff"
    Header set Referrer-Policy "strict-origin-when-cross-origin"
    Header set Content-Security-Policy "default-src 'self'; script-src 'self'; object-src 'none'; frame-ancestors 'self'"
    Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
</IfModule>
```

If you have an SSL VirtualHost (`*:443`) add the same snippet there, and also add HSTS:

```apacheconf
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
```

Notes:

* Adjust the `Content-Security-Policy` value to match your app's required resources (CDNs, fonts, etc.).
* Avoid `unsafe-inline` in `script-src` unless you add nonces or hashes.

---

## 4. Enable headers module & restart Apache

Enable the headers module, test config and restart:

```bash
sudo a2enmod headers
sudo apache2ctl configtest
sudo systemctl restart apache2
```

If `configtest` reports issues, fix them before restarting.

---

## 5. Verify headers after change

Fetch response headers after the change:

```bash
curl -I http://127.0.0.1/dvwa > headers-after.txt
```

Compare `headers-before.txt` and `headers-after.txt` to verify the new headers are present.

You can also check a single header directly, for example:

```bash
curl -I http://127.0.0.1/dvwa | grep -i Content-Security-Policy
```

---

## 6. Scan an external test site with securityheaders.com

Use securityheaders.com to analyze headers of a public test site (example):

* Scan URL: `http://testphp.vulnweb.com`

Open the site in your browser and paste the URL into securityheaders.com, or use the web UI to run the scan.

**Note:** securityheaders.com scans remote sites. To test your local DVWA instance, either expose it (carefully) or mirror the header changes on a public test host.

---



