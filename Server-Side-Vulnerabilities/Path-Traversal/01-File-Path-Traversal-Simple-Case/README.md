# Lab: File Path Traversal, Simple Case

---

## Lab Description

This lab contains a path traversal vulnerability in the display of product images.

The application uses a user-controlled `filename` parameter to retrieve product images. By manipulating this parameter with path traversal sequences, it is possible to access files outside the intended directory.

PortSwigger classifies this as an **Apprentice-level** path traversal lab. :contentReference[oaicite:0]{index=0}

---

## Objective

Retrieve the contents of the `/etc/passwd` file from the server.

---

## Vulnerability Type

**File Path Traversal / Directory Traversal**

**CWE:** CWE-22 — Improper Limitation of a Pathname to a Restricted Directory

---

## Vulnerable Feature

The vulnerable functionality is the product image loading endpoint:

`/image`

The endpoint accepts a user-controlled parameter:

`filename`

A normal request looks like:

`GET /image?filename=21.jpg HTTP/2`

The application is expected to restrict this parameter to files within the intended image directory.

However, the parameter can be manipulated using path traversal sequences.

---

## Entry Point

The vulnerable endpoint is:

`/image`

The vulnerable parameter is:

`filename`

Example:

`GET /image?filename=21.jpg HTTP/2`

---

## Source

The attacker-controlled input originates from the `filename` parameter.

Example:

`filename=21.jpg`

This value can be modified using Burp Suite.

---

## Sink

The `filename` parameter is used by the application when retrieving a file from the server's filesystem.

Because the supplied path is not sufficiently restricted, traversal sequences can be used to access files outside the intended directory.

---

## Root Cause

The root cause is insufficient validation and restriction of user-controlled file paths.

The application accepts path traversal sequences such as `../` instead of ensuring that the requested file remains inside the intended image directory.

---

## Data Flow

`User Input → filename parameter → /image endpoint → Server-side file retrieval → Requested file`

Normal request:

`filename=21.jpg → Image directory → 21.jpg`

Malicious request:

`filename=../../../etc/passwd → Path traversal → Escape intended directory → /etc/passwd`

---

## Exploitation Process

### Step 1 — Access the Lab

The lab was opened through the PortSwigger Web Security Academy.

The official lab is titled:

**File path traversal, simple case**

The lab contains a path traversal vulnerability in the display of product images. :contentReference[oaicite:1]{index=1}

---

### Step 2 — Identify the Image Request

A product image request was identified and intercepted using **Burp Suite Community Edition**.

The original request was:

`GET /image?filename=21.jpg HTTP/2`

The important parameter was:

`filename=21.jpg`

This indicated that the client controlled the filename being requested.

---

### Step 3 — Test for Path Traversal

The `filename` parameter was modified to include directory traversal sequences.

The following payload was used:

`../../../etc/passwd`

The resulting request was:

`GET /image?filename=../../../etc/passwd HTTP/2`

The `../` sequences attempt to move upward through the filesystem hierarchy.

---

### Step 4 — Trigger the Vulnerability

The modified request was sent to the server.

The lab was successfully solved using the following value for the `filename` parameter:

`../../../etc/passwd`

The PortSwigger learning path identifies this as the **File path traversal, simple case** Apprentice lab. :contentReference[oaicite:2]{index=2}

---

## Payload

`../../../etc/passwd`

Full parameter:

`filename=../../../etc/passwd`

---

## Why the Attack Works

The application expects the `filename` parameter to reference an image inside its intended directory.

For example:

`21.jpg`

However, the application does not properly restrict the supplied path.

The sequence:

`../`

means to move one directory level upward.

Therefore:

`../../../etc/passwd`

moves upward through multiple directories before attempting to access:

`/etc/passwd`

This allows the attacker to escape the intended image directory and access a file elsewhere on the server.

PortSwigger describes path traversal as a vulnerability that can allow attackers to interact with arbitrary files on the server. :contentReference[oaicite:3]{index=3}

---

## Result

The path traversal payload successfully solved the lab:

`../../../etc/passwd`

The lab objective was to retrieve the contents of `/etc/passwd`, demonstrating that the vulnerable image retrieval functionality could be manipulated to access a file outside the intended directory.

---

## Impact

A successful path traversal vulnerability can allow an attacker to access files outside the application's intended directory.

Depending on the server configuration and file permissions, this may expose:

- Operating system files
- Application source code
- Configuration files
- Credentials
- API keys
- Database configuration
- Other sensitive information

PortSwigger notes that path traversal can provide access to sensitive server-side data and, in some circumstances, potentially allow modification of application data or behavior. :contentReference[oaicite:4]{index=4}

---

## Mitigation

Recommended defenses include:

1. Avoid using user-controlled input directly in filesystem operations whenever possible.

2. Use a whitelist of permitted files instead of accepting arbitrary filenames.

3. Validate and sanitize user-controlled file paths.

4. Canonicalize the resulting filesystem path before accessing it.

5. Verify that the canonicalized path remains inside the intended base directory.

6. Prevent traversal sequences such as `../` from escaping the intended directory.

7. Ensure that sensitive operating system and application files cannot be accessed through user-controlled file retrieval functionality.

---

## Tools Used

- Burp Suite Community Edition
- PortSwigger Web Security Academy
- Web Browser

---

## Key Learning

This lab demonstrated how a file retrieval feature can become vulnerable when user-controlled input is used to construct filesystem paths without proper validation.

The main technique learned was identifying parameters that reference files and testing whether path traversal sequences such as `../` can be used to escape the intended directory.

The lab also demonstrated how Burp Suite can be used to intercept and modify HTTP requests during web application security testing.

---

## Skills Practiced

- Burp Suite Proxy
- HTTP request interception
- HTTP parameter manipulation
- File path traversal testing
- Directory traversal analysis
- Server-side vulnerability identification
- Filesystem path analysis
- Vulnerability validation

---

## References

- [PortSwigger Web Security Academy — Path Traversal](https://portswigger.net/web-security/file-path-traversal)
- [PortSwigger Web Security Academy — Path Traversal Learning Path](https://portswigger.net/web-security/learning-paths/path-traversal)
