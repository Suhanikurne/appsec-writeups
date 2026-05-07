# Blind OS Command Injection via Output Redirection

## Platform
PortSwigger Web Security Academy

## Category
OS Command Injection

## Lab Goal
Execute the `whoami` command and retrieve its output.

---

# Vulnerability Description

The application was vulnerable to blind OS command injection in the feedback functionality.

User-supplied input was embedded directly into a shell command without proper sanitization.

Although command output was not displayed in the HTTP response, the vulnerability could still be exploited using:

- time-based detection
- output redirection techniques

The application also exposed a writable directory:

```text
/var/www/images/
```

This allowed command output to be redirected into a file that could later be retrieved through the image endpoint.

---

# Initial Analysis

The feedback form submitted multiple user-controlled parameters:

- name
- email
- subject
- message

The application returned:

```text
200 OK
```

without displaying command output directly.

This suggested a potential blind command injection scenario.

---

# Identifying the Vulnerable Parameter

## Testing Name Parameter

A delay payload was injected into the `name` parameter:

```text
|| ping -c 10 127.0.0.1 ||
```

The application responded normally with no delay.

Result:
- Parameter not vulnerable

---

## Testing Email Parameter

The same delay payload was injected into the `email` parameter:

```text
test@test.ca||ping -c 10 127.0.0.1||
```

The response was delayed by approximately 10 seconds.

This confirmed that the `email` parameter was vulnerable to command injection.

---

# Exploitation

Since the application did not return command output directly, output redirection was used.

The following payload redirected the output of the `whoami` command into a writable file:

```text
test@test.ca||whoami>/var/www/images/name.txt||
```

Injected request:

```http
email=test@test.ca||whoami>/var/www/images/name.txt||
```

The server executed the command and created:

```text
/var/www/images/name.txt
```

---

# Retrieving Command Output

The generated file was then accessed through the image retrieval functionality.

The contents of `name.txt` contained the result of the `whoami` command.

---

# Result

Successfully executed the `whoami` command and retrieved command output through output redirection.

---

# Impact

An attacker could exploit this vulnerability to:

- Execute arbitrary operating system commands
- Read sensitive files
- Access internal system information
- Potentially gain remote code execution
- Fully compromise the underlying server

OS command injection vulnerabilities are extremely dangerous and can lead to complete server compromise.

---

# Root Cause

The application directly embedded unsanitized user input into a shell command.

The server failed to:
- validate input
- sanitize shell metacharacters
- isolate user-controlled data from command execution context

---

# Recommended Fixes

## 1. Avoid Shell Command Execution
Do not use shell commands when safer internal APIs are available.

## 2. Sanitize User Input
Reject dangerous shell metacharacters such as:

```text
|
&
;
$
>
<
```

## 3. Use Safe APIs
Use parameterized process execution methods that separate commands from user input.

## 4. Restrict Writable Directories
Prevent applications from writing arbitrary files to publicly accessible locations.

## 5. Apply Principle of Least Privilege
Run application services with minimal operating system permissions.

---

# Tools Used

- Burp Suite (Intercept, Repeater)
- Browser
- Ubuntu Linux

---

# Key Takeaways

- Blind command injection can still be exploited even when output is not directly visible.
- Time-based payloads are useful for identifying vulnerable parameters.
- Output redirection can be used to exfiltrate command results.
- Publicly accessible writable directories significantly increase impact.
- OS command injection vulnerabilities can lead to full server compromise.
