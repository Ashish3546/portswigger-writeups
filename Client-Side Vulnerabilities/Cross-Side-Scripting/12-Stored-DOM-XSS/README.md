# Lab: Stored DOM XSS

## Lab Description

This lab demonstrates a **Stored DOM-Based Cross-Site Scripting (DOM XSS)** vulnerability in the blog comment functionality. User-supplied comments are retrieved asynchronously using JavaScript, parsed from JSON, and rendered into the page. Due to improper HTML sanitization, an attacker can inject malicious HTML that executes when the stored comment is displayed.

---

## Objective

Exploit the stored DOM XSS vulnerability to execute the `alert()` function and solve the lab.

---

## Vulnerability Type

- Stored Cross-Site Scripting (Stored DOM XSS)

---

## Vulnerable Feature

Blog comment functionality.

---

## Entry Point

The **Comment Body** field in the comment submission form.

---

## Source

The application retrieves stored comments using an `XMLHttpRequest` and parses the JSON response.

```javascript
let comments = JSON.parse(this.responseText);
```

The user-controlled comment body is accessed through:

```javascript
comment.body
```

---

## Sink

The application inserts the processed comment directly into the DOM using `innerHTML`.

```javascript
commentBodyPElement.innerHTML = escapeHTML(comment.body);
```

Since `innerHTML` parses HTML, any HTML that bypasses the sanitizer is interpreted by the browser.

---

## Root Cause

The application attempts to sanitize user input using the following function:

```javascript
function escapeHTML(html) {
    return html.replace('<', '&lt;').replace('>', '&gt;');
}
```

The developer incorrectly uses JavaScript's `String.replace()` without a global replacement.

As a result:

- Only the **first** `<` character is escaped.
- Only the **first** `>` character is escaped.
- Any additional HTML tags remain unchanged.

This incomplete sanitization allows an attacker to bypass the filter.

---

## Exploitation Process

1. Submit a comment containing harmless angle brackets followed by a malicious HTML tag.
2. The first `<` and `>` are escaped by the sanitizer.
3. The remaining HTML tag is left untouched.
4. The browser parses the remaining HTML when assigned to `innerHTML`.
5. The injected event handler executes JavaScript.

---

## Payload

```html
<><img src=x onerror=alert(1)>
```

---

## Why the Payload Works

Input:

```html
<><img src=x onerror=alert(1)>
```

After sanitization:

```html
&lt;&gt;<img src=x onerror=alert(1)>
```

The harmless `<>` consumes the only two replacements performed by the sanitizer.

The `<img>` tag remains unescaped and is interpreted as HTML by `innerHTML`.

Since the image source is invalid, the `onerror` event executes `alert(1)`.

---

## Result

Successfully achieved Stored DOM XSS by bypassing the application's incomplete HTML sanitization and executing JavaScript when the stored comment was rendered.

---

## Key Learning

This lab demonstrates that sanitization is only effective when it correctly handles **every occurrence** of dangerous characters.

Important takeaways:

- `String.replace()` replaces only the first occurrence unless a global replacement is used.
- `innerHTML` should never receive untrusted input without proper context-aware sanitization.
- DOM XSS analysis should always follow the data flow:
  - **Source**
  - **Sanitization**
  - **Sink**
- Understanding how browsers parse HTML is more valuable than memorizing payloads.

---

## Skills Practiced

- DOM XSS Analysis
- JavaScript Source-to-Sink Tracing
- HTML Parsing Behavior
- Weak Sanitization Bypass
- Browser DOM Inspection
- Security Code Review
