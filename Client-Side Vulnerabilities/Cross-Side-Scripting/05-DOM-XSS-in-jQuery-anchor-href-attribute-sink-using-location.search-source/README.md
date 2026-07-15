# DOM XSS in jQuery anchor href attribute sink using location.search source

## Lab Description

This lab contains a DOM-based Cross-Site Scripting (XSS) vulnerability in the submit feedback page functionality.

The application uses the `returnPath` query parameter to dynamically modify the `href` attribute of the back link using jQuery.

---

## Objective

Perform a DOM-based XSS attack that successfully executes the `alert(document.cookie)` function.

---

## Vulnerability Type

DOM-based Cross-Site Scripting (DOM XSS)

---

## 🔗 Vulnerable Functionality

### Entry Point

Submit Feedback page with a user-controlled `returnPath` query parameter.

### Source

`window.location.search`

### Tainted Data

Value of the `returnPath` query parameter extracted using `URLSearchParams`.

### Sink

jQuery `.attr()` method modifying the `href` attribute.

### Context

Anchor `href` attribute context (`javascript:` URL scheme).

---

## Methodology

1. Tested the submit feedback functionality and inspected the page URL.
2. Identified the user-controlled `returnPath` query parameter.
3. Examined the client-side JavaScript and found that it reads data from `window.location.search`.
4. Observed that `URLSearchParams` extracts the value of the `returnPath` parameter.
5. Identified that the value is passed directly into the jQuery `.attr()` method to modify the `href` attribute of the back link.
6. Recognized that the sink places attacker-controlled data inside an anchor `href` attribute.
7. Injected a `javascript:` URL as the value of the `returnPath` parameter.
8. Clicked the back link, causing the browser to execute the JavaScript code.
9. Successfully triggered the `alert(document.cookie)` function.

---

## Data Flow

```text
URL
↓
window.location.search
↓
URLSearchParams
↓
returnPath parameter
↓
jQuery .attr()
↓
href attribute
↓
javascript: URL
↓
User clicks the link
↓
JavaScript execution
```

---

## Payload

```text
javascript:alert(document.cookie)
```

---

## Result

Successfully executed `alert(document.cookie)` through the DOM-based XSS vulnerability by injecting a `javascript:` URL into the anchor element's `href` attribute and clicking the modified link.

---

## Key Learnings

- DOM XSS can occur when attacker-controlled data from `window.location.search` flows into an unsafe DOM sink.
- `URLSearchParams` is used to parse and retrieve query parameters from the current URL.
- The jQuery `.attr()` method can become a dangerous sink when untrusted input is used to modify security-sensitive attributes such as `href`.
- An anchor element's `href` attribute can accept the `javascript:` URL scheme, causing JavaScript to execute when the link is clicked.
- Reaching a sink does not always mean immediate execution; the execution mechanism depends on the context.
- In this lab, the payload required user interaction — clicking the modified back link — before the JavaScript executed.
- `javascript:document.cookie` evaluates the expression and returns the cookie string, but it does not call a function that visibly demonstrates JavaScript execution.
- `javascript:alert(document.cookie)` calls the `alert()` function, producing a visible effect and proving that JavaScript execution occurred.
- Understanding the complete flow is essential for analysing DOM XSS:

  `Source → Transformation → Sink → Context/Parser → Execution`

---

## Personal Notes

- The source was `window.location.search`, which contains the query string from the current URL.
- `URLSearchParams` extracted the attacker-controlled `returnPath` value.
- The vulnerable sink was the jQuery `.attr()` method because it placed the untrusted value directly into the `href` attribute.
- The important part was identifying the context of the sink: an anchor `href` attribute.
- Because the value was placed inside `href`, the `javascript:` URL scheme could be used to turn the link into an execution path.
- The JavaScript did not execute merely because the payload reached the `href` attribute; the link had to be clicked.
- Always trace DOM-based vulnerabilities as:

  `Source → Transformation → Sink → Context → Execution`
