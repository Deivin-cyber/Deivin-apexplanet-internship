# DVWA — SQL Injection Attack Scenarios + Mitigation Notes

**Purpose:** Demonstrate SQL Injection (manual and automated with sqlmap) on DVWA (low security) and show how to prevent it using prepared statements (PHP `mysqli` and PHP `PDO`). Include reproduction steps, evidence (commands & sample output), remediation notes, and a suggested GitHub-friendly file structure.

---

## Table of Contents

1. Overview
2. Environment
3. Attack scenarios

   * Manual SQL injection (union-based)
   * Automated dump using `sqlmap`
4. Demonstration commands (what I ran)
5. Mitigation: Prepared Statements

   * `mysqli` (procedural)
   * `mysqli` (OOP)
   * `PDO`
6. Additional hardening recommendations
7. How to add this file to GitHub
8. Appendix & references

---

## 1. Overview

This document shows how a basic SQL Injection (SQLi) can be used to extract user data from DVWA (Damn Vulnerable Web App) running on Kali. It also shows defensive code using prepared statements to prevent SQLi.

> **Important:** Only perform these tests in a legal, controlled environment (your local VM or lab). Do not attack systems you don't own or have permission to test.

---

## 2. Environment

* Kali Linux (VM)
* DVWA installed and configured (security level: `low`)
* Web server: Apache (localhost / 127.0.0.1)
* Tools used: browser, `sqlmap` (installed on Kali)

Example DVWA URL used in this demo:

```
http://127.0.0.1/DVWA/vulnerabilities/sqli/
```

Cookies used: the active `PHPSESSID` and `security=low` are required for `sqlmap` attacks.

---

## 3. Attack scenarios

### A. Manual SQL Injection (union-based)

1. Open the DVWA SQLi page in your browser.
2. In the `id` parameter input, inject a union query to dump the `users` table fields:

Example payload (union-based):

```
1 UNION SELECT user, password FROM users#
```

`#` is a comment in MySQL and truncates the rest of the SQL.

When executed on a vulnerable page, this will cause the application to display the username and hashed password columns.

### B. Automated dump using `sqlmap`

You can automate discovery & dump using `sqlmap`.

Example `sqlmap` command used in this demo (note: adjust cookie and URL for your setup):

```
sqlmap -u "http://127.0.0.1/DVWA/vulnerabilities/sqli/?id=18&Submit=Submit" \
  --cookie="PHPSESSID=6dd0a943453e9825ba2a379361f6a5bf; security=low" \
  -p id --batch --dump -T users -C user,password
```

Explanation of flags:

* `-u` : target URL
* `--cookie` : provide session cookie (DVWA requires login/session)
* `-p` : parameter to test (`id`)
* `--batch` : non-interactive
* `--dump` : dump table contents
* `-T users` : target table
* `-C user,password` : columns to dump

**Example output:**

`sqlmap` will identify injectable parameter(s) and dump the requested columns showing usernames and password hashes.

---

## 4. Demonstration commands (what I ran)

Manual payload used in the DVWA form input:

```
1 UNION SELECT user, password FROM users#
```

`sqlmap` command used (example):

```
sqlmap -u "http://127.0.0.1/DVWA/vulnerabilities/sqli/?id=18Submit=Submit" \
  -cookie="PHPSESSID=6dd0a943453e9825ba2a379361f6a5bf; security-low" \
  -batch -dump -T users -C user,password
```

> Note: In some environments you may need to URL-encode or wrap arguments differently. The cookie name for DVWA may be `security` (set to `low`) rather than `security-low` — ensure cookies match what your browser session shows.

---

## 5. Mitigation: Prepared Statements

The core prevention is **never** to interpolate user input directly into SQL. Instead use parameterized queries (prepared statements). Below are examples in PHP using `mysqli` (procedural and OOP) and `PDO`.

### A. `mysqli` (procedural) — safe example

```php
<?php
// connect
$conn = mysqli_connect('127.0.0.1', 'dvwauser', 'dvwapass', 'dvwa');

// unsafe: $id = $_GET['id'];
$id = isset($_GET['id']) ? $_GET['id'] : 0;

// prepared statement
$stmt = mysqli_prepare($conn, "SELECT first_name, last_name FROM users WHERE user_id = ?");
mysqli_stmt_bind_param($stmt, 'i', $id); // 'i' = integer
mysqli_stmt_execute($stmt);
mysqli_stmt_bind_result($stmt, $first, $last);
while (mysqli_stmt_fetch($stmt)) {
    echo htmlentities($first) . ' ' . htmlentities($last) . "\n";
}
mysqli_stmt_close($stmt);
mysqli_close($conn);
?>
```

Key points:

* Use `?` placeholders and bind input with the correct type (`i`, `s`, `d`, `b`).
* This separates query structure from data — prevents SQLi.
* Use `htmlentities()` or equivalent when printing to avoid XSS when showing DB data.

### B. `mysqli` (OOP) example

```php
<?php
$mysqli = new mysqli('127.0.0.1','dvwauser','dvwapass','dvwa');
$id = isset($_GET['id']) ? (int)$_GET['id'] : 0;
if ($stmt = $mysqli->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?")) {
    $stmt->bind_param('i', $id);
    $stmt->execute();
    $stmt->bind_result($first, $last);
    while ($stmt->fetch()) {
        echo htmlspecialchars($first) . ' ' . htmlspecialchars($last) . "<br>";
    }
    $stmt->close();
}
$mysqli->close();
?>
```

### C. `PDO` example (recommended for modern apps)

```php
<?php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=dvwa;charset=utf8mb4', 'dvwauser', 'dvwapass', [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_EMULATE_PREPARES => false,
]);
$id = isset($_GET['id']) ? $_GET['id'] : 0;
$stmt = $pdo->prepare('SELECT first_name, last_name FROM users WHERE user_id = :id');
$stmt->execute([':id' => $id]);
while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
    echo htmlspecialchars($row['first_name']) . ' ' . htmlspecialchars($row['last_name']) . "<br>";
}
?>
```

Notes for PDO:

* Setting `PDO::ATTR_EMULATE_PREPARES => false` ensures true server-side prepared statements when possible.
* Bind by name (e.g., `:id`) for readability.

---

## 6. Additional hardening recommendations

* **Least privilege DB user:** Ensure the DB account used by the web app has only required permissions (no `DROP`, no global privileges).
* **Input validation & type casting:** Validate input types (cast `id` to integer) and use allow-lists for known good values.
* **WAF:** Consider a Web Application Firewall for extra protection (e.g., ModSecurity) in production.
* **Use secure password storage:** Passwords in DVWA are intentionally vulnerable; real apps must use `password_hash()` / `bcrypt` / `argon2`.
* **Security testing in CI/CD:** Include automated scanning (e.g., sqlmap, static analysis) in your pipeline for QA environments.

---



