# OS Command Injection

This section contains my writeups for **OS Command Injection** labs from the PortSwigger Web Security Academy.

OS Command Injection occurs when an application passes user-controlled input to an operating system command without properly preventing command manipulation. An attacker may be able to inject additional commands that are executed by the underlying operating system.

## Labs

| # | Lab | Difficulty | Status |
|---|---|---|---|
| 01 | [OS command injection, simple case](./01-OS-Command-Injection-Simple-Case/) | Apprentice | ✅ Solved |

---

## 01. OS Command Injection, Simple Case

**Lab:** OS command injection, simple case  
**Difficulty:** Apprentice  
**Status:** ✅ Solved

This lab contains an OS command injection vulnerability in the product stock checker. The application executes a shell command containing user-supplied product and store IDs and returns the raw output from the command in the response.

The objective is to execute the `whoami` command and determine the operating-system user under which the application is running.

### Writeup

👉 [View the complete lab writeup](./01-OS-Command-Injection-Simple-Case/)

---

## Concepts Covered

- OS Command Injection
- User-controlled input reaching an OS command
- Shell command separators
- Command execution through application parameters
- Using `whoami` to identify the current operating-system user
- Testing command injection using Burp Suite Repeater

## Tools Used

- Burp Suite
- PortSwigger Web Security Academy
- Web Browser

## Learning Outcome

The lab demonstrates how an application can become vulnerable when user-controlled parameters are incorporated into operating-system commands without sufficient input validation or command execution safeguards.

The practical testing process involved intercepting the stock-check request, modifying the user-controlled `storeId` parameter, injecting a command, and observing the command output returned by the application.

## Reference

[PortSwigger Web Security Academy — OS Command Injection](https://portswigger.net/web-security/os-command-injection)
