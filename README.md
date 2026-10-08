# Old Sessions - CTF Write-up

## Overview

**Challenge:** Old Sessions
**Platform:** CyLab
**Category:** Web Security
**Operating System:** Windows
**Browser:** Brave
**Tools Used:** None

### Objective

The objective of this challenge was to investigate how user sessions were stored and identify whether the session mechanism could be abused to gain unauthorized access to an administrative account and retrieve the flag.

---

## 1. Initial Reconnaissance

I started by interacting with the web application normally through the browser.

Since the challenge focused on **sessions**, I considered whether the application might be storing session information in a location that could be accessed directly.

One of the things I wanted to investigate was whether:

* There was a dedicated page containing stored sessions.
* Session cookies were being reused between users.
* An administrator's session could potentially be obtained.
* The same session information could be used to authenticate as another user.

No external tools were required for the initial investigation. I used the Brave browser and its built-in developer tools.

---

## 2. Inspecting the Page Source

I inspected the source code of the web page to look for information that was not directly visible in the application's interface.

While reviewing the source, I discovered a hint pointing towards a `/sessions` directory.

The relevant part of the page source indicated that a sessions-related resource existed.

> **Finding:** The application exposed a `/sessions` path that was not directly visible from the normal interface.

### Source page Screenshot
<img width="910" height="226" alt="image" src="https://github.com/user-attachments/assets/62c4a5ee-dc2b-48ad-a59a-8215c3d6ba0e" />

```text
![Page source revealing sessions directory](images/page-source.png)
```

---

## 3. Accessing the Sessions Directory

Based on the information discovered in the source code, I navigated to:

```text
https://<challenge-link>/sessions
```

The `/sessions` page exposed session information associated with users of the application.

The page showed the cookies/session values being used for authentication.

This was significant because session cookies are commonly used by web applications to maintain an authenticated user's state.

### Sessions Page Screenshot
<img width="1039" height="236" alt="image" src="https://github.com/user-attachments/assets/b500b85e-454c-4f35-aae3-21c5986a60b9" />

```text
![Sessions page](images/sessions-page.png)
```

---

## 4. Identifying the Admin Session

While examining the sessions displayed on the page, I identified the session cookie associated with the administrator account.

The application was effectively exposing authentication-related session information that should normally remain protected on the server side.

This meant that obtaining another user's session cookie could potentially allow authentication as that user without knowing their password.

---

## 5. Session Cookie Manipulation

I then examined the cookies stored by the browser for the application.

The existing session cookie was associated with the currently authenticated user.

I replaced the existing cookie value with the session cookie value corresponding to the administrator account that had been exposed on the `/sessions` page.

### Screenshot

<img width="1288" height="284" alt="image" src="https://github.com/user-attachments/assets/6d802e2e-4a48-4874-ae6e-e38969541632" />


```text
![Modified session cookie](images/admin-cookie.png)
```

After changing the cookie, I refreshed the application.

The application recognized the modified session as belonging to the administrator and granted access to the administrator account.

---

## 6. Obtaining the Flag

After successfully authenticating as the administrator through the modified session cookie, I accessed the administrative area of the application.

The flag was available after gaining administrator access.


# Vulnerability Analysis

The primary vulnerability was **session information disclosure combined with improper session management**.

The application exposed stored session information through the `/sessions` endpoint. This allowed an attacker to obtain another user's authentication cookie.

Because the application trusted the session cookie as proof of authentication, replacing the user's cookie with the administrator's session cookie resulted in authentication as the administrator.

### Attack Chain

```text
Page Source
     ↓
Discover /sessions
     ↓
Access Sessions Page
     ↓
Identify Admin Session Cookie
     ↓
Modify Browser Session Cookie
     ↓
Refresh Application
     ↓
Authenticated as Admin
     ↓
Capture Flag
```

---

# Key Security Lessons

### 1. Session identifiers must be protected

Session cookies should be treated as sensitive authentication credentials. Anyone who obtains a valid session identifier may be able to impersonate the associated user.

### 2. Session information should not be publicly exposed

A `/sessions` endpoint should not expose active session identifiers to unauthenticated or unauthorized users.

### 3. Authentication should not rely solely on exposed client-side values

The server must securely manage session state and ensure that session identifiers cannot be obtained or manipulated by unauthorized users.

### 4. Source-code comments and references can reveal hidden functionality

Inspecting the page source was useful during reconnaissance because it revealed the existence of a directory that was not immediately visible through the application's interface.

---

# Methodology Summary

| Stage             | Action                            | Result                           |
| ----------------- | --------------------------------- | -------------------------------- |
| Reconnaissance    | Inspected the web application     | No obvious session functionality |
| Source Analysis   | Inspected page source             | Discovered `/sessions`           |
| Enumeration       | Accessed `/sessions`              | Session information was exposed  |
| Session Analysis  | Identified administrator's cookie | Admin session obtained           |
| Exploitation      | Replaced browser session cookie   | Authenticated as administrator   |
| Post-Exploitation | Accessed admin functionality      | Flag captured                    |

---

# Tools and Environment

### Operating System

* Windows

### Browser

* Brave Browser

### Tools

No external penetration-testing tools were required.

The challenge was completed using:

* Browser
* Browser Developer Tools
* Page source inspection
* Browser cookie/session management

---

# Conclusion

This challenge demonstrated how improper session management can lead to account takeover.

The attack did not require password cracking, exploitation frameworks, or specialized security tools. By performing basic reconnaissance, inspecting the page source, discovering the exposed `/sessions` directory, and identifying the administrator's session cookie, it was possible to impersonate the administrator and obtain the flag.

The key lesson from this challenge is that **session identifiers are authentication credentials and must be protected with the same level of care as passwords**.
