# Authentication Bypass via Information Disclosure

**Platform:** PortSwigger Web Security Academy  
**Category:** Information Disclosure  
**Difficulty:** Apprentice  
**Status:** Solved ✅

## Lab Description

This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.

The objective is to obtain the header name, use it to bypass the lab's authentication, access the administration interface, and delete the user `carlos`.

## Lab Credentials

- **Username:** `wiener`
- **Password:** `peter`

## Vulnerability

**Authentication Bypass via Information Disclosure**

The application relies on a custom HTTP header to determine the client's IP address. The header can be discovered through the HTTP `TRACE` method.

Once the header name was identified, Burp Suite Match and Replace was used to add the header with the value `127.0.0.1`, causing the application to treat the request as originating from the local machine.

## Exploitation

### Step 1 — Access the Administration Interface

After logging in as `wiener`, I attempted to access the administration interface at `/admin`.

The application denied access because the request was not considered to originate from the local IP address.

### Step 2 — Capture the Request

The `/admin` request was intercepted using Burp Suite and sent to Repeater for further testing.

The initial request was:

`GET /admin`

### Step 3 — Change the Request Method to TRACE

The request method was changed from `GET` to `TRACE`:

`TRACE /admin`

After sending the request, the response disclosed a custom HTTP header:

`X-Custom-IP-Authorization`

This revealed the header used by the front-end to identify the client's IP address.

### Step 4 — Configure Burp Match and Replace

Burp Suite's **Proxy → Match and replace** feature was configured to automatically add the discovered header to requests.

The rule was configured as:

- **Type:** Request header
- **Match:** Blank
- **Replace:** `X-Custom-IP-Authorization: 127.0.0.1`

Leaving the Match field blank causes Burp Suite to add the specified header to the request.

### Step 5 — Access the Administration Panel

After enabling the Match and Replace rule, I accessed `/admin` again.

Burp automatically added:

`X-Custom-IP-Authorization: 127.0.0.1`

The application therefore treated the request as originating from the local IP address and granted access to the administration interface.

### Step 6 — Delete the User

The administration panel displayed the available users, including `carlos`.

I selected the delete option for `carlos`.

The lab was then marked as **Solved**.

## Attack Flow

`GET /admin`

↓

Access denied

↓

`TRACE /admin`

↓

Discover `X-Custom-IP-Authorization`

↓

Configure Burp Match and Replace

↓

Add `X-Custom-IP-Authorization: 127.0.0.1`

↓

Access `/admin`

↓

Delete `carlos`

↓

**Lab Solved ✅**

## Why It Worked

The application trusted the `X-Custom-IP-Authorization` header when determining whether a request originated from the local machine.

The `TRACE` request disclosed the existence of this header, allowing the authentication mechanism to be understood.

By supplying the header with the value `127.0.0.1`, the request was treated as a local request and the administration interface became accessible.

## Impact

An attacker who discovers and manipulates the trusted header may be able to bypass the intended authentication or access-control restriction.

This could provide unauthorized access to administrative functionality and allow privileged actions to be performed.

## Mitigation

- Do not rely on client-controlled HTTP headers for authentication or authorization decisions.
- Implement proper server-side authentication and authorization.
- Treat IP-related headers as untrusted unless they are securely supplied by a trusted intermediary.
- Restrict administrative functionality using robust access-control mechanisms.
- Disable unnecessary HTTP methods such as `TRACE`.
- Avoid exposing internal security mechanisms through application responses.

## Tools Used

- Burp Suite
- Burp Repeater
- Burp Proxy Match and Replace
- Web Browser
- PortSwigger Web Security Academy

## Evidence

- `PS-07-01_InfoDisclosure_AuthBypass_LabPage.png`
- `PS-07-02_InfoDisclosure_TRACE_Response.png`
- `PS-07-03_InfoDisclosure_MatchReplace.png`
- `PS-07-04_InfoDisclosure_AdminPanel.png`
- `PS-07-05_InfoDisclosure_AuthBypass_Solved.png`

## Key Takeaways

- Information disclosure can reveal internal application security mechanisms.
- HTTP `TRACE` can expose information added to requests by intermediary components.
- Client-controlled headers should not be trusted for authentication or authorization.
- Burp Suite Match and Replace can be useful for testing header-based access controls.
- Administrative access should be protected using proper authentication and authorization mechanisms.

## Reference

PortSwigger Web Security Academy — Authentication bypass via information disclosure
