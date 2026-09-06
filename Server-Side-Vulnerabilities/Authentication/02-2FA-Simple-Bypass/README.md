from pathlib import Path

readme = r"""# Lab: 2FA Simple Bypass

## Lab Description

This lab demonstrates a two-factor authentication bypass vulnerability caused by improper enforcement of the 2FA verification step.

The application allows a user to authenticate with a valid username and password, but fails to properly enforce the 2FA verification before granting access to authenticated functionality.

By directly navigating to the authenticated account page without completing the 2FA step, an attacker can bypass the second authentication factor.

---

## Objective

Bypass the two-factor authentication mechanism and access Carlos's account page without knowing his 2FA verification code.

---

## Vulnerability Type

- Two-Factor Authentication Bypass
- Improper Authentication State Enforcement

---

## Vulnerable Feature

Login and 2FA verification functionality.

---

## Entry Point

Login and 2FA verification functionality.

Relevant endpoints:

    POST /login
    GET /login2
    GET /my-account

---

## Source

The attacker-controlled input originates from the authentication flow:

    username
    password
    2FA verification code

The application establishes an authenticated session after the username and password are validated.

The 2FA verification step is then expected to act as an additional authentication barrier before access to authenticated functionality.

---

## Sink

The server-side authorization logic protecting the authenticated account page:

    GET /my-account

The application fails to properly verify that the 2FA step has been completed before allowing access to the account page.

---

## Root Cause

The root cause is improper enforcement of the authentication state.

After successfully submitting valid username and password credentials, the application allows the user to reach the 2FA verification stage.

However, the application does not properly verify that the 2FA challenge has been completed before serving the authenticated account page.

This allows an attacker to bypass the 2FA verification page by directly requesting:

    /my-account

The application therefore treats the partially authenticated session as sufficiently authenticated to access protected functionality.

---

## Data Flow

    Username / Password
            ↓
       Login Form
            ↓
       POST /login
            ↓
    Credential Validation
            ↓
    Authenticated Session
            ↓
    2FA Verification Page
            ↓
        GET /login2
            ↓
    2FA Verification Not Properly Enforced
            ↓
    Direct Request to /my-account
            ↓
       Carlos's Account

---

## Exploitation Process

1. Logged into the application using the provided credentials:

       username=wiener
       password=peter

2. The application sent a 2FA verification code to the provided email client.

3. Retrieved the 2FA verification code from the Academy email client.

4. Completed the 2FA verification process for the Wiener account.

5. Accessed the account page and observed that the authenticated account page was available at:

       /my-account

6. Logged out of the application.

7. Logged in using the victim's credentials:

       username=carlos
       password=montoya

8. The application requested Carlos's 2FA verification code.

9. Since Carlos's 2FA code was not available, the verification step was not completed.

10. Instead of attempting to obtain the 2FA code, manually changed the URL from the 2FA verification page:

       /login2

    to:

       /my-account

11. The application returned Carlos's account page despite the 2FA verification step not being completed.

12. The lab was successfully solved.

---

## Payload

No parameter manipulation or brute-force payload was required.

The bypass consisted of directly requesting the authenticated account endpoint:

    GET /my-account

instead of completing the expected 2FA verification flow.

---

## Why the Attack Works

The application correctly verifies the username and password before presenting the 2FA challenge.

However, the application fails to enforce the 2FA requirement when the user directly requests the account page.

The expected authentication flow is:

    Username + Password
            ↓
      2FA Verification
            ↓
    Authenticated Account

However, the vulnerable application allows:

    Username + Password
            ↓
    2FA Verification Page
            ↓
    Direct Request to /my-account
            ↓
    Authenticated Account

The server therefore fails to distinguish between:

    2FA completed

and:

    2FA pending

when processing access to the account page.

This allows the attacker to bypass the second authentication factor without knowing the victim's 2FA code.

---

## Result

Successfully bypassed Carlos's two-factor authentication mechanism without obtaining his 2FA verification code.

The attacker was able to access:

    /my-account

as Carlos, completing the lab.

---

## Key Learning

This lab demonstrates that implementing a 2FA verification page is not sufficient to provide secure multi-factor authentication.

Important takeaways:

- Every protected endpoint must enforce the required authentication state.
- A valid username and password should not automatically grant access to functionality protected by 2FA.
- Applications must distinguish between partially authenticated and fully authenticated sessions.
- Authentication checks should be performed server-side rather than relying on the user following the intended navigation flow.
- Direct access to protected endpoints should not allow users to skip authentication steps.
- Multi-factor authentication must be enforced consistently across all authenticated functionality.
- URL manipulation can reveal broken authentication state management.
- Burp Suite can be used to inspect and understand authentication flows.

---

## Skills Practiced

- Authentication Testing
- Two-Factor Authentication Testing
- Authentication Bypass
- Authentication State Analysis
- HTTP Request Analysis
- Burp Suite
- URL Manipulation
- Web Application Authentication Testing
"""

path = Path("/mnt/data/Lab_2_2FA_Simple_Bypass_README.md")
path.write_text(readme, encoding="utf-8")

print(path)
