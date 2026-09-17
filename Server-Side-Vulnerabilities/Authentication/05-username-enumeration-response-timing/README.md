# PortSwigger Lab: Username Enumeration via Response Timing

## 📌 Lab Description

This lab is vulnerable to username enumeration through differences in response timing.

The objective was to:

1. Enumerate a valid username.
2. Brute-force the user's password.
3. Access the account page.

## 🛠️ Tools Used

- Burp Suite
- Burp Repeater
- Burp Intruder
- Pitchfork attack
- X-Forwarded-For
- Candidate username wordlist
- Candidate password wordlist

## 🔎 Step 1: Investigating the Login

Captured the `POST /login` request and sent it to Burp Repeater.

Repeated login attempts caused temporary IP-based blocking.

The lab supported the:

`X-Forwarded-For`

header, which could be changed to simulate a different client IP.

## ⚙️ Step 2: Bypassing the IP-Based Protection

A long password was used to amplify the difference in processing time.

The request was sent to Intruder.

### Intruder Configuration

- Attack type: `Pitchfork`
- Payload position 1: `X-Forwarded-For`
- Payload position 2: `username`
- Password: Long static string (~100 characters)

The first payload generated different IP values while the second payload contained the candidate usernames.

## ⏱️ Step 3: Identifying the Username

Most invalid usernames produced relatively short and consistent response times.

During testing, one candidate produced a noticeably longer response.

The candidate was then tested manually in Repeater using fresh `X-Forwarded-For` values.

### Verification

The suspected username repeatedly produced approximately:

`1200–1300 ms`

while an invalid username produced approximately:

`178–234 ms`

This confirmed the username through a repeatable timing difference.

## ⚠️ Troubleshooting

Some early Intruder requests were intercepted by the college/hostel captive portal instead of the PortSwigger lab.

Those responses had characteristics such as:

- Different response length
- Proxy-related headers
- Captive portal HTML

These results were discarded.

Later, a response-length anomaly was also observed. It was investigated separately instead of automatically treating it as proof of a valid username.

The primary signal remained the repeatable response-time difference.

## 🔐 Step 4: Password Brute Force

Created another Intruder attack using:

- Pitchfork
- `X-Forwarded-For` payload
- Identified username
- Candidate password list

The successful password produced:

`HTTP 302`

The password corresponding to that request was identified.

## 🎯 Result

Successfully logged in to the user's account and accessed the account page.

## 💡 Key Learning

Username enumeration can be performed without obvious response differences.

When authentication processing differs internally, response timing can leak whether a username exists.

Important factors when analyzing timing attacks:

- Establish a baseline.
- Repeat suspicious requests.
- Use fresh IP/header values when rate limiting is present.
- Ignore responses that originate from intermediary infrastructure such as captive portals.

## 🔗 Reference

https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing
