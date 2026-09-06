# Lab: Password Reset Broken Logic

## Lab Description

This lab demonstrates a password reset vulnerability caused by broken authentication logic.

The application's password reset functionality uses a temporary password reset token together with a user-controlled username parameter. However, the application fails to properly associate the reset token with the account it was originally issued for.

By modifying the username parameter while using a valid reset token issued for another account, an attacker can reset the victim's password and subsequently authenticate as the victim.

---

## Objective

Reset Carlos's password by exploiting the vulnerable password reset functionality, authenticate using the new password, and access Carlos's "My account" page.

---

## Vulnerability Type

- Password Reset Broken Logic
- Improper Token-to-Account Binding
- Authentication Bypass

---

## Vulnerable Feature

Password reset functionality.

---

## Entry Point

Password reset functionality.

Relevant endpoint:

    POST /forgot-password

The password reset process uses the following parameters:

    temp-forgot-password-token
    username
    new-password-1
    new-password-2

---

## Source

The attacker-controlled input originates from the password reset request:

    temp-forgot-password-token
    username
    new-password-1
    new-password-2

The reset token is supplied through the password reset link, while the username is submitted as a separate parameter in the password reset form.

The application processes these values during the password reset operation.

---

## Sink

The password modification operation performed by the server after processing the reset request.

The vulnerable logic allows the user-controlled `username` parameter to determine which account's password is changed without properly verifying that the reset token belongs to that account.

---

## Root Cause

The root cause is improper binding between the password reset token and the target user account.

A secure password reset mechanism should associate the generated reset token with the specific account for which the reset was requested.

For example:

    Reset Token
         ↓
    Associated Account
         ↓
       Wiener

When the password is changed, the server should determine the target account from the valid reset token rather than trusting a user-controlled username parameter.

In the vulnerable application, the reset token and username are processed independently.

The application verifies that the reset token is valid but does not properly verify that the token belongs to the username supplied in the request.

This allows a valid reset token issued for Wiener to be combined with:

    username=carlos

resulting in Carlos's password being changed.

---

## Data Flow

    Request Password Reset
            ↓
       username=wiener
            ↓
    Password Reset Token Generated
            ↓
    Reset Link Sent to Wiener
            ↓
    Password Reset Form
            ↓
    Valid Reset Token
            +
    User-Controlled Username
            ↓
    Password Reset Processing
            ↓
    Username Determines Target Account
            ↓
    Carlos's Password Changed
            ↓
    Login as Carlos
            ↓
       /my-account

---

## Exploitation Process

1. Opened the password reset functionality.

2. Submitted the username:

       username=wiener

3. The application generated a temporary password reset token and sent a password reset email to Wiener.

4. Opened the Academy Email Client and retrieved the password reset link.

5. The reset link contained the following parameter:

       temp-forgot-password-token=REDACTED

6. Followed the reset link and inspected the password reset request using Burp Suite.

7. The password reset request contained:

       temp-forgot-password-token=REDACTED
       username=wiener
       new-password-1=TEST_PASSWORD
       new-password-2=TEST_PASSWORD

8. Sent the password reset request to Burp Repeater for testing.

9. Changed only the username parameter:

       username=wiener

    to:

       username=carlos

10. Kept the original valid reset token unchanged.

11. Sent the modified request.

12. The server responded with:

       HTTP/2 302 Found

    and redirected to:

       Location: /

13. This indicated that the password reset request had been accepted and processed.

14. Attempted to authenticate using:

       username=carlos
       password=TEST_PASSWORD

15. Authentication was successful.

16. Accessed:

       /my-account

17. Successfully accessed Carlos's account page and completed the lab.

---

## Payload

The original password reset request contained:

    temp-forgot-password-token=VALID-WIENER-TOKEN
    username=wiener
    new-password-1=TEST_PASSWORD
    new-password-2=TEST_PASSWORD

The vulnerable request was modified to:

    temp-forgot-password-token=VALID-WIENER-TOKEN
    username=carlos
    new-password-1=TEST_PASSWORD
    new-password-2=TEST_PASSWORD

The reset token remained unchanged while only the username parameter was modified.

---

## Why the Attack Works

The password reset token is intended to authorize a password reset for a specific account.

The expected flow is:

    Password Reset Request
            ↓
       Token Generated
            ↓
    Token Associated with Wiener
            ↓
       Reset Request
            ↓
    Token Identifies Wiener
            ↓
    Wiener's Password Changed

However, the vulnerable application effectively processes the reset request as:

    Valid Reset Token
            +
    username=carlos
            ↓
    Password Reset Processing
            ↓
    Carlos's Password Changed

The application verifies that the reset token is valid but fails to verify that the token was originally issued for Carlos.

This creates a broken relationship between:

    Reset Token ←→ User Account

Because the username is controlled by the client, it can be changed to another valid username.

As a result, an attacker who possesses a valid password reset token for their own account can use it to reset another user's password.

---

## Result

Successfully exploited the password reset functionality to change Carlos's password using a reset token originally issued for Wiener.

The modified password was then used to authenticate as Carlos.

Access to:

    /my-account

was successfully obtained, completing the lab.

---

## Key Learning

This lab demonstrates the importance of correctly binding password reset tokens to the accounts for which they were generated.

Important takeaways:

- Password reset tokens must be securely associated with a specific user account.
- A valid reset token should not be usable to reset another user's password.
- User-controlled parameters should not determine the identity of the account being reset.
- The server should derive the target account from the reset token itself.
- Password reset functionality must validate both token authenticity and token ownership.
- HTTP requests should be inspected to understand how authentication state and reset tokens are handled.
- Burp Suite Repeater can be used to safely test whether authentication parameters are properly associated.
- A successful password reset for another account can lead directly to account takeover.

---

## Skills Practiced

- Authentication Testing
- Password Reset Testing
- Broken Authentication Logic Analysis
- Token Analysis
- Token-to-Account Binding Analysis
- Account Takeover Testing
- HTTP Request Analysis
- Burp Suite
- Burp Repeater
- Parameter Manipulation
- Web Application Security Testing
