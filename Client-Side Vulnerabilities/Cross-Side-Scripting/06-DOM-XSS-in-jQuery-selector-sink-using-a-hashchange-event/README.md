# DOM XSS in jQuery selector sink using a hashchange event

## Lab Description

This lab contains a DOM-based Cross-Site Scripting (XSS) vulnerability in the home page functionality.

The application uses the URL fragment (`location.hash`) to find and scroll to a blog post whose title contains the supplied value. The attacker-controlled fragment is passed into the jQuery `$()` selector function.

---

## Objective

Perform a DOM-based XSS attack that successfully executes the `print()` function in the victim's browser.

---

## Vulnerability Type

DOM-based Cross-Site Scripting (DOM XSS)

---

## 🔗 Vulnerable Functionality

### Entry Point

URL fragment (`#`) on the home page.

### Source

`window.location.hash`

### Tainted Data

The attacker-controlled value extracted from the URL fragment using `.slice(1)` and processed using `decodeURIComponent()`.

### Sink

jQuery `$()` selector function.

### Context

jQuery selector context where attacker-controlled input can be interpreted as HTML.

---

## Methodology

1. Examined the client-side JavaScript running on the home page.
2. Identified a `hashchange` event handler attached to the `window` object.
3. Observed that the application reads attacker-controlled data from `window.location.hash`.
4. Identified that `.slice(1)` removes the leading `#` character from the fragment.
5. Observed that `decodeURIComponent()` decodes the fragment value before it is used.
6. Identified the jQuery `$()` function as the sink receiving the attacker-controlled input.
7. Recognized that the vulnerable code only executes when a `hashchange` event occurs.
8. Used an `<iframe>` to first load the vulnerable page and then modify its URL fragment after the iframe loaded.
9. Appended an `<img>` element with an `onerror` event handler to the iframe URL.
10. Changing the fragment triggered the `hashchange` event and caused the payload to reach the jQuery selector sink.
11. The invalid image source triggered the `onerror` event and successfully executed the `print()` function.

---

## Data Flow

```text
Attacker-controlled URL fragment
↓
window.location.hash
↓
.slice(1)
↓
decodeURIComponent()
↓
jQuery $()
↓
HTML interpretation
↓
<img src=x onerror=print()>
↓
Image loading fails
↓
onerror event fires
↓
JavaScript execution
```

---

## Payload

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src += '<img src=x onerror=print()>'"></iframe>
```

---

## Result

Successfully executed the `print()` function through the DOM-based XSS vulnerability by using an iframe to trigger a `hashchange` event and deliver an HTML payload to the vulnerable jQuery `$()` selector sink.

---

## Key Learnings

- `window.location.hash` can act as an attacker-controlled source in DOM-based vulnerabilities.
- The URL fragment is the part of the URL that appears after the `#` character.
- `.slice(1)` removes the leading `#` from the value returned by `window.location.hash`.
- `decodeURIComponent()` decodes URL-encoded characters before the value reaches the sink.
- The jQuery `$()` function can become a dangerous sink when attacker-controlled input is interpreted as HTML.
- The vulnerable code in this lab was registered inside a `hashchange` event handler.
- A payload reaching a vulnerable sink is not always enough; the code path containing the sink must also be triggered.
- The iframe was used to first load the vulnerable page and then change its URL fragment.
- Changing the fragment after the page loaded triggered the `hashchange` event.
- In the iframe's `onload` handler, `this` refers to the iframe element.
- Modifying `this.src` changed the iframe's URL and therefore changed the fragment of the loaded page.
- The `<img>` element used an invalid `src` value so that image loading failed and triggered the `onerror` event.
- The `onerror` event then executed the required `print()` function.
- Understanding the complete flow is essential for analysing DOM XSS:

  `Source → Transformation → Sink → Context/Parser → Execution`

---

## Personal Notes

- The source was `window.location.hash`.
- The attacker-controlled fragment was processed using `.slice(1)` and `decodeURIComponent()` before reaching the sink.
- The sink was the jQuery `$()` selector function.
- The important part of this lab was not only finding the source and sink but also understanding when the vulnerable code executes.
- The vulnerable code only ran when the browser fired a `hashchange` event.
- Simply placing the payload in the fragment was not enough for the final exploit because the required event needed to be triggered in the victim's browser.
- The iframe created the required sequence:

  `Load page → iframe onload → change fragment → hashchange → vulnerable code runs → payload reaches sink → execution`

- `this.src += ...` modifies the iframe's own `src` because `this` refers to the iframe inside its `onload` event handler.
- This lab showed that DOM XSS analysis requires understanding both the data flow and the event flow.
- Always trace DOM-based vulnerabilities as:

  `Source → Transformation → Sink → Context → Trigger → Execution`
