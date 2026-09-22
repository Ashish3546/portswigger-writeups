# Authentication 🔐

This section contains my write-ups and practical solutions for authentication vulnerabilities from the **PortSwigger Web Security Academy**.

Authentication vulnerabilities can allow attackers to gain unauthorized access to user accounts, sensitive information, or functionality. The labs in this section focus on identifying weaknesses in authentication mechanisms and understanding how they can be exploited.

---

## 📚 Labs Completed

| **#** | **Lab** | **Difficulty** | **Status** |
|-------|---------|----------------|------------|
| 01 | [Username Enumeration via Different Responses](./01-Username-Enumeration-Via-Different-Responses) | Apprentice | ✅ Solved |
| 02 | [2FA Simple Bypass](./02-2FA-Simple-Bypass) | Apprentice | ✅ Solved |
| 03 | [Password Reset Broken Logic](./03-Password-Reset-Broken-Logic) | Apprentice | ✅ Solved |
| 04 | [Username Enumeration via Subtly Different Responses](./04-username-enumeration-subtly-different-responses) | Practitioner | ✅ Solved |
| 05 | [Username Enumeration via Response Timing](./05-username-enumeration-response-timing) | Practitioner | ✅ Solved |
| 06 | [Broken Brute-Force Protection, IP Block](./06-broken-bruteforce-protection-ip-block) | Practitioner | ✅ Solved |
| 07 | [Username Enumeration via Account Lock](./07-username-enumeration-via-account-lock) | Practitioner | ✅ Solved |
| 08 | [2FA Broken Logic — MFA Authentication Bypass](./08-2FA%20Broken%20Logic%20%E2%80%94%20MFA%20Authentication%20Bypass) | Practitioner | ✅ Solved |
| 09 | [Brute-Forcing a Stay-Logged-In Cookie](./Brute-Forcing-a-Stay-Logged-In-Cookie) | Practitioner | ✅ Solved |

---

## 🧠 Topics Covered

- Username Enumeration
- Password-Based Authentication
- Brute-Force Attacks
- Brute-Force Protection
- IP-Based Brute-Force Protection
- Account Locking
- Two-Factor Authentication
- MFA Authentication Bypass
- Authentication Logic Flaws
- Password Reset Vulnerabilities
- Session Persistence
- Stay-Logged-In Cookies
- Cookie-Based Authentication
- Response Analysis
- Response Timing Analysis

---

## 🛠️ Tools

- Burp Suite
- Burp Proxy
- Burp Repeater
- Burp Intruder
- Burp Comparer
- Burp Decoder
- PortSwigger Web Security Academy
- Wordlists
- HTTP Request/Response Analysis

---

## 🎯 Learning Focus

The goal of these labs is not only to complete the challenges but to understand:

- How authentication mechanisms work
- How authentication weaknesses can be identified
- How HTTP requests and responses reveal authentication behavior
- How usernames can be enumerated through response differences
- How response timing can reveal authentication information
- How brute-force attacks can be performed using Burp Intruder
- How brute-force protection mechanisms can be bypassed
- How account-locking mechanisms can introduce username enumeration flaws
- How two-factor authentication can fail due to incorrect implementation
- How password reset functionality can introduce authentication vulnerabilities
- How persistent authentication cookies can be abused
- How Burp Suite can be used to automate authentication testing
- How authentication vulnerabilities can be prevented

---

## 🔍 Key Techniques Practiced

### Username Enumeration

Identifying valid usernames by analyzing:

- Different response messages
- Response length
- Subtle response differences
- Response timing
- Account-lock behavior

### Brute-Force Testing

Using **Burp Intruder** to:

- Test candidate usernames
- Test candidate passwords
- Analyze response status codes
- Compare response lengths
- Extract specific response content
- Identify successful authentication attempts

### Authentication Bypass

Testing weaknesses in:

- Two-factor authentication
- MFA verification logic
- Password reset functionality
- Session persistence
- Authentication state validation

### Authentication Token Analysis

Analyzing:

- Session cookies
- Stay-logged-in cookies
- Cookie encoding
- Password-derived authentication tokens

---

## 📊 Lab Progress

**Total Labs Completed:** 9

**Authentication Techniques Practiced:** Multiple

**Primary Tool:** Burp Suite

**Training Platform:** PortSwigger Web Security Academy

---

## 📖 References

- [PortSwigger Web Security Academy — Authentication](https://portswigger.net/web-security/authentication)
- [PortSwigger — Vulnerabilities in Password-Based Login](https://portswigger.net/web-security/authentication/password-based)
- [PortSwigger — Vulnerabilities in Other Authentication Mechanisms](https://portswigger.net/web-security/authentication/other-mechanisms)
- [PortSwigger — Authentication Learning Path](https://portswigger.net/web-security/learning-paths/authentication-vulnerabilities)

---

⬅️ [Back to Server-Side Vulnerabilities](../README.md)
