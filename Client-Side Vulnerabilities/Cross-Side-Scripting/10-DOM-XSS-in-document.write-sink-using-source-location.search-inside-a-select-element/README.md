# Lab: DOM XSS in `document.write` Sink Using Source `location.search` Inside a Select Element

## Lab Description

This lab demonstrates a DOM-Based Cross-Site Scripting (DOM XSS) vulnerability where user-controlled input from the URL query parameter is written directly into the HTML document using the `document.write()` function. The input is reflected inside a `<select>` element without proper sanitization, allowing an attacker to escape the existing HTML context and inject executable HTML that results in JavaScript execution.

---

## Objective

Exploit the DOM XSS vulnerability by identifying the source and sink, escaping the existing HTML context, injecting executable HTML, and triggering JavaScript execution.

---

## Vulnerability Type

- DOM-Based Cross-Site Scripting (DOM XSS)
- HTML Context Injection
- Client-Side XSS

---

# 🔗 Vulnerable Functionality

## Entry Point

- `storeId` URL parameter

### Source

```javascript
location.search
```

### Tainted Data

The value of the `storeId` URL parameter is obtained from the browser URL and stored in a JavaScript variable before being written into the HTML document.

### Sink

```javascript
document.write()
```

### Context

HTML Context (Inside a `<select>` element)

---

## Methodology

### Step 1

Inspect the page source or Developer Tools to identify where the application reads user-controlled input.

Observed JavaScript:

```javascript
var store = (new URLSearchParams(window.location.search)).get('storeId');
```

This confirms that the application reads user input from the URL query parameter.

---

### Step 2

Locate where the application writes the user-controlled input into the page.

Observed JavaScript:

```javascript
document.write('<option selected>' + store + '</option>');
```

This confirms that attacker-controlled input is written directly into the HTML document using `document.write()`.

---

### Step 3

Determine the execution context.

The reflected input appears inside the following HTML structure:

```html
<select name="storeId">
    <option selected>USER_INPUT</option>
</select>
```

Since the browser is parsing HTML inside a `<select>` element, arbitrary HTML cannot be injected directly without first escaping the existing context.

---

### Step 4

Escape the current HTML context by closing the `<select>` element.

```html
</select>
```

This causes the browser to exit the existing HTML structure.

---

### Step 5

Inject executable HTML capable of executing JavaScript.

Instead of injecting plain text, inject an HTML element containing an event handler.

---

### Step 6

Use an `<img>` element with an invalid source. When the image fails to load, the `onerror` event executes the supplied JavaScript.

---

## Data Flow

```text
storeId URL Parameter
        │
        ▼
location.search
        │
        ▼
JavaScript Variable
        │
        ▼
document.write()
        │
        ▼
<option>
        │
        ▼
Escape </select>
        │
        ▼
Inject <img>
        │
        ▼
onerror Event
        │
        ▼
Browser Executes JavaScript
```

---

## Payload

```html
</select><img src=x onerror=alert(1)>
```

---

## Result

The payload successfully escapes the existing `<select>` element, injects a malicious `<img>` element, and triggers the `onerror` event. Since the image source is invalid, the browser executes the injected JavaScript, successfully solving the lab.

---

## Prevention

- Avoid using `document.write()` with untrusted input.
- Use safer DOM APIs such as `createElement()` and `textContent`.
- Validate and sanitize all user-controlled input before rendering.
- Apply context-aware output encoding based on the rendering context.
- Implement a strong Content Security Policy (CSP) as a defense-in-depth mechanism.

---

## Skills Practiced

- DOM XSS Identification
- Source & Sink Analysis
- HTML Context Analysis
- Browser HTML Parsing
- Payload Construction
- Event Handler Injection
- Client-Side Security Analysis
- Secure Coding Awareness

---

## Key Learnings

- Always identify the **Source**, **Sink**, and **Execution Context** before selecting a payload.
- DOM XSS vulnerabilities occur entirely within the browser and may not involve server-side processing.
- Understanding the HTML context is essential before crafting a payload.
- Escaping the existing HTML structure is often required before injecting executable content.
- HTML event handlers such as `onerror` can execute JavaScript without requiring `<script>` tags.
- Browser HTML parsing behavior determines how injected content is interpreted.

---

## Personal Notes

This lab reinforced the importance of understanding browser parsing behavior before attempting payload injection. Instead of immediately testing common payloads, tracing the complete client-side data flow from the source (`location.search`) to the sink (`document.write()`) made it possible to identify the execution context and construct the correct payload. The exercise demonstrated that successful DOM XSS exploitation depends more on understanding how browsers interpret HTML than on memorizing payloads.
