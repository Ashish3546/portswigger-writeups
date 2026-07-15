# 🛡️ PortSwigger Web Security Academy — Lab Writeups

A structured collection of my hands-on web application security learning and lab writeups from the **PortSwigger Web Security Academy**.

This repository documents not only the payloads used to solve each lab, but also the reasoning behind them — including the vulnerable context, source-to-sink data flow, browser behaviour, exploitation process, mistakes encountered, and key lessons learned.

> **Goal:** Build a strong practical understanding of web application vulnerabilities by focusing on *why* an exploit works, not just *what payload* solves the lab.

---

## 🎯 Learning Approach

For each vulnerability, I follow a simple methodology:

```text
Understand the application
        ↓
Identify attacker-controlled input
        ↓
Trace the data flow
        ↓
Identify the execution context
        ↓
Find the vulnerable sink
        ↓
Construct a context-specific payload
        ↓
Observe browser behaviour
        ↓
Document the complete reasoning
```

The core mental model used throughout the repository is:

```text
SOURCE → TRANSFORMATION → SINK → CONTEXT → EXECUTION
```

---

## 📚 Vulnerabilities Covered

### Cross-Site Scripting (XSS)

| # | Lab | Type | Status |
|---|-----|------|--------|
| 01 | Reflected XSS into HTML context with nothing encoded | Reflected XSS | ✅ Solved |
| 02 | Stored XSS into HTML context with nothing encoded | Stored XSS | ✅ Solved |
| 03 | DOM XSS in `document.write` sink using `location.search` | DOM XSS | ✅ Solved |
| 04 | DOM XSS in `innerHTML` sink using `location.search` | DOM XSS | ✅ Solved |
| 05 | DOM XSS in jQuery anchor `href` attribute sink | DOM XSS | ✅ Solved |
| 06 | DOM XSS in jQuery selector sink using a `hashchange` event | DOM XSS | ✅ Solved |

> More labs and vulnerability categories will be added as I progress through the Web Security Academy.

---

## 🧠 Concepts Learned So Far

Through these labs, I have explored several important browser and web-security concepts:

- Reflected, Stored, and DOM-based XSS
- HTML execution contexts
- HTML attributes and event handlers
- DOM sources and sinks
- `location.search`
- `location.hash`
- `URLSearchParams`
- `document.write()`
- `innerHTML`
- jQuery `$()` selector behaviour
- jQuery `.attr()`
- Anchor elements and the `href` attribute
- The `javascript:` URL scheme
- Browser events such as `onload`, `onerror`, and `hashchange`
- URL query strings and fragments
- `<iframe>` elements
- Exploit delivery using an attacker-controlled server
- The difference between executing JavaScript on an attacker-controlled page and exploiting a vulnerable origin
- Context-specific payload construction

---

## 🔬 Example: Thinking Beyond Payloads

A payload alone does not explain a vulnerability.

For example:

```html
<img src=x onerror=alert(1)>
```

The important questions are:

```text
Where did the input come from?
        ↓
How did the application process it?
        ↓
Which sink received it?
        ↓
Which parser interpreted it?
        ↓
Why did the event handler execute?
```

The same payload will not work in every context.

```text
HTML Context       → HTML elements and event handlers

JavaScript Context → JavaScript syntax and execution rules

URL Context        → URL parsing and schemes

DOM Context        → Depends on the source, sink, and browser behaviour
```

Understanding the **execution context** is therefore more important than memorising payloads.

---

## 🧩 DOM XSS Mental Model

DOM-based XSS can occur entirely within the victim's browser.

```text
Attacker-controlled data
        ↓
DOM Source
        ↓
Client-side JavaScript
        ↓
Optional transformation
        ↓
Dangerous DOM Sink
        ↓
Browser interprets the data
        ↓
JavaScript execution
```

Example:

```text
location.search
        ↓
JavaScript processing
        ↓
innerHTML
        ↓
HTML parser
        ↓
Event handler
        ↓
JavaScript execution
```

Another example:

```text
location.hash
        ↓
hashchange event
        ↓
jQuery selector sink
        ↓
HTML interpretation
        ↓
Event handler
        ↓
JavaScript execution
```

---

## 🌐 Understanding Exploit Delivery

Some vulnerabilities require more than manually entering a payload.

An attacker-controlled webpage can be used to deliver an exploit to a victim:

```text
Attacker-controlled webpage
        ↓
Victim visits the page
        ↓
Browser loads or interacts with
the vulnerable application
        ↓
Vulnerable application processes
attacker-controlled input
        ↓
JavaScript executes in the
vulnerable application's context
```

The key distinction is:

```text
Executing JavaScript on my own page
        ≠
Exploiting another website

Attacker-controlled input
executing through vulnerable code
        =
XSS
```

---

## 📂 Repository Structure

```text
PortSwigger-Web-Security-Academy/
│
├── README.md
│
└── XSS/
    ├── 01-reflected-xss-html-context.md
    ├── 02-stored-xss-html-context.md
    ├── 03-dom-xss-document-write.md
    ├── 04-dom-xss-innerhtml.md
    ├── 05-dom-xss-in-jquery-anchor-href-attribute.md
    └── 06-dom-xss-in-jquery-selector-sink-hashchange.md
```

The repository structure will expand as I progress into additional vulnerability categories.

---

## 📝 What Each Writeup Contains

Each lab writeup aims to include:

- Lab description
- Vulnerability type
- Vulnerable context
- Relevant source and sink
- Payload used
- Step-by-step exploitation process
- Explanation of why the payload works
- Browser behaviour involved
- Problems and failed attempts encountered
- Key concepts learned
- Mitigation strategies

---

## 🛠️ Tools & Technologies

- PortSwigger Web Security Academy
- Burp Suite
- Browser Developer Tools
- HTML
- JavaScript
- DOM APIs
- jQuery
- Git
- GitHub

---

## 🚀 Progress

```text
Cross-Site Scripting (XSS)
██████░░░░░░░░░░░░░░  In Progress
```

Current focus: **Cross-Site Scripting (XSS)**

The repository will continue to grow as I work through more labs covering areas such as:

- SQL Injection
- Cross-Site Request Forgery
- Authentication vulnerabilities
- Access control vulnerabilities
- Server-Side Request Forgery
- XML External Entity injection
- Path traversal
- File upload vulnerabilities
- Web cache vulnerabilities
- API testing
- Server-side vulnerabilities

---

## ⚠️ Disclaimer

This repository is created strictly for **educational purposes and authorised security testing**.

All documented exercises were performed in intentionally vulnerable environments provided by the **PortSwigger Web Security Academy**.

The techniques documented here should only be used against systems that you own or have explicit permission to test.

---

## 🔗 References

- PortSwigger Web Security Academy
- PortSwigger Web Security Learning Materials
- MDN Web Docs
- OWASP

---

## 👨‍💻 Author

**Chinthakuntlawar Ashish**

B.Tech — Cyber Security & Digital Forensics

Learning web application security through hands-on labs, vulnerability analysis, and practical documentation.

---

⭐ This repository documents my journey from understanding individual payloads to understanding the browser behaviour and application logic that make web vulnerabilities possible.
