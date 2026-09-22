# Cross-Site Scripting (XSS) 🛡️

This section contains my detailed write-ups and practical solutions for **Cross-Site Scripting (XSS)** labs from the **PortSwigger Web Security Academy**.

Cross-Site Scripting vulnerabilities occur when an application processes attacker-controlled input in an unsafe way, allowing JavaScript to execute in a user's browser. The labs in this section focus on understanding different XSS types, identifying vulnerable contexts, tracing data from sources to sinks, and constructing context-specific payloads.

---

## 📂 Types Covered

- Reflected XSS
- Stored XSS
- DOM-Based XSS
- Reflected DOM XSS
- HTML Context XSS
- HTML Attribute Context XSS
- JavaScript Context XSS
- jQuery-Based XSS
- AngularJS Expression Injection

---

## ✅ Completed Labs

| **No.** | **Lab** | **Status** |
|---------|---------|------------|
| 01 | Reflected XSS into HTML context with nothing encoded | ✅ Solved |
| 02 | Stored XSS into HTML context with nothing encoded | ✅ Solved |
| 03 | DOM XSS in `document.write` sink using source `location.search` | ✅ Solved |
| 04 | DOM XSS in `innerHTML` sink using source `location.search` | ✅ Solved |
| 05 | DOM XSS in jQuery anchor `href` attribute sink using `location.search` | ✅ Solved |
| 06 | DOM XSS in jQuery selector sink using a `hashchange` event | ✅ Solved |
| 07 | Reflected XSS into attribute with angle brackets HTML-encoded | ✅ Solved |
| 08 | Stored XSS into anchor `href` attribute with double quotes HTML-encoded | ✅ Solved |
| 09 | Reflected XSS into a JavaScript string with angle brackets HTML-encoded | ✅ Solved |
| 10 | DOM XSS in `document.write` sink using source `location.search` inside a select element | ✅ Solved |
| 11 | DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded | ✅ Solved |
| 12 | Reflected DOM XSS | ✅ Solved |
| 13 | Stored DOM XSS | ✅ Solved |

---

## 🧠 Topics Covered

- Reflected XSS
- Stored XSS
- DOM-Based XSS
- Reflected DOM XSS
- Stored DOM XSS
- HTML Context
- HTML Attribute Context
- JavaScript String Context
- `document.write` Sink
- `innerHTML` Sink
- jQuery `href` Attribute Sink
- jQuery Selector Sink
- `hashchange` Event
- `location.search`
- URL-Based Sources
- DOM Sources and Sinks
- AngularJS Expressions
- Context-Aware Payload Construction
- HTML Encoding
- JavaScript Execution

---

## 🔄 Sources and Sinks

A major focus of these labs is understanding the flow of attacker-controlled data through an application.

### Sources

Sources are locations from which attacker-controlled data enters the application.

Examples covered in these labs include:

- `location.search`
- `location.hash`
- URL parameters
- User-controlled input
- Stored application data

### Sinks

Sinks are locations where attacker-controlled data is processed in a potentially unsafe way.

Examples covered include:

- `document.write`
- `innerHTML`
- jQuery selectors
- HTML attributes
- Anchor `href` attributes
- JavaScript strings
- AngularJS expressions

The general data flow is:

**Source → Tainted Data → Sink → JavaScript Execution**

