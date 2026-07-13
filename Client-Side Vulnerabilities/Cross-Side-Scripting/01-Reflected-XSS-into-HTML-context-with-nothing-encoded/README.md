# Reflected XSS into HTML context with nothing encoded

## Lab Description

This lab contains a simple reflected Cross-Site Scripting (XSS) vulnerability in the search functionality.

The objective is to execute JavaScript by exploiting the reflected search parameter.

---

## Objective

Perform a reflected XSS attack that successfully calls the `alert()` function.

---

## Vulnerability Type

Reflected Cross-Site Scripting (Reflected XSS)

---

## Vulnerable Feature

Search Functionality

---

## Entry Point

Search Parameter

---

## Context

HTML Context (between HTML tags)

---

## Methodology

1. Tested the search functionality using normal input.
2. Observed that the supplied input was immediately reflected in the response.
3. Verified that no HTML encoding was applied.
4. Determined that the reflection occurred inside an HTML context.
5. Injected a JavaScript payload.
6. Successfully triggered the `alert()` function.

---

## Payload

```html
<script>alert()</script>
```

---

## Result

Successfully executed arbitrary JavaScript through the reflected search parameter, confirming the presence of a reflected XSS vulnerability.

---

## Key Learning

- Reflected XSS occurs when user input is immediately reflected back in the HTTP response without proper output encoding.
- Always identify where the input is reflected before selecting a payload.
- Understanding the HTML context is critical for constructing successful XSS payloads.

---

## Personal Notes

- Reflection occurred inside an HTML context.
- No output encoding was present.
- The `<script>` payload executed successfully.
