# Referer-Based Access Control Bypass

## Platform
PortSwigger Web Security Academy

## Category
Access Control Vulnerability

## Lab Goal
- Exploit flawed access controls
- Promote the user `wiener` to administrator

---

# Vulnerability Description

The application enforced administrative functionality based on the `Referer` HTTP header instead of properly validating user privileges server-side.

Because the server trusted a client-controlled header for authorization decisions, administrative actions could be performed by unauthorized users.

---

# Initial Analysis

The application provided the following administrator credentials:

```text
administrator:admin
```

After logging in as the administrator, the admin panel functionality was analyzed.

An administrative request used to promote users was identified.

Example functionality:
- Promote a normal user to administrator

---

# Observing the Vulnerable Request

While promoting the user `carlos`, the following behavior was observed:

- The request included a trusted `Referer` header
- The server accepted the action based on this header
- No proper server-side authorization validation was enforced

This indicated that access control relied on client-controlled data.

---

# Exploitation

## Step 1 — Log In as Normal User

Credentials used:

```text
wiener:peter
```

---

## Step 2 — Capture Administrative Request

Using Burp Suite Repeater:
- The previously observed admin request was intercepted
- Session cookie was modified from administrator session to Wiener's session
- Username parameter was modified from `carlos` to `wiener`

The manipulated request was then replayed while preserving the trusted `Referer` header.

---

# Result

The application processed the request successfully and promoted the user `wiener` to administrator.

---

# Impact

An attacker could exploit this vulnerability to:

- Bypass authorization controls
- Gain administrative privileges
- Perform unauthorized administrative actions
- Fully compromise application security

Improper access control mechanisms can lead to complete privilege escalation.

---

# Root Cause

The application relied on the `Referer` HTTP header for authorization decisions.

HTTP headers such as `Referer` are fully controllable by the client and must never be trusted for enforcing security-sensitive functionality.

The application also failed to validate user privileges server-side before executing administrative actions.

---

# Recommended Fixes

## 1. Enforce Server-Side Authorization
All authorization checks must be validated server-side using authenticated user roles and permissions.

## 2. Do Not Trust Client-Controlled Headers
Headers such as `Referer` should never be used for access control decisions.

## 3. Implement Role-Based Access Control (RBAC)
Administrative actions should only be accessible to users with appropriate roles.

## 4. Log and Monitor Privilege Changes
Sensitive actions such as privilege escalation should be logged and monitored.

---

# Tools Used

- Burp Suite (Repeater)
- Browser
- Ubuntu Linux

---

# Key Takeaways

- Client-controlled headers are not reliable security controls.
- Access control validation must always occur server-side.
- Misconfigured authorization logic can result in privilege escalation vulnerabilities.
- Burp Suite Repeater is highly effective for testing access control flaws and replaying modified requests.
