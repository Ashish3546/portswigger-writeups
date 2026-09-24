# Information Disclosure

This section contains my writeups for **Information Disclosure** labs from the PortSwigger Web Security Academy.

Information disclosure occurs when an application unintentionally reveals information that can be useful for understanding or attacking the application.

## Labs

| # | Lab | Difficulty | Status |
|---|---|---|---|
| 01 | [Authentication bypass via information disclosure](./01-Authentication-Bypass-via-Information-Disclosure/) | Apprentice | ✅ Solved |

---

## 01. Authentication Bypass via Information Disclosure

**Lab:** Authentication bypass via information disclosure  
**Difficulty:** Apprentice  
**Status:** ✅ Solved

This lab contains an authentication bypass vulnerability in the administration interface. The application relies on a custom HTTP header to determine whether a request originates from a local IP address.

The header name is not initially known, but it can be discovered through information disclosure using the HTTP `TRACE` method.

### Writeup

👉 [View the complete lab writeup](./01-Authentication-Bypass-via-Information-Disclosure/)

---

## Concepts Covered

- Information disclosure
- HTTP `TRACE` method
- Custom HTTP headers
- IP-based access control
- Authentication bypass
- Burp Suite Match and Replace
- Request header manipulation

## Tools Used

- Burp Suite
- PortSwigger Web Security Academy
- Web Browser

## Learning Outcome

The lab demonstrates how information unintentionally disclosed by an application can reveal details about its security mechanisms.

In this case, the `TRACE` response revealed the custom `X-Custom-IP-Authorization` header. The header was then modified to use `127.0.0.1`, allowing access to the administration interface and enabling deletion of the `carlos` user.

## Reference

PortSwigger Web Security Academy — Authentication bypass via information disclosure
