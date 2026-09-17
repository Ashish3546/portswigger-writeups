# PortSwigger Lab: Broken Brute-Force Protection, IP Block

## 📌 Lab Description

This lab contains a logic flaw in its password brute-force protection.

The objective was to:

1. Brute-force Carlos's password.
2. Bypass the account/IP protection mechanism.
3. Log in as Carlos.
4. Access the account page.

## 🛠️ Tools Used

- Burp Suite
- Burp Intruder
- Pitchfork attack
- Candidate password wordlist

## 🔎 Step 1: Understanding the Protection

The application temporarily blocked login attempts after three incorrect attempts.

However, testing revealed that successfully logging into my own account reset the failed-login counter.

Known valid credentials:

`wiener:peter`

Target account:

`carlos`

## 💡 Identifying the Logic Flaw

The protection could be bypassed by alternating successful and failed authentication attempts.

The attack sequence was:

wiener:peter  
carlos:password1  
wiener:peter  
carlos:password2  
wiener:peter  
carlos:password3  
...

The successful login to `wiener` resets the failed-attempt counter before another Carlos password is tested.

## ⚙️ Step 2: Intruder Configuration

Used:

`Pitchfork`

Two payload positions were configured:

### Payload 1 — Username

Alternating values:

wiener  
carlos  
wiener  
carlos  
wiener  
carlos  
...

### Payload 2 — Password

The known password `peter` was placed before every candidate password:

peter  
candidate1  
peter  
candidate2  
peter  
candidate3  
...

This ensured that every Carlos password attempt was preceded by a successful login to the Wiener account.

## 🔎 Step 3: Analyzing the Results

Normal failed Carlos login attempts returned:

`HTTP 200`

The known successful Wiener login returned:

`HTTP 302`

The important result was a `302` response associated with the username:

`carlos`

The corresponding password was:

`master`

The response redirected to:

`/my-account?id=carlos`

which confirmed successful authentication as Carlos.

## 🎯 Result

Successfully logged in as Carlos and accessed the account page.

## 💡 Key Learning

Brute-force protection can fail because of flawed state management.

A protection mechanism may appear effective while still being bypassable if:

- Failed attempts are tracked incorrectly.
- Successful authentication resets the wrong counter.
- Counters are associated with sessions/IPs rather than accounts.
- Multiple authentication states interact unexpectedly.

The key vulnerability here was not the password itself, but the logic used to track failed attempts.

## 🔗 Reference

https://portswigger.net/web-security/authentication/password-based/lab-broken-bruteforce-protection-ip-block
