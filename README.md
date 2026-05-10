# Web-app-vulnerabilities
> **Goal**: This guide demonstrates deep understanding of web application security by analyzing 3 critical OWASP vulnerabilities. Each section includes attack scenarios, vulnerable vs secure code, and fixes aligned with secure development standards.  > **Target Audience**: Developers, SOC Analysts, Junior Penetration Testers, Bug Bounty Hunters.
Comprehensive Report: 3 Critical Web Vulnerabilities with Attack Scenarios & Fixes

https://img.shields.io/badge/version-1.0-blue
https://img.shields.io/badge/Owasp-Top%2010-red
https://img.shields.io/badge/license-MIT-green

Date: 2025-01-15
Author: [Your Name]
Audience: Developers, Security Engineers, Bug Bounty Hunters

---

Table of Contents

1. SQL Injection (SQLi)
2. Cross-Site Request Forgery (CSRF)
3. Broken Authentication
4. Summary for Developers
5. Recommended Testing Tools

---

1. SQL Injection (SQLi)

Property Details
OWASP 2021 A03:2021 - Injection
Severity Critical
Exploitability Easy (automated tools available)
Impact Full database theft, data deletion, privilege escalation

How It Works

The attacker injects malicious SQL code into user input fields (e.g., login form, search box, URL parameters). The application sends this unfiltered input directly to the database, which executes it as part of a legitimate query.

Detailed Attack Scenario (Login Bypass)

Step Description
1. Identify target Web app with a login form (username + password)
2. Inject payload Username field: admin' -- , Password: any value (e.g., x)
3. Original query SELECT * FROM users WHERE username = '$username' AND password = '$password'
4. Resulting query SELECT * FROM users WHERE username = 'admin' -- ' AND password = 'x'
5. Outcome -- comments out the password check → successful admin login

Data Exfiltration Scenario (Union-based)

· Malicious input in URL parameter:
    https://example.com/product?id=1 UNION SELECT username, password FROM users
· Result: Attacker retrieves all usernames and passwords.

How to Fix (Secure Coding)

Method Description
Parameterized Queries Use PDO / Prepared Statements to separate SQL logic from data
Input Validation Whitelist allowed values (e.g., intval() for IDs)
Least Privilege DB Account Never use root or admin for application queries
Stored Procedures Restrict direct table access when possible

Secure Example (PHP - PDO):

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE username = :username');
$stmt->execute(['username' => $_POST['username']]);
```

---

2. Cross-Site Request Forgery (CSRF)

Property Details
OWASP 2021 A01:2021 - Broken Access Control
Severity High
Exploitability Medium
Impact Unauthorized actions performed on behalf of a logged-in user (email change, password reset, fund transfer)

How It Works

The attacker tricks the victim's browser into sending an unauthorized request to a web app where the victim is already authenticated. The browser automatically includes session cookies.

Detailed Attack Scenario (Email Change)

Step Description
1. Victim Logged into https://bank.com with an active session
2. Attacker Creates a malicious page or email containing: <img src="https://bank.com/change-email?new=attacker@evil.com" width="0" height="0">
3. Victim visits page Browser sends a GET request to bank.com with session cookies automatically
4. Outcome Victim's email changed to attacker@evil.com → attacker can reset the password

How to Fix

Method Description
CSRF Token Unique, unpredictable token embedded in each form and validated server-side
SameSite Cookies Set SameSite=Lax or Strict to prevent cross-origin cookie sending
Re-authentication for sensitive actions Require current password before changing email or password
Use POST method (not GET) Combined with CSRF tokens; never use GET for state-changing actions

Secure Example:

```php
// Generate CSRF token
$_SESSION['csrf_token'] = bin2hex(random_bytes(32));

// In form
<input type="hidden" name="csrf_token" value="<?php echo $_SESSION['csrf_token']; ?>">

// Validation
if ($_POST['csrf_token'] !== $_SESSION['csrf_token']) { die("Invalid request"); }
```

---

3. Broken Authentication

Property Details
OWASP 2021 A07:2021 - Identification and Authentication Failures
Severity Critical
Exploitability Easy to Medium
Impact Full account takeover, identity theft, access to sensitive data

How It Works

The application implements identification and authentication mechanisms incorrectly, allowing attackers to compromise passwords, sessions, or authentication keys.

Detailed Attack Scenario (Account Takeover via Brute Force)

Step Description
1. Discovery Application allows unlimited login attempts with no CAPTCHA or delay
2. Dictionary attack Use a list of common passwords (123456, password, admin123) against a known account (e.g., admin@example.com)
3. Successful exploitation Password admin123 works after several attempts
4. Impact Full access to admin panel, user data, and permissions

Common Broken Authentication Flaws

· Insecure sessions – Session ID exposed in URL, too short, or predictable
· Weak password reset – Guessable security questions or short reset codes (e.g., 4 digits)
· Incomplete logout – Server-side session not invalidated
· Plaintext or weak password storage – MD5 without salt

How to Fix

Method Description
Strong password policy Minimum 8 characters, mix of cases/digits/symbols
Brute force protection Rate limiting + CAPTCHA after 3 failures + increasing delays
Secure sessions HttpOnly, Secure, SameSite cookies; invalidate session on logout
Multi-factor authentication (MFA) Mandatory for sensitive accounts (admins, banking)
Strong password hashing bcrypt or Argon2 with random salt (never plaintext or MD5)
Logout from all devices On password change or reset

Secure Password Storage (PHP - bcrypt):

```php
// Hashing for storage
$hashed_password = password_hash($_POST['password'], PASSWORD_BCRYPT);

// Verification
if (password_verify($_POST['password'], $hashed_password_from_db)) {
    // Login successful
}
```

---

Summary for Developers

Vulnerability Minimum Fix
SQL Injection Use prepared statements for every database query
CSRF Add CSRF token to each form that modifies data + SameSite=Lax cookies
Broken Authentication Enable CAPTCHA after 3 failed attempts + bcrypt + invalidate sessions on logout

---

Recommended Testing Tools

Vulnerability Tools
SQL Injection SQLmap, Burp Suite (Intruder)
CSRF OWASP CSRFTester, Burp Suite (Generate CSRF PoC)
Broken Authentication Hydra (Brute Force), OWASP ZAP, Nuclei

---

Disclaimer
 explicit written permission from the syst

---

End of Report ✓