DOM-based XSS commonly occurs when client-side JavaScript takes attacker-controlled data from a source and passes it to an unsafe sink. ([PortSwigger](https://portswigger.net/web-security/cross-site-scripting/dom-based))

---

## 🎯 Learning Objectives

Through these labs, I focused on understanding:

- The difference between reflected, stored, and DOM-based XSS
- How user-controlled input reaches vulnerable application contexts
- How to identify XSS sources and sinks
- How to determine the context in which input is reflected
- How HTML encoding affects payload construction
- How attribute contexts differ from HTML contexts
- How JavaScript string contexts can be exploited
- How `location.search` can act as a DOM XSS source
- How `location.hash` can be used as a DOM XSS source
- How `document.write` can introduce DOM XSS
- How `innerHTML` can introduce DOM XSS
- How jQuery functionality can introduce XSS
- How AngularJS expressions can introduce client-side injection
- How stored input can result in persistent XSS
- How reflected input can result in reflected XSS
- How to analyze client-side JavaScript for DOM-based vulnerabilities
- How to construct payloads according to the vulnerable context

---

## 🛠️ Tools Used

- Burp Suite
- Burp Proxy
- Burp Repeater
- Burp Intruder
- Burp Decoder
- Browser Developer Tools
- JavaScript
- PortSwigger Web Security Academy

---

## 🔍 Key Techniques Practiced

### Reflected XSS

Testing whether attacker-controlled input from an HTTP request is reflected into the application's immediate response and executed as JavaScript.

---

### Stored XSS

Testing whether malicious input is stored by the application and later rendered to users in an unsafe context.

Stored XSS is also known as persistent or second-order XSS. ([PortSwigger](https://portswigger.net/web-security/cross-site-scripting/stored))

---

### DOM-Based XSS

Tracing attacker-controlled data through client-side JavaScript to determine whether it reaches a dangerous DOM sink.

Examples practiced include:

- `location.search` → `document.write`
- `location.search` → `innerHTML`
- `location.search` → jQuery
- `location.hash` → jQuery selector
- URL source → client-side sink

---

### Context Identification

Identifying the exact location where attacker-controlled input appears, such as:

- HTML body
- HTML attribute
- `href` attribute
- JavaScript string
- DOM manipulation
- JavaScript framework expression

The correct payload depends heavily on the context in which the input is processed. ([PortSwigger](https://portswigger.net/web-security/cross-site-scripting/contexts))

---

## 🧪 Payload Analysis

The labs required testing and adapting payloads according to the vulnerable context.

Common proof-of-concept techniques included JavaScript execution through appropriate HTML elements, event handlers, attributes, and JavaScript contexts.

The purpose of the payloads was to demonstrate controlled JavaScript execution within the lab environment rather than simply relying on a single universal XSS payload.

PortSwigger recommends using simple JavaScript execution as a proof of concept when testing XSS and provides context-specific techniques for different injection locations. ([PortSwigger](https://portswigger.net/web-security/cross-site-scripting/exploiting))

---

## 🛡️ Prevention Concepts

The labs also helped reinforce common XSS defenses:

- Validate input where appropriate
- Encode user-controlled data before output
- Apply context-specific output encoding
- Avoid unsafe DOM sinks when possible
- Use safe DOM APIs instead of unsafe HTML insertion
- Implement an appropriate Content Security Policy
- Avoid passing untrusted data directly into JavaScript execution contexts
- Use framework-provided escaping mechanisms correctly

PortSwigger recommends output encoding based on the context in which data is inserted, along with appropriate input validation and Content Security Policy as an additional layer of defense. ([PortSwigger](https://portswigger.net/web-security/cross-site-scripting/preventing))

---

## 📊 Lab Progress

**Total Labs Completed:** 13

**Primary Vulnerability:** Cross-Site Scripting (XSS)

**Main Categories Practiced:**

- Reflected XSS
- Stored XSS
- DOM XSS
- Reflected DOM XSS
- Stored DOM XSS

**Primary Tools:** Burp Suite and Browser Developer Tools

**Training Platform:** PortSwigger Web Security Academy

**Difficulty Covered:** Apprentice

---

## 📖 References

- [PortSwigger Web Security Academy — Cross-Site Scripting](https://portswigger.net/web-security/cross-site-scripting)
- [PortSwigger — Reflected XSS](https://portswigger.net/web-security/cross-site-scripting/reflected)
- [PortSwigger — Stored XSS](https://portswigger.net/web-security/cross-site-scripting/stored)
- [PortSwigger — DOM-Based XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- [PortSwigger — XSS Contexts](https://portswigger.net/web-security/cross-site-scripting/contexts)
- [PortSwigger — Preventing XSS](https://portswigger.net/web-security/cross-site-scripting/preventing)

---

⬅️ [Back to Client-Side Vulnerabilities](../README.md)
