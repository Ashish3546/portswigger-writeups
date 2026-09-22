# 05 - User ID Controlled by Request Parameter

**Platform:** PortSwigger Web Security Academy  
**Category:** Access Control  
**Vulnerability:** Horizontal Privilege Escalation / IDOR  
**Difficulty:** Apprentice  
**Status:** Solved ✅

---

## 📌 Lab Description

This lab contains a horizontal privilege escalation vulnerability on the user account page.

The application uses a user-controlled `id` parameter in the URL to determine which user's account information should be displayed.

The objective is to obtain the API key belonging to the user `carlos` while authenticated as the user `wiener`.

---

## 🎯 Objective

Obtain the API key for the user `carlos` and submit it as the solution.

---

## 🧠 Vulnerability Concept

### Horizontal Privilege Escalation

Horizontal privilege escalation occurs when a user is able to access resources belonging to another user with the same level of privileges.

For example:

`Wiener → Own Account`

`Carlos → Own Account`

Wiener should not be able to access Carlos's account.

However, if the application uses a user-controlled parameter such as:

`/my-account?id=wiener`

and does not properly verify that the requested account belongs to the authenticated user, the parameter may be manipulated to reference another user's account.

### IDOR

This vulnerability is also an example of an **Insecure Direct Object Reference (IDOR)**.

An IDOR occurs when an application uses user-supplied input to directly access an object or resource without properly verifying whether the current user is authorized to access it.

---

## 🔍 Testing

### 1. Log in as Wiener

The supplied credentials were used:

- **Username:** `wiener`
- **Password:** `peter`

After authentication, the **My account** page was accessed.

The URL contained a user-controlled `id` parameter:

`/my-account?id=wiener`

This indicated that the application was using the value of the `id` parameter to determine which user's account information to display.

---

### 2. Test the User ID Parameter

The `id` parameter was modified from:

`/my-account?id=wiener`

to:

`/my-account?id=carlos`

The request was made while still authenticated as **Wiener**.

The application returned Carlos's account page instead of restricting access to Wiener's account.

---

## 💥 Exploitation

The attack can be represented as:

`Authenticated User: Wiener`

`Original Request: GET /my-account?id=wiener`

`Modified Request: GET /my-account?id=carlos`

Even though the authenticated session still belonged to Wiener, the application accepted `carlos` as the requested user ID and returned Carlos's account information.

The API key displayed on Carlos's account page was then submitted as the lab solution.

---

## 🧪 Why It Worked

The application trusted the user-controlled `id` parameter without sufficiently verifying that the authenticated user was authorized to access the requested account.

Conceptually, the vulnerable behavior can be represented as:

`requested_user = request.id`

`display_account(requested_user)`

Instead, the application should verify that the requested account belongs to, or is accessible by, the authenticated user before returning the data.

Because the necessary authorization check was missing or insufficient, changing the parameter allowed access to another user's account.

---

## 📊 Attack Flow

`Wiener logs in`

↓

`GET /my-account?id=wiener`

↓

`Wiener's account is displayed`

↓

`Change id=wiener to id=carlos`

↓

`GET /my-account?id=carlos`

↓

`Carlos's account is displayed`

↓

`Carlos's API key is obtained`

↓

`Lab solved ✅`

---

## 🔐 Impact

An attacker could potentially access sensitive information belonging to other users without knowing their credentials.

Depending on the affected functionality, this type of vulnerability could result in:

- Unauthorized access to personal information
- Exposure of API keys or other sensitive data
- Access to private account information
- Unauthorized actions on another user's behalf

---

## 🛡️ Mitigation

The application should perform authorization checks on the server side for every request involving user-specific resources.

Recommended measures include:

- Enforce authorization checks server-side.
- Do not rely on user-controlled parameters for authorization decisions.
- Verify that the authenticated user is authorized to access the requested resource.
- Deny access when authorization cannot be established.
- Apply access-control checks consistently across all endpoints.
- Test endpoints using identifiers belonging to other users.

---

## 🛠️ Tools Used

- PortSwigger Web Security Academy
- Web Browser
- Burp Suite

---

## 📸 Evidence

The following screenshots document the exploitation process:

- `PS-03-01_AccessControl_UserID_RequestParameter_LabPage.png`
- `PS-03-02_AccessControl_UserID_RequestParameter_Wiener.png`
- `PS-03-03_AccessControl_UserID_RequestParameter_Carlos.png`
- `PS-03-04_AccessControl_UserID_RequestParameter_Solved.png`

---

## 📚 Key Takeaways

- Access control determines what an authenticated user is authorized to access.
- Horizontal privilege escalation occurs when a user accesses another user's resources.
- User-controlled identifiers should not be trusted for authorization decisions.
- IDOR vulnerabilities can occur when object references are directly exposed through request parameters.
- Authentication and authorization are different concepts.
- An authenticated user should only be able to access resources they are authorized to access.
- Authorization checks must be performed server-side.

---

## 🔗 Reference

[PortSwigger - Testing for IDORs](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/access-controls/testing-for-idors)
