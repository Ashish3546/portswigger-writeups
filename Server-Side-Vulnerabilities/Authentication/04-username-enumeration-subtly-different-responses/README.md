# PortSwigger Lab: Username Enumeration via Subtly Different Responses

## 📌 Lab Description

This lab is subtly vulnerable to username enumeration and password brute-force attacks.

The objective was to:

1. Enumerate a valid username.
2. Brute-force the user's password.
3. Log in and access the account page.

## 🛠️ Tools Used

- Burp Suite
- Burp Intruder
- Grep - Extract
- Candidate username wordlist
- Candidate password wordlist

## 🔎 Step 1: Username Enumeration

Captured the `POST /login` request and sent it to Burp Intruder.

### Intruder Configuration

- Attack type: `Sniper`
- Payload position: `username`
- Payload type: `Simple list`
- Payload: Candidate usernames

Initially, the responses appeared very similar, with only small differences in response length.

Instead of relying only on the HTTP status code or response length, I used:

`Settings → Grep - Extract`

and extracted:

`Invalid username or password.`

### Finding the Username

The extracted response showed a subtle difference.

Most responses contained:

`Invalid username or password.`

while one response differed slightly in its punctuation/formatting.

This identified the valid username.

## 🔐 Step 2: Password Brute Force

The identified username was kept static and the password parameter was marked as the payload position.

Example:

username=identified-user&password=§invalid-password§

The candidate password list was then loaded into Intruder.

### Result

Most requests returned:

`HTTP 200`

One request returned:

`HTTP 302`

The `302` response indicated a successful authentication attempt.

The corresponding password was identified and used to log in.

## 🎯 Result

Successfully authenticated to the account and accessed the account page.

## 💡 Key Learning

Username enumeration does not always produce obvious differences.

Small differences such as:

- Error-message punctuation
- Response length
- Status codes
- Response timing
- Headers

can reveal whether a username exists.

Using Grep - Extract made the subtle difference much easier to identify.

## 🔗 Reference

https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-subtly-different-responses
