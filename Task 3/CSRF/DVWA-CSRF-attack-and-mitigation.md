# DVWA — CSRF (Cross-Site Request Forgery) — Attack Scenario + Mitigation

**Repository:** `web-app-security/dvwa-csrf`

**Purpose:** Demonstrate a CSRF attack (using a crafted HTML link that changes a user password on DVWA), reproduce the issue, explain why it works, and provide robust mitigations (synchronizer token pattern, SameSite cookies, double-submit cookie, Referer/Origin checks). Include example payloads, reproduction steps, secure coding snippets (PHP), and GitHub commit instructions.

---

## Table of Contents

1. Overview
2. Environment
3. Vulnerable scenario (what happened)
4. Attack payload (what you used)
5. Reproduction steps (lab)
6. Why this works
7. Mitigations (detailed)

   * Synchronizer (CSRF) token (recommended)
   * SameSite cookies
   * Double-submit cookie
   * Referer / Origin header checks
   * Use POST + avoid state-changing GET
8. Code examples (PHP)

   * Vulnerable endpoint example
   * Fix: Synchronizer token implementation
   * Quick mitigation: SameSite cookie config
9. Detection & logging suggestions
10. How to add this file to GitHub
11. Appendix & references

---

## 1. Overview

Cross-Site Request Forgery lets an attacker cause an authenticated user’s browser to perform actions on a vulnerable site without their consent. In this lab your crafted HTML link changed the victim's password to `1234` when clicked.

> Only test in a controlled lab environment (DVWA on Kali). Do not attack systems you don't own or have permission to test.

---

## 2. Environment

* Kali Linux VM running DVWA (security: `low`)
* Apache + PHP
* Browser with an authenticated DVWA session

---

## 3. Vulnerable scenario (what happened)

You created a simple HTML page with a link. When an authenticated DVWA user clicks the link, the application processed the request and changed the user's password to `1234`. This indicates the password-change endpoint accepted the GET request (or an unauthenticated request without CSRF protection) and performed a state-changing action.

---

## 4. Attack payload (what you used)

Your proof-of-concept HTML page (attacker page):

```html
<html>
<body>
<h3>Click to download</h3>
<a href="http://127.0.0.1/DVWA/vulnerabilities/csrf/?password_new=1234&password_conf=1234&Change=Change#">Movie 720p</a>
</body>
</html>
```

Clicking the link issued a request to the DVWA CSRF endpoint with password fields set to `1234`, triggering a password change.

---

## 5. Reproduction steps (lab)

1. Start DVWA and log in as a normal user.
2. Ensure you have a valid session cookie in the browser.
3. Open the attacker HTML page in the same browser (or in another tab) and click the `Movie 720p` link.
4. Observe that the DVWA account password changed to `1234` (try logging out and logging in with the new password).

**Capture a request/response** (optional) with browser devtools or Burp Suite to show parameters and response.

---

## 6. Why this works

* The password-change endpoint accepts parameters via a simple GET/POST request and performs a state-changing action without verifying the request origin or requiring a CSRF token.
* The browser automatically attaches the session cookie to same-origin requests, so the server sees the request as authenticated.
* No anti-CSRF defenses (synchronizer token, SameSite cookie, or origin checks) are present on the endpoint.

---

## 7. Mitigations (detailed)

### A. Synchronizer token pattern (recommended)

* Generate a per-session random token on the server and store it in the session.
* Embed the token in state-changing forms as a hidden input and validate it on the server for every sensitive action.
* Reject requests without a valid token.

Pros: Strong, widely used. Works even if SameSite isn't available.
Cons: Requires server-side changes to render forms and validate tokens.

### B. SameSite cookies

* Set session cookies with `SameSite=Lax` or `SameSite=Strict` (Lax usually balances usability and protection). This prevents cookies from being sent on cross-site requests for many cases.

Example (PHP session cookie):

```php
session_set_cookie_params(['samesite' => 'Lax', 'secure' => true, 'httponly' => true]);
session_start();
```

Limitations: Older browsers may not support `SameSite`. Not a complete substitute for CSRF tokens when third-party POST forms are needed.

### C. Double-submit cookie

* Send a CSRF token both as a cookie and as a request parameter (or header). The server verifies both match.
* Does not require server-side storage of the token (stateless), but cookie must be readable by JavaScript to copy into the request.

### D. Referer / Origin header checks

* Validate the `Origin` (preferred for POST) or `Referer` header to ensure requests originate from your domain.
* Reject requests with absent or mismatched Origin/Referer for sensitive actions.

Limitations: Some clients remove/rewrite Referer headers; slightly less robust than tokens but useful as defense-in-depth.

### E. Use POST and protect state changes

* Ensure actions that change state (password change) use POST, not GET. Browsers and crawlers may prefetch GET links; using GET for state changes is unsafe.

---

## 8. Code examples (PHP)

### A. Vulnerable endpoint (example)

```php
// vulnerable_change.php (do NOT use in production)
if (isset($_REQUEST['Change'])) {
    $new = $_REQUEST['password_new'];
    $conf = $_REQUEST['password_conf'];
    if ($new === $conf) {
        // update password in DB (insecure example)
        // ...
        echo "Password changed";
    }
}
```

### B. Fix: Synchronizer token implementation (simplified)

**On form render (change-password page):**

```php
session_start();
if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}
$token = $_SESSION['csrf_token'];
?>
<form method="POST" action="/vulnerabilities/csrf/change_password.php">
  <input type="password" name="password_new" />
  <input type="password" name="password_conf" />
  <input type="hidden" name="csrf_token" value="<?= htmlspecialchars($token) ?>" />
  <button type="submit" name="Change">Change</button>
</form>
```

**On server (change_password.php):**

```php
session_start();
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405); exit;
}
if (empty($_POST['csrf_token']) || !hash_equals($_SESSION['csrf_token'], $_POST['csrf_token'])) {
    http_response_code(403); echo "CSRF token validation failed"; exit;
}
// proceed with password change
```

Notes:

* Use `hash_equals()` to compare tokens to avoid timing attacks.
* Regenerate token on sensitive events if desired.

### C. Quick mitigation: Set SameSite on session cookie (example)

```php
// before session_start()
session_set_cookie_params(['samesite' => 'Lax', 'secure' => true, 'httponly' => true]);
session_start();
```

Combine SameSite with tokens for defense-in-depth.

---

## 9. Detection & logging suggestions

* Log state-changing requests (IP, user-agent, referer/origin, POST body) and monitor for repeated requests with identical parameters.
* Alert if many password changes occur from the same IP or user in a short time.
* Use WAF rules to detect suspicious cross-origin requests or known CSRF patterns.

---



*End of document.*
