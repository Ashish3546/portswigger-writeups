# DOM XSS in `innerHTML` sink using source `location.search`

## Lab Description

This lab contains a DOM-based Cross-Site Scripting (XSS) vulnerability in the blog search functionality.

The application uses the `innerHTML` property to insert data obtained from `location.search` directly into the page.

---

## Objective

Perform a DOM-based XSS attack that successfully calls the `alert()` function.

---

## Vulnerability Type

DOM-Based Cross-Site Scripting (DOM XSS)

---

## Vulnerable Functionality

Blog Search Functionality

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

`innerHTML`

---

## Context

HTML Context (inside a `<div>` element)

---

## Methodology

1. Tested the search functionality with normal input.
2. Inspected the client-side JavaScript.
3. Identified that the application read user-controlled input from `window.location.search`.
4. Observed that the search value was assigned directly to an element using `innerHTML`.
5. Initially tested a `<script>` payload but found that it did not execute when inserted through `innerHTML`.
6. Determined that an HTML element with an event handler could be used instead.
7. Injected an `<img>` element with an `onerror` event handler.
8. Successfully triggered the `alert()` function.

---

## Data Flow

```
URL
        ↓
window.location.search
        ↓
Search Query
        ↓
innerHTML
        ↓
HTML Context
        ↓
<img onerror>
        ↓
JavaScript Execution
```

---

## Payload

```html
<img src=x onerror=alert()>
```

---

## Result

Successfully executed arbitrary JavaScript through the DOM-based XSS vulnerability by injecting an HTML element with an event handler.

---

## Key Learning

- DOM XSS occurs when client-side JavaScript processes untrusted data and inserts it into the DOM using an unsafe sink.
- The `innerHTML` property parses input as HTML.
- Injecting a `<script>` element through `innerHTML` did not execute in this lab, but an HTML element with an event handler (`onerror`) successfully executed JavaScript.
- Understanding the behavior of the sink is essential for selecting an appropriate payload.

---

## Personal Notes

- Both DOM XSS labs used the same source: `window.location.search`.
- The difference was the sink:
  - `document.write()` required escaping the HTML attribute context.
  - `innerHTML` required an event-handler-based payload.
- Tracing **Source → Tainted Data → Sink** made it much easier to understand and solve DOM XSS.
