# Lab: 09-brute-forcing-a-stay-logged-in-cookie

---

## Lab Description

This lab demonstrates a vulnerability in a persistent authentication mechanism.

The application allows users to remain logged in after closing their browser using a `stay-logged-in` cookie. The cookie is generated using a predictable combination of the username and an MD5 hash of the password.

Because the cookie can be generated offline for candidate passwords, an attacker can brute-force the victim's password by generating authentication cookies and testing them against the application.

---

## Objective

Brute-force Carlos's `stay-logged-in` cookie and gain access to his **My account** page.

### Provided Credentials

```text
Username: wiener
Password: peter
```

### Victim

```text
Username: carlos
```

---

## Vulnerability Type

**Weak Persistent Authentication / Brute-Forceable Authentication Cookie**

The persistent authentication cookie is deterministically generated from:

```text
Base64(username + ":" + MD5(password))
```

This makes it possible to generate valid candidate authentication cookies without knowing the victim's password directly.

---

## Vulnerable Feature

The vulnerable functionality is the application's:

```text
Stay logged in
```

feature.

The application uses two authentication mechanisms:

```text
Normal Login
    ↓
Session Cookie

Stay Logged In
    ↓
Persistent Authentication Cookie
```

The persistent authentication cookie is vulnerable because it is derived directly from the user's password.

---

## Entry Point

The vulnerable entry point is the `stay-logged-in` cookie sent with requests to authenticated pages.

Example:

```http
GET /my-account?id=carlos HTTP/2
Host: <LAB-ID>.web-security-academy.net
Cookie: stay-logged-in=<PAYLOAD>
```

The `stay-logged-in` value was selected as the Intruder payload position.

---

## Source

The attacker-controlled source is the candidate password supplied to Burp Intruder.

Each candidate password is transformed into a potential persistent authentication cookie.

```text
Candidate Password
        ↓
      MD5
        ↓
carlos:<MD5 hash>
        ↓
     Base64
        ↓
stay-logged-in Cookie
```

---

## Sink

The generated cookie is sent to the authenticated endpoint:

```http
GET /my-account?id=carlos
```

The application processes the cookie and determines whether the request should be authenticated as Carlos.

---

## Root Cause

The root cause is the use of a predictable password-derived value as a persistent authentication credential.

The application effectively generates the cookie using:

```text
Base64(username + ":" + MD5(password))
```

Base64 provides encoding rather than security, while MD5 is not suitable for securely storing passwords or constructing authentication credentials.

Because the cookie is deterministically derived from the password, an attacker can generate candidate cookies offline and test them against the application.

---

## Data Flow

```text
Candidate Password
        │
        ▼
    MD5 Hash
        │
        ▼
carlos:<MD5 hash>
        │
        ▼
   Base64 Encode
        │
        ▼
stay-logged-in Cookie
        │
        ▼
GET /my-account?id=carlos
        │
        ▼
Application validates cookie
        │
        ├── Invalid → 302 /login
        │
        └── Valid → 200 My Account
```

---

## Exploitation Process

### 1. Capture the Stay-Logged-In Cookie

Logged in as `wiener` with the **Stay logged in** option enabled and captured the `stay-logged-in` cookie.

The cookie initially appeared to be a random Base64 string.

---

### 2. Decode the Cookie

Base64 decoding revealed a structure similar to:

```text
wiener:<32-character-hash>
```

The 32-character hexadecimal value indicated that a hash function such as MD5 could be involved.

---

### 3. Identify the Hash

The known password was:

```text
peter
```

Calculating:

```text
MD5("peter")
```

produced the same hash contained in the cookie.

This confirmed that the cookie follows:

```text
Base64(username + ":" + MD5(password))
```

---

### 4. Prepare the Carlos Request

The request was changed to target Carlos:

```http
GET /my-account?id=carlos HTTP/2
Host: <LAB-ID>.web-security-academy.net
Cookie: stay-logged-in=§PAYLOAD§
```

The normal `session` cookie was removed.

This was important because the objective was to test the persistent authentication mechanism independently rather than use the existing authenticated session.

---

### 5. Configure Burp Intruder

The request was sent to **Burp Suite Intruder**.

Attack type:

```text
Sniper
```

Payload position:

```text
stay-logged-in=§PAYLOAD§
```

---

### 6. Configure Payload Processing

Three payload-processing rules were configured in the following order:

```text
1. Hash → MD5
2. Add Prefix → carlos:
3. Encode → Base64
```

Therefore, every candidate password was automatically transformed:

```text
Candidate Password
        ↓
      MD5
        ↓
carlos:<hash>
        ↓
     Base64
        ↓
Generated Cookie
```

---

### 7. Load Candidate Passwords

The candidate password list provided by PortSwigger was loaded into Intruder as a **Simple List** payload.

Burp automatically processed each candidate using the configured transformation chain.

---

### 8. Configure Grep-Match

To identify the successful request, the following authenticated-only string was configured under **Grep - Match**:

```text
Update email
```

A successful authentication would return Carlos's My Account page containing this functionality.

---

### 9. Analyze the Responses

Most candidate passwords produced:

```http
HTTP/2 302 Found
Location: /login
```

These indicated that the generated authentication cookie was invalid.

One request produced:

```text
Status: 200
Grep Match: Update email
```

The response also contained:

```text
Your username is: carlos
```

This confirmed successful authentication as Carlos.

---

## Payload

The original payload was the candidate password.

Burp Intruder processed each candidate using:

```text
MD5
  ↓
Add Prefix: carlos:
  ↓
Base64 Encode
```

The resulting value was then inserted into:

```http
Cookie: stay-logged-in=<GENERATED-VALUE>
```

No manual generation of individual cookies was required.

---

## Why the Attack Works

The attack works because the persistent authentication cookie is predictable.

Instead of using a random, server-generated authentication token, the application derives the token from information that can be reproduced when the username and a candidate password are known.

The process is:

```text
Password
   ↓
MD5
   ↓
Username + Hash
   ↓
Base64
   ↓
Authentication Cookie
```

Since the victim's username is known:

```text
carlos
```

an attacker can take every candidate password and generate a corresponding authentication cookie.

The application then reveals whether the candidate was correct through the response.

This converts the authentication mechanism into a brute-forceable credential.

---

## Result

The brute-force attack successfully identified the valid candidate password.

The successful request returned:

```text
HTTP/2 200 OK
```

and the response contained:

```text
Your username is: carlos
```

along with:

```text
Update email
```

This confirmed authenticated access to Carlos's **My account** page.

**Lab successfully solved.**

---

## Key Learning

- Base64 is an encoding mechanism, not encryption.
- A 32-character hexadecimal value can be an indication of an MD5 hash, which can then be verified against known input.
- Persistent authentication cookies should be unpredictable and independently generated.
- Password-derived authentication tokens can become vulnerable to offline brute-force attacks.
- Burp Intruder payload processing can automate multi-step transformations such as hashing, prefixing, and encoding.
- Removing unrelated authentication state can be important when testing a specific authentication mechanism.
- Response differences such as status codes, response length, and authenticated-only content can be used to identify successful requests.
- Grep-Match is useful for automatically identifying a successful result among many Intruder responses.

---

## Skills Practiced

- Web Authentication Testing
- Persistent Authentication Analysis
- Cookie Analysis
- Base64 Decoding
- MD5 Hash Identification
- Hash Verification
- Burp Suite Intruder
- Payload Processing
- Brute-Force Testing
- Grep-Match
- HTTP Request/Response Analysis
- Session vs Persistent Authentication
- Authentication Mechanism Analysis
