# Lab: Stored DOM XSS

## Lab Description

This lab demonstrates a **Stored DOM-Based Cross-Site Scripting (DOM XSS)** vulnerability in the blog comment functionality. User-supplied comments are stored by the application, retrieved asynchronously using JavaScript, and rendered into the page.

Due to an incomplete HTML sanitization routine, an attacker can inject HTML that is later interpreted by the browser, resulting in JavaScript execution.

---

## Objective

Exploit the stored DOM XSS vulnerability to execute the `alert()` function.

---

## Vulnerability Type

- Stored DOM-Based Cross-Site Scripting (DOM XSS)

---

## Vulnerable Feature

Blog comment functionality.

---

## Entry Point

Comment Body field.

---

## Source

The application retrieves stored comments using an asynchronous `XMLHttpRequest`.

```javascript
let comments = JSON.parse(this.responseText);
```

The attacker-controlled value is:

```javascript
comment.body
```

---

## Sink

The application inserts the processed comment into the page using:

```javascript
commentBodyPElement.innerHTML = escapeHTML(comment.body);
```

Since `innerHTML` parses HTML instead of treating it as plain text, any HTML that bypasses sanitization is rendered by the browser.

---

## Root Cause

The application attempts to sanitize user input using:

```javascript
function escapeHTML(html) {
    return html.replace('<', '&lt;')
               .replace('>', '&gt;');
}
```

However, JavaScript's `String.replace()` only replaces the **first occurrence** unless a global replacement is explicitly used.

As a result:

- Only the first `<` is escaped.
- Only the first `>` is escaped.
- All remaining HTML remains untouched.

This incomplete sanitization allows HTML injection.

---

## Data Flow

```
User Comment
      │
      ▼
Database
      │
      ▼
XMLHttpRequest
      │
      ▼
JSON.parse()
      │
      ▼
comment.body
      │
      ▼
escapeHTML()
      │
      ▼
innerHTML
      │
      ▼
Browser HTML Parser
      │
      ▼
JavaScript Execution
```

---

## Exploitation Process

1. Submit a crafted comment containing harmless angle brackets followed by a malicious HTML element.
2. The first pair of angle brackets is consumed by the weak sanitizer.
3. The remaining HTML tag bypasses the filter.
4. The browser parses the unescaped HTML through `innerHTML`.
5. The invalid image triggers the `onerror` event.
6. JavaScript executes.

---

## Payload

```html
<><img src=x onerror=alert(1)>
```

---

## Why the Payload Works

The sanitizer escapes only the first `<` and first `>`.

Input:

```html
<><img src=x onerror=alert(1)>
```

After sanitization:

```html
&lt;&gt;<img src=x onerror=alert(1)>
```

The harmless `<>` consumes the only replacements performed by the sanitizer.

The remaining `<img>` tag is interpreted as real HTML by `innerHTML`.

Because the image source is invalid, the `onerror` event executes JavaScript.

---

## Result

Successfully achieved Stored DOM XSS by bypassing incomplete HTML sanitization and abusing the `innerHTML` sink.

---

## Key Learning

This lab demonstrates that sanitization must correctly process **every occurrence** of dangerous characters.

Important takeaways:

- `String.replace()` replaces only the first occurrence by default.
- Weak sanitization is often worse than no sanitization because it creates a false sense of security.
- `innerHTML` should never receive untrusted input without proper context-aware sanitization.
- Browser parsing behavior is essential for understanding DOM XSS.
- Always analyze DOM vulnerabilities using the Source → Sanitizer → Sink methodology.

---

## Skills Practiced

- Stored DOM XSS Analysis
- JavaScript Security Review
- Source-to-Sink Tracing
- HTML Parser Behavior
- Weak Sanitization Bypass
- Browser DOM Inspection
- DOM-Based Exploitation
