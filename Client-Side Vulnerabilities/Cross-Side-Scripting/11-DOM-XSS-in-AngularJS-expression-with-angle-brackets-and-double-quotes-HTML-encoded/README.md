# Lab: DOM XSS in AngularJS Expression with Angle Brackets and Double Quotes HTML-Encoded

## Lab Description

This lab demonstrates a DOM-Based Cross-Site Scripting (DOM XSS) vulnerability caused by AngularJS expression injection. User-controlled input is reflected into an AngularJS template where angle brackets (`<`, `>`) and double quotes (`"`) are HTML-encoded. Although traditional HTML injection is prevented, AngularJS still evaluates expressions enclosed within `{{ }}`, allowing arbitrary JavaScript execution.

---

## Objective

Exploit the DOM XSS vulnerability by identifying the AngularJS expression context, leveraging an existing AngularJS scope object, and executing arbitrary JavaScript using the JavaScript `Function` constructor.

---

## Vulnerability Type

- DOM-Based Cross-Site Scripting (DOM XSS)
- Client-Side Template Injection (CSTI)
- AngularJS Expression Injection

---

# 🔗 Vulnerable Functionality

## Entry Point

- Search Bar

### Source

User-controlled search parameter.

### Tainted Data

The search query is reflected into an AngularJS expression without proper sanitization.

### Sink

AngularJS Expression Evaluation

```html
{{ USER_INPUT }}
```

### Context

AngularJS Expression Context

---

## Methodology

### Step 1

Enter a test value into the search bar and inspect the page using Developer Tools.

Locate where the input is reflected.

Observed HTML:

```html
<h1>0 search results for 'USER_INPUT'</h1>
```

This confirms that user-controlled input is reflected into the page.

---

### Step 2

Inspect the page source and identify that AngularJS is enabled.

Observed HTML:

```html
<body ng-app>
```

This indicates that AngularJS automatically evaluates expressions enclosed within `{{ }}`.

---

### Step 3

Test simple AngularJS expressions.

Example:

```text
{{2*3}}
```

Result:

```text
6
```

This confirms that AngularJS is evaluating expressions instead of rendering them as plain text.

---

### Step 4

Determine the filtering behavior.

Observations:

- Angle brackets (`<` and `>`) are HTML-encoded.
- Double quotes (`"`) are HTML-encoded.
- Arithmetic expressions are evaluated.
- Traditional HTML injection is ineffective.

This confirms that the vulnerability is not HTML injection but AngularJS expression injection.

---

### Step 5

Identify an existing AngularJS scope object.

The `$on` function is available within the AngularJS scope.

Its `constructor` property references JavaScript's `Function` constructor, allowing dynamic function creation.

---

### Step 6

Use the `Function` constructor to create and immediately execute arbitrary JavaScript.

---

## Data Flow

```text
Search Bar
      │
      ▼
User Input
      │
      ▼
AngularJS Template
      │
      ▼
AngularJS Expression Evaluation
      │
      ▼
$on.constructor
      │
      ▼
Function Constructor
      │
      ▼
Create JavaScript Function
      │
      ▼
Execute Function
      │
      ▼
Browser Executes JavaScript
```

---

## Payload

```javascript
{{$on.constructor('alert(1)')()}}
```

---

## Result

The payload is evaluated by AngularJS as an expression. The `$on` object's `constructor` references JavaScript's `Function` constructor, which dynamically creates a new function containing `alert(1)`. The trailing `()` immediately invokes the newly created function, successfully executing arbitrary JavaScript and solving the lab.

---

## Prevention

- Never evaluate user-controlled input as AngularJS templates.
- Avoid compiling or rendering untrusted templates using AngularJS.
- Treat user input as plain text instead of executable template expressions.
- Upgrade or migrate away from unsupported AngularJS versions where possible.
- Implement context-aware output encoding and input validation.
- Enforce a strong Content Security Policy (CSP) to reduce the impact of XSS vulnerabilities.

---

## Skills Practiced

- DOM XSS Identification
- Client-Side Template Injection (CSTI)
- AngularJS Expression Analysis
- Source & Sink Analysis
- Execution Context Analysis
- JavaScript Object Model
- Function Constructor Abuse
- Payload Construction
- Secure Coding Awareness

---

## Key Learnings

- Always identify the execution context before selecting a payload.
- AngularJS expressions are evaluated differently from normal HTML.
- HTML encoding alone does not prevent AngularJS expression injection.
- Client-Side Template Injection (CSTI) can lead to arbitrary JavaScript execution.
- Existing AngularJS scope objects may expose powerful JavaScript functionality.
- Understanding JavaScript's `constructor` property and `Function` constructor is essential for analyzing AngularJS expression injection.
- Successful exploitation depends on understanding framework behavior rather than memorizing payloads.

---

## Personal Notes

This lab reinforced that not every Cross-Site Scripting vulnerability relies on HTML injection. Even though angle brackets and double quotes were encoded, AngularJS still interpreted expressions inside `{{ }}`. By tracing the execution context, experimenting with simple expressions, and understanding how JavaScript's `Function` constructor can be reached through an existing AngularJS scope object, the vulnerability became much easier to understand. The biggest takeaway was that understanding the framework's expression engine is far more valuable than memorizing exploitation payloads.
