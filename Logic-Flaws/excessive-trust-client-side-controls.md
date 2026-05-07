# Excessive Trust in Client-Side Controls

## Platform
PortSwigger Web Security Academy

## Category
Business Logic Vulnerability

## Lab Goal
Purchase a "Lightweight l33t Leather Jacket" for an unintended price.

---

# Vulnerability Description

The application trusted client-controlled parameters during the purchasing workflow.

Instead of validating product pricing server-side, the application relied on values supplied by the client, allowing attackers to manipulate item prices before checkout.

This resulted in a business logic vulnerability that enabled unauthorized price modification.

---

# Provided Credentials

```text
wiener:peter
```

---

# Initial Analysis

While adding products to the cart, the request responsible for cart updates was intercepted using Burp Suite.

The intercepted request contained client-controlled pricing information for the selected product.

This indicated that pricing values were not securely enforced server-side.

---

# Exploitation

## Step 1 — Intercept Add-to-Cart Request

Using Burp Suite Proxy and Repeater:
- The add-to-cart request was intercepted
- Product pricing parameters were identified

---

## Step 2 — Manipulate Product Price

The product price was modified from:

```text
1337
```

to:

```text
1
```

The manipulated request was then forwarded to the server.

---

## Step 3 — Complete Purchase

Logged into the provided account:

```text
wiener:peter
```

The manipulated product was added to the cart successfully.

Checkout was completed using the modified pricing values.

---

# Result

Successfully purchased:

```text
11 Lightweight l33t Leather Jackets
```

for:

```text
$11 total
```

instead of the intended price of:

```text
$1337 per item
```

---

# Impact

An attacker could exploit this vulnerability to:

- Purchase products at unauthorized prices
- Manipulate financial transactions
- Cause direct business revenue loss
- Abuse application purchasing workflows

Business logic flaws can lead to severe financial impact even without traditional technical vulnerabilities.

---

# Root Cause

The application trusted client-side pricing values without validating them server-side.

Critical business logic decisions such as product pricing should never rely on client-controlled data.

---

# Recommended Fixes

## 1. Enforce Pricing Server-Side
Product prices must always be validated using trusted server-side data sources.

## 2. Never Trust Client-Controlled Values
Sensitive values such as:
- pricing
- discounts
- permissions
- account identifiers

must not be trusted from user requests.

## 3. Validate Checkout Workflow
Implement server-side verification during checkout to prevent manipulated transactions.

## 4. Monitor Suspicious Transactions
Flag abnormal pricing behavior and unauthorized modifications.

---

# Tools Used

- Burp Suite (Proxy, Repeater, Intercept)
- Browser
- Ubuntu Linux

---

# Key Takeaways

- Business logic vulnerabilities can have serious financial impact.
- Client-side validation alone is not a security control.
- Critical transaction data must always be validated server-side.
- Burp Suite is effective for identifying insecure trust relationships in application workflows.
