# Stored XSS into HTML context with nothing encoded

## Lab Description

This lab contains a stored Cross-Site Scripting (XSS) vulnerability in the comment functionality.

The objective is to execute JavaScript when a user views the blog post.

---

## Objective

Perform a stored XSS attack that successfully calls the `alert()` function when the blog post is viewed.

---

## Vulnerability Type

Stored Cross-Site Scripting (Stored XSS)

---

## Vulnerable Functionality

Comment Functionality

---

## Entry Point

Comment Submission Form

---

## Context

HTML Context (between HTML tags)

---

## Methodology

1. Submitted a normal comment to observe how user input was processed.
2. Verified that the submitted comment was permanently stored by the application.
3. Observed that the stored comment was rendered inside the HTML without output encoding.
4. Determined that arbitrary HTML and JavaScript could be injected.
5. Submitted a JavaScript payload through the comment form.
6. Reloaded the blog post and confirmed successful execution of the payload.

---

## Payload

```html
<script>alert()</script>
```

---

## Result

Successfully executed arbitrary JavaScript whenever the blog post containing the malicious comment was viewed, confirming a stored XSS vulnerability.

---

## Key Learning

- Stored XSS occurs when malicious input is permanently stored by the application and later executed in users' browsers.
- Unlike reflected XSS, the victim does not need to submit the payload themselves; simply viewing the affected page is sufficient.
- Identifying where user input is stored and later rendered is essential during testing.

---

## Personal Notes

- The payload was stored in the application's comment section.
- The script executed every time the blog post was viewed.
- The vulnerability existed because user input was rendered without output encoding.
