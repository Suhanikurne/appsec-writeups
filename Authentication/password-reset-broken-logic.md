# Password Reset Broken Logic

## Platform
PortSwigger Web Security Academy

## Category
Authentication Vulnerability

## Lab Goal
- Reset Carlos's password
- Log in as Carlos
- Access Carlos's account page

---

# Vulnerability Description

The application was vulnerable to a broken password reset logic flaw.

During the password reset process, the application trusted a user-controlled username parameter without properly validating whether the password reset token belonged to that user.

As a result, an attacker could modify the username parameter and reset another user's password.

---

# Provided Information

## Attacker Credentials

```text
wiener:peter
```

## Victim Username

```text
carlos
```

---

# Initial Analysis

A password reset was initiated for the attacker account (`wiener`).

While intercepting the password reset workflow using Burp Suite, a POST request containing the following parameters was observed:

- reset token
- username
- new password

This suggested that the application relied on client-supplied parameters during the password reset process.

---

# Exploitation

## Step 1 — Initiate Password Reset

Started password reset flow for:

```text
wiener
```

---

## Step 2 — Intercept Reset Request

Using Burp Suite Intercept, the password reset POST request was captured.

The request contained:
- valid reset token
- username parameter
- new password field

---

## Step 3 — Modify Username Parameter

The username parameter was modified from:

```text
wiener
```

to:

```text
carlos
```

A new password was then supplied.

---

## Step 4 — Submit Modified Request

The manipulated request was forwarded to the server.

The application accepted the request and reset Carlos's password using Wiener's valid reset token.

---

## Step 5 — Log In as Carlos

Successfully authenticated using:
- username: `carlos`
- attacker-controlled password

Carlos's account page was then accessed successfully.

---

# Result

Successfully reset Carlos's password and gained unauthorized access to his account.

---

# Impact

An attacker could exploit this vulnerability to:

- Reset passwords for arbitrary users
- Take over victim accounts
- Bypass normal authentication controls
- Fully compromise user accounts

Broken password reset logic can directly lead to account takeover.

---

# Root Cause

The application failed to properly bind the password reset token to the intended user account.

Instead of validating the relationship between:
- reset token
- authenticated reset flow
- target username

the server trusted a client-controlled username parameter.

---

# Recommended Fixes

## 1. Bind Reset Tokens to Specific Users
Password reset tokens must only be valid for the user who initiated the reset request.

## 2. Avoid Trusting Client-Controlled Parameters
Critical account actions should never rely solely on client-supplied identifiers.

## 3. Validate Reset Workflow Server-Side
The server must verify:
- token ownership
- token expiration
- intended target account

## 4. Use Single-Use Reset Tokens
Reset tokens should expire immediately after successful use.

---

# Tools Used

- Burp Suite (Intercept, Repeater)
- Browser
- Ubuntu Linux

---

# Key Takeaways

- Password reset functionality is a high-value target for attackers.
- Logic flaws in authentication workflows can lead to full account takeover.
- Security-sensitive actions must never rely on untrusted client-controlled parameters.
- Proper server-side validation is essential during password reset operations.
