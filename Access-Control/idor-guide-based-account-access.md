# IDOR via GUID-Based User Identifier

## Platform
PortSwigger Web Security Academy

## Category
Access Control (IDOR / Horizontal Privilege Escalation)

## Lab Goal
- Find Carlos's GUID
- Access Carlos's account information
- Retrieve Carlos's API key

---

# Vulnerability Description

The application was vulnerable to an Insecure Direct Object Reference (IDOR).

Although user accounts were identified using GUIDs instead of predictable numeric IDs, the application failed to properly enforce authorization checks when accessing user account data.

As a result, an attacker could access another user's sensitive information by obtaining their GUID.

---

# Initial Analysis

After logging into the application using the provided credentials:

```text
wiener:peter
```

The following request was observed:

```http
GET /my-account?id=453c55f7-7537-4674-b864-767df79898dc HTTP/2
```

The application used GUIDs to identify users instead of usernames or sequential numeric IDs.

At first glance, this appeared more secure because GUIDs are difficult to guess directly.

---

# Enumerating User GUIDs

While browsing the blog section of the application, user profile references were discovered inside blog author links.

Observed URL:

```http
/blogs?userId=0dbb7330-2c98-4a47-8e76-e2176c860494
```

Inspecting the page source revealed that this GUID belonged to the user `carlos`.

Example snippet:

```html
<a href='/blogs?userId=0dbb7330-2c98-4a47-8e76-e2176c860494'>carlos</a>
```

This exposed Carlos's user identifier.

---

# Exploitation

The identified GUID was used to access Carlos's account page directly.

Example request:

```http
GET /my-account?id=0dbb7330-2c98-4a47-8e76-e2176c860494
```

The application returned Carlos's account information, including his API key.

Retrieved API key:

```text
DMM4qH59euJIITqDokcbbWjfFv3WJ3wP
```

---

# Impact

An attacker could exploit this vulnerability to:

- Access sensitive information belonging to other users
- Retrieve API keys or personal data
- Perform unauthorized actions on behalf of other users
- Compromise account confidentiality

In real-world applications, IDOR vulnerabilities can lead to severe data exposure and account compromise.

---

# Root Cause

The application relied solely on user-supplied object identifiers (GUIDs) without validating whether the authenticated user was authorized to access the requested resource.

Using GUIDs alone does not provide security if proper authorization checks are missing.

---

# Recommended Fixes

## 1. Enforce Authorization Checks
Validate that the authenticated user is authorized to access the requested resource before returning data.

## 2. Avoid Client-Controlled Object References
Do not rely solely on user-controlled identifiers for access decisions.

## 3. Implement Server-Side Access Control
Authorization must always be verified server-side.

## 4. Minimize Sensitive Data Exposure
Avoid exposing internal identifiers unnecessarily in URLs or page source.

---

# Tools Used

- Burp Suite (Intercept, Repeater)
- Browser Developer Tools
- Ubuntu Linux

---

# Key Takeaways

- GUIDs are not a replacement for authorization checks.
- IDOR vulnerabilities can still exist even when identifiers are difficult to guess.
- Information disclosure in page source or public endpoints can aid attackers in enumerating user identifiers.
- Proper server-side authorization validation is critical for preventing horizontal privilege escalation.
