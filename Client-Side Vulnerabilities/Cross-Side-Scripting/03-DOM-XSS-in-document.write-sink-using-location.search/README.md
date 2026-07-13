# DOM XSS in `document.write` sink using source `location.search`

## Lab Description

This lab contains a DOM-based Cross-Site Scripting (XSS) vulnerability in the search query tracking functionality.

The application uses the `document.write()` function to write data obtained from `location.search` directly into the HTML document.

---

## Objective

Perform a DOM-based XSS attack that successfully calls the `alert()` function.

---

## Vulnerability Type

DOM-Based Cross-Site Scripting (DOM XSS)

---

## Vulnerable Functionality

Search Query Tracking

---

## Entry Point

Search Parameter (`?search=`)

---

## Source

`window.location.search`

---

## Tainted Data

Search query value

---

## Sink

`document.write()`

---

## Context

HTML Attribute Context (`src` attribute of an `<img>` element)

---

## Methodology

1. Inspected the client-side JavaScript.
2. Identified that the application read user-controlled input from `window.location.search`.
3. Traced the flow of the input into the `trackSearch()` function.
4. Observed that the value was written directly into the page using `document.write()`.
5. Determined that the input was inserted inside the `src` attribute of an `<img>` element.
6. Escaped the HTML attribute context and injected a JavaScript payload.
7. Successfully triggered the `alert()` function.

---

## Data Flow

```
URL
        ↓
window.location.search
        ↓
query
        ↓
trackSearch(query)
        ↓
document.write()
        ↓
HTML Attribute Context
        ↓
JavaScript Execution
```

---

## Payload

```html
"><script>alert()</script>
```

---

## Result

Successfully escaped the HTML attribute context and executed arbitrary JavaScript through the DOM-based XSS vulnerability.

---

## Key Learning

- DOM XSS occurs entirely within the browser.
- The application trusted data from `window.location.search`.
- Understanding the complete data flow from **Source → Tainted Data → Sink** made the vulnerability easy to identify.
- Because the payload was inserted inside an HTML attribute, escaping the attribute context was required before injecting JavaScript.

---

## Personal Notes

- This was my first lab where I traced the complete Source → Sink flow.
- Reading the client-side JavaScript made identifying the vulnerability much easier.
- The payload required escaping the HTML attribute before creating a new `<script>` element.
