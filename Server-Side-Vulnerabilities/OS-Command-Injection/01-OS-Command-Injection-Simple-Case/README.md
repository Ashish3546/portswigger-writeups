# 01 - OS Command Injection, Simple Case

## Platform
PortSwigger Web Security Academy

## Category
OS Command Injection

## Vulnerability
OS Command Injection / Shell Injection

## Difficulty
Apprentice

## Status
Solved

---

## Lab Description

This lab contains an OS command injection vulnerability in the product stock checker.

The application executes a shell command containing user-supplied product and store IDs and returns the raw output from the command in its response.

The objective was to execute the `whoami` command and determine the name of the current operating-system user.

---

## Concept

OS command injection occurs when an application incorporates user-controlled input into an operating-system command executed by the server.

If the application passes user input directly into a shell command without appropriate validation or safe command handling, an attacker may be able to inject additional commands.

The important distinction in this lab is between:

- HTTP parameter syntax
- Shell command syntax

For example, in:

`productId=1&storeId=1`

the `&` separates HTTP form parameters.

However, a shell metacharacter placed inside a parameter value can be interpreted later by the server-side shell.

In this lab, the `storeId` parameter was vulnerable to command injection.

---

## Testing

### Step 1 - Identify the Stock Checker

The lab provides a product page containing a stock-checking feature.

A product and store can be selected and the `Check stock` functionality can be used to generate a request.

---

### Step 2 - Intercept the Request

The stock-check request was intercepted using Burp Suite.

The original request body contained:

`productId=1&storeId=1`

The two parameters were:

- `productId`
- `storeId`

The `storeId` parameter was selected as the injection point.

---

### Step 3 - Send the Request to Repeater

The request was sent to Burp Suite Repeater so that the parameters could be modified and the server response could be observed.

The baseline request returned the normal stock-check result.

---

### Step 4 - Test Command Injection

The `storeId` value was modified to:

`1|whoami`

The resulting request body was:

`productId=1&storeId=1|whoami`

The `|` character acts as a shell command separator/pipeline operator.

This caused the server-side command execution context to process `whoami` as an additional command.

---

### Step 5 - Observe the Result

The server returned the output of the injected command in the HTTP response.

The `whoami` command identifies the operating-system account under which the command is being executed.

This confirmed that the application was executing user-controlled input as part of a shell command.

The lab was subsequently marked as solved.

---

## Exploitation Flow

`Browser`
→ `Stock Checker`
→ `HTTP Request`
→ `storeId` parameter
→ `Server-side shell command`
→ `Injected command`
→ `whoami`
→ `Command output returned in response`

---

## Why the Vulnerability Worked

The application incorporated user-supplied `productId` and `storeId` values into a shell command.

Because the `storeId` value was not sufficiently restricted from containing shell syntax, the `|` character could be interpreted by the shell.

This allowed the intended stock-check command and the injected `whoami` command to be processed in the same shell command context.

---

## Impact

OS command injection can allow an attacker to execute arbitrary operating-system commands with the privileges of the affected application process.

Depending on the privileges and environment of the server, this can potentially result in:

- Reading sensitive files
- Modifying application data
- Executing unauthorized system commands
- Accessing other systems reachable by the server
- Compromising the application or underlying server

The actual impact depends on the permissions available to the process executing the command.

---

## Mitigation

The preferred defense is to avoid calling operating-system commands from application-layer code when a safer API or library can perform the required operation.

If OS commands must be used with user-supplied input:

- Validate input against a strict allowlist.
- Accept only the expected format and characters.
- Reject shell metacharacters and unexpected whitespace where appropriate.
- Avoid constructing shell command strings from user input.
- Use APIs that execute a specific program with separate arguments instead of passing a complete command string through a shell.

Input validation should be treated as a defense layer rather than relying solely on escaping shell metacharacters.

---

## Tools Used

- Burp Suite
- Burp Proxy
- Burp Repeater
- PortSwigger Web Security Academy

---

## Evidence

The following screenshots document the exploitation process:

- `PS-06-01_OSCommandInjection_SimpleCase_LabPage.png` - Lab description and objective
- `PS-06-02_OSCommandInjection_StockChecker.png` - Product stock checker
- `PS-06-03_OSCommandInjection_Baseline_Request.png` - Original stock-check request
- `PS-06-04_OSCommandInjection_Whoami_Request.png` - Modified request containing the command injection
- `PS-06-05_OSCommandInjection_Solved.png` - Successful lab result

---

## Key Takeaways

- OS command injection occurs when user-controlled input reaches a shell command.
- Burp Suite can be used to intercept and modify HTTP parameters.
- HTTP parameter separators and shell command separators operate at different processing layers.
- A parameter can appear harmless at the HTTP layer but become dangerous when incorporated into a shell command.
- `whoami` can be used to identify the operating-system user executing the command.
- Burp Repeater is useful for manually testing command injection and observing the resulting response.

---

## References

- PortSwigger Web Security Academy - OS Command Injection
- PortSwigger Web Security Academy - OS Command Injection, Simple Case
