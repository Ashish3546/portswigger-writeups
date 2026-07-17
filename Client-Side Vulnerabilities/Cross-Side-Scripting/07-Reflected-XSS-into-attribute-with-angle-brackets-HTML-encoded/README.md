# Lab: Reflected XSS into attribute with angle brackets HTML-encoded

## Lab Description

This lab contains a reflected Cross-Site Scripting (XSS) vulnerability in the search functionality. User input is reflected inside an HTML attribute where angle brackets (`<` and `>`) are HTML-encoded. The application does not properly sanitize attribute injection, allowing an attacker to inject a new event handler.

---

## Objective

Perform a reflected XSS attack by breaking out of the existing HTML attribute, injecting a new attribute, and executing JavaScript using the `alert()` function.

---

## Vulnerability Type

- Reflected Cross-Site Scripting (Reflected XSS)
- HTML Attribute Injection
- Event Handler Injection

---

# 🔗 Vulnerable Functionality

### Entry Point

Search Bar

### Source

User-controlled search parameter.

### Tainted Data

The search query is reflected into the `value` attribute of an HTML `<input>` element.

### Sink

```html
<input value="USER_INPUT">
```

### Context

HTML Attribute Context (`value` attribute)

---

# Methodology

### Step 1

Enter any text in the search bar.

Example:

```
test
```

Inspect the page using Developer Tools and locate where the input is reflected.

Observed HTML:

```html
<input
type="text"
placeholder="Search the blog..."
name="search"
value="test">
```

The user input is reflected inside the `value` attribute.

---

### Step 2

Since the payload is inside an attribute, first terminate the current attribute value using a double quote (`"`).

---

### Step 3

Inject a new HTML event attribute that can execute JavaScript.

The `onfocus` event was selected because the vulnerable element is an `<input>` element, which naturally supports the `focus` event.

---

### Step 4

Add the `autofocus` boolean attribute so that the browser automatically focuses the input when the page loads.

---

### Step 5

Use a dummy attribute (`x`) at the end to consume the application's trailing quotation mark and keep the final HTML syntactically valid.

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
<input value="USER_INPUT">
      │
      ▼
Break out of value attribute
      │
      ▼
Inject autofocus
      │
      ▼
Inject onfocus event
      │
      ▼
Browser automatically focuses input
      │
      ▼
JavaScript executes
```

---

# Payload

```text
" autofocus onfocus=alert(1) x="
```

---

# Result

The injected payload creates a new `onfocus` event handler inside the `<input>` element.

When the page loads, the `autofocus` attribute automatically focuses the input field, triggering the `onfocus` event and executing the JavaScript payload.

The lab is successfully solved.

---

# Key Learnings

- XSS payloads depend on the context in which user input is reflected.
- Angle bracket encoding prevents new HTML tags from being injected but does not prevent attribute injection.
- HTML boolean attributes (such as `autofocus`) do not require a value.
- Event handler attributes (`onfocus`, `onclick`, etc.) execute JavaScript without requiring `<script>` tags.
- HTML attribute values may be quoted or unquoted.
- The browser parses the final HTML document, not the payload in isolation.
- Properly balancing HTML attributes is essential for successful exploitation.

---

# Personal Notes

This lab introduced HTML attribute injection and demonstrated that escaping the current attribute is often sufficient to achieve code execution without injecting new HTML elements.

A key takeaway was understanding the difference between HTML context and HTML attribute context, as well as learning how browser HTML parsing affects payload construction.
