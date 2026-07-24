# Lab: Reflected DOM XSS

## Lab Description

This lab demonstrates a **Reflected DOM-Based Cross-Site Scripting (DOM XSS)** vulnerability. User-controlled input is reflected by the server inside a JSON response, retrieved asynchronously using an `XMLHttpRequest`, and executed by JavaScript using the dangerous `eval()` function.

Instead of safely parsing the JSON response, the application evaluates it as JavaScript, allowing an attacker to break out of the intended string context and execute arbitrary JavaScript.

---

## Objective

Exploit the reflected DOM XSS vulnerability to execute the `alert()` function.

---

## Vulnerability Type

- Reflected DOM-Based Cross-Site Scripting (DOM XSS)

---

## Vulnerable Feature

Search functionality.

---

## Entry Point

Search parameter supplied through the URL.

```
?search=
```

---

## Source

The application retrieves search results using an asynchronous `XMLHttpRequest`.

```javascript
xhr.open("GET", "/search-results" + window.location.search);
```

The server reflects the user-supplied search parameter inside a JSON response.

```json
{
    "results": [],
    "searchTerm": "USER_INPUT"
}
```

The attacker-controlled value is therefore:

```javascript
searchTerm
```

---

## Sink

Instead of parsing the JSON safely, the application executes it using:

```javascript
eval('var searchResultsObj = ' + this.responseText);
```

Since `eval()` interprets the supplied string as JavaScript code, attacker-controlled input can break out of the intended context and execute arbitrary JavaScript.

---

## Root Cause

The application incorrectly uses `eval()` to process server responses.

The server attempts to escape quotation marks using backslashes.

However, by carefully manipulating escape characters, an attacker can change how the JavaScript parser interprets the string.

This results in JavaScript execution instead of harmless data parsing.

---

## Data Flow

```
User Input
      │
      ▼
URL Search Parameter
      │
      ▼
Server JSON Response
      │
      ▼
XMLHttpRequest
      │
      ▼
eval()
      │
      ▼
JavaScript Execution
```

---

## Exploitation Process

1. Inject a payload into the search parameter.
2. The server reflects the payload inside a JSON string.
3. JavaScript receives the response through an `XMLHttpRequest`.
4. The response is passed directly into `eval()`.
5. By escaping the JavaScript string correctly, the payload breaks out of the original context.
6. Arbitrary JavaScript executes inside the victim's browser.

---

## Payload

```text
\"-alert(1)}//
```

---

## Why the Payload Works

The server escapes quotation marks before returning the JSON response.

The injected backslash changes how JavaScript interprets the escaped quote.

This prematurely terminates the original string, allowing the remaining payload to be interpreted as executable JavaScript.

Finally, the trailing `//` comments out the remaining code to prevent syntax errors.

---

## Result

Successfully achieved Reflected DOM XSS by abusing unsafe JavaScript execution through `eval()`.

---

## Key Learning

This lab highlights why `eval()` should never be used to process untrusted data.

Important takeaways:

- `eval()` executes JavaScript instead of parsing data.
- JSON should always be processed using `JSON.parse()`.
- Browser parsing behavior is as important as server-side escaping.
- DOM XSS should always be analyzed using a Source → Sink approach.
- Understanding JavaScript execution contexts is more valuable than memorizing payloads.

---

## Skills Practiced

- DOM XSS Analysis
- Source-to-Sink Tracing
- JavaScript String Context Analysis
- JSON Reflection Analysis
- Escape Character Manipulation
- Burp Suite Response Inspection
- Browser DevTools Debugging
