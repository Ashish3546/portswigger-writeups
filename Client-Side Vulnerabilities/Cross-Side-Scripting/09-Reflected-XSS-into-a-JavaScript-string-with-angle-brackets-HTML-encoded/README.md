# Lab: Reflected XSS into a JavaScript string with angle brackets HTML-encoded

## Lab Description

This lab contains a reflected Cross-Site Scripting (XSS) vulnerability where user input is reflected inside a JavaScript string. Although angle brackets (`<` and `>`) are HTML-encoded, the application does not properly sanitize JavaScript string termination, allowing an attacker to break out of the string and execute arbitrary JavaScript.

---

## Objective

Exploit the reflected XSS vulnerability by escaping the existing JavaScript string, injecting JavaScript code, and commenting out the remaining script to maintain valid syntax.

---

## Vulnerability Type

- Reflected Cross-Site Scripting (Reflected XSS)
- JavaScript String Injection
- Script Context Injection

---

# 🔗 Vulnerable Functionality

### Entry Point

Search Bar

### Source

User-controlled search parameter.

### Tainted Data

The search query is reflected inside a JavaScript string.

### Sink

```javascript
var searchTerms = 'USER_INPUT';
```

### Context

JavaScript String Context

---

# Methodology

### Step 1

Enter a test value in the search bar.

Inspect the page source or Developer Tools to locate where the input is reflected.

Observed JavaScript:

```javascript
var searchTerms = 'test';
```

This confirms that the input is reflected inside a JavaScript string.

---

### Step 2

Notice that angle brackets (`<` and `>`) are HTML-encoded, making HTML tag injection ineffective.

Instead of thinking in HTML, identify that the browser is currently parsing JavaScript.

---

### Step 3

Terminate the existing JavaScript string using a single quote (`'`).

---

### Step 4

End the current JavaScript statement with a semicolon (`;`).

---

### Step 5

Inject arbitrary JavaScript code.

---

### Step 6

Comment out the remainder of the original script using a single-line comment (`//`) to prevent syntax errors.

---

# Data Flow

```
Search Bar
      │
      ▼
HTTP Request
      │
      ▼
Server Response
      │
      ▼
JavaScript String
      │
      ▼
Close String (')
      │
      ▼
Terminate Statement (;)
      │
      ▼
Inject JavaScript
      │
      ▼
Comment Remaining Code (//)
      │
      ▼
Browser Executes Payload
```

---

# Payload

```text
'; alert(1)//
```

---

# Result

The payload successfully terminates the existing JavaScript string, injects a new JavaScript statement, and comments out the remaining script. The browser executes the injected `alert(1)` function, successfully solving the lab.

---

# Key Learnings

- Always identify whether the browser is parsing HTML or JavaScript before selecting a payload.
- HTML payloads such as `<script>` are ineffective inside JavaScript strings.
- Breaking out of a JavaScript string requires understanding JavaScript syntax rather than HTML parsing.
- A successful JavaScript string breakout typically involves:
  - Closing the string.
  - Ending the current statement.
  - Injecting malicious JavaScript.
  - Commenting out the remaining code.
- Browser parsing rules determine how user input is interpreted.

---

# Personal Notes

This lab reinforced one of the most important concepts in Cross-Site Scripting: **the payload depends entirely on the execution context**. Instead of attempting HTML injection, the vulnerability was exploited by thinking like a JavaScript parser. Escaping the string, terminating the statement, executing JavaScript, and commenting out the remaining code demonstrated how understanding parser behavior is more valuable than memorizing payloads.
