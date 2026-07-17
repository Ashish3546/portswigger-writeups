# Lab: Stored XSS into anchor `href` attribute with double quotes HTML-encoded

## Lab Description

This lab contains a stored Cross-Site Scripting (XSS) vulnerability in the comment functionality. User input from the website field is stored by the application and later reflected inside the `href` attribute of an anchor (`<a>`) element. Although double quotes are HTML-encoded, the application does not restrict dangerous URI schemes, allowing JavaScript execution when the link is clicked.

---

## Objective

Exploit the stored XSS vulnerability by supplying a malicious `javascript:` URI in the website field so that clicking the author's name executes JavaScript.

---

## Vulnerability Type

- Stored Cross-Site Scripting (Stored XSS)
- Dangerous URI Scheme Injection
- HTML Attribute Injection (`href`)

---

# 🔗 Vulnerable Functionality

### Entry Point

Comment Form → Website Field

### Source

User-controlled website parameter.

### Tainted Data

The supplied website URL is stored by the application and later inserted into the `href` attribute of the author's profile link.

### Sink

```html
<a id="author" href="USER_INPUT">
    Username
</a>
```

### Context

HTML Attribute Context (`href` attribute)

---

# Methodology

### Step 1

Navigate to the comment section of the blog post.

---

### Step 2

Inspect how the submitted comment is rendered.

Without providing a website:

```html
<p>
    <img src="...">
    Ashish
</p>
```

After providing a website:

```html
<a id="author" href="https://example.com">
    Ashish
</a>
```

This confirms that the website field controls the value of the `href` attribute.

---

### Step 3

Instead of supplying a normal URL, provide a JavaScript URI.

---

### Step 4

Submit the comment.

The malicious URI is stored by the application.

---

### Step 5

Click the author's name.

The browser interprets the `javascript:` URI and executes the embedded JavaScript.

---

# Data Flow

```
Website Field
      │
      ▼
Application Stores Input
      │
      ▼
Comment Rendered
      │
      ▼
<a href="USER_INPUT">
      │
      ▼
User Clicks Author Name
      │
      ▼
Browser Executes javascript: URI
      │
      ▼
alert(1)
```

---

# Payload

```text
javascript:alert(1)
```

---

# Result

The malicious `javascript:` URI is stored in the application's database. When another user (or the attacker) clicks the author's name, the browser executes the JavaScript payload, successfully triggering an alert and solving the lab.

---

# Key Learnings

- Stored XSS persists on the server until the content is removed.
- The `href` attribute can execute JavaScript through the `javascript:` URI scheme.
- Not every XSS attack requires injecting HTML tags or event handlers.
- The browser follows the URI scheme specified in the `href` attribute.
- Different HTML attributes have different security implications and require different payloads.

---

# Personal Notes

This lab reinforced the importance of identifying the HTML context before choosing a payload. Instead of escaping the attribute or injecting event handlers, the vulnerability was exploited by abusing the browser's support for the `javascript:` URI scheme. It also demonstrated how a previously learned concept from an earlier reflected XSS lab could be applied directly to a stored XSS scenario.
