# 2FA Simple Bypass

## Platform
PortSwigger Web Security Academy

## Category
Authentication Vulnerability

## Lab Goal
- Bypass two-factor authentication
- Access Carlos's account page

---

# Vulnerability Description

The application implemented two-factor authentication (2FA), but failed to properly enforce MFA verification before granting access to authenticated resources.

As a result, it was possible to bypass the second authentication step and directly access protected account functionality.

---

# Provided Credentials

## Attacker Account

```text
wiener:peter
```

## Victim Account

```text
carlos:montoya
```

---

# Initial Analysis

After logging in using the attacker account, the application redirected the user to a second authentication step requiring an MFA code.

The following behavior was observed:

- Successful username/password authentication created a partially authenticated session
- MFA verification occurred through a separate request
- Sensitive endpoints appeared accessible before MFA completion

This indicated a potential flaw in the authentication workflow.

---

# Exploitation

## Step 1 — Analyze MFA Flow

Logged in using:

```text
wiener:peter
```

The MFA verification request (`/login2`) was intercepted using Burp Suite.

---

## Step 2 — Attempt Victim Login

Logged out of the attacker account.

Then logged in using victim credentials:

```text
carlos:montoya
```

Before submitting the MFA code:
- The `/login2` request was intercepted
- The request was dropped

This prevented MFA verification from completing.

---

## Step 3 — Directly Access Protected Resource

After dropping the MFA request, the browser URL was manually changed to:

```http
/my-account
```

The application granted access to Carlos's account page without requiring valid MFA verification.

---

# Result

Successfully bypassed two-factor authentication and accessed Carlos's account page.

---

# Impact

An attacker could exploit this vulnerability to:

- Bypass MFA protections
- Access victim accounts without completing second-factor authentication
- Compromise account confidentiality
- Defeat an important layer of authentication security

MFA bypass vulnerabilities significantly weaken account security mechanisms.

---

# Root Cause

The application failed to properly validate MFA completion before granting access to authenticated resources.

The session created after password authentication was treated as sufficiently authenticated even though MFA verification had not yet been completed.

---

# Recommended Fixes

## 1. Enforce MFA Verification Server-Side
Access to protected resources should only be granted after successful MFA completion.

## 2. Use Distinct Authentication States
Applications should clearly separate:
- partially authenticated sessions
- fully authenticated sessions

## 3. Restrict Access Before MFA Completion
Users pending MFA verification should not be able to access sensitive endpoints.

## 4. Validate Session State Consistently
All sensitive routes should verify that MFA requirements have been satisfied.

---

# Tools Used

- Burp Suite (Intercept, Repeater)
- Browser
- Ubuntu Linux

---

# Key Takeaways

- MFA implementations can fail if authentication states are not enforced correctly.
- Partial authentication should never provide access to protected functionality.
- Authentication workflow flaws can completely undermine two-factor authentication security.
- Burp Suite is effective for analyzing multi-step authentication flows.
