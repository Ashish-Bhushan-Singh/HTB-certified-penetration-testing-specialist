# Section 08: Web Enumeration

**Module:** 02 — Getting Started  
**Topics:** Web Content Discovery, FFUF, Robots.txt, HTML Inspection, Credential Discovery  
**Status:** Answer documented

---

## Questions & Answers

### Question 1: Discover the Hidden Web Login and Retrieve the Flag

**Question**

Enumerate the web application, discover the administrative login page, identify the credentials, and retrieve the flag.

### Step 1: Discover Web Content Using FFUF

**Command Used**

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
-u http://154.57.164.83:30616/FUZZ -ac
```

**Command Explanation**

| Option | Description |
|---|---|
| `-w` | Specifies the wordlist used for content discovery. |
| `-u` | Specifies the target URL. |
| `FUZZ` | Placeholder replaced by each wordlist entry. |
| `-ac` | Enables automatic calibration to help filter responses that match the server's baseline behavior. |

The scan used the SecLists `common.txt` wordlist to discover accessible web paths.

**Observed Results**

```text
index.php       [Status: 200, Size: 990, Words: 424, Lines: 55]
robots.txt      [Status: 200, Size: 45, Words: 3, Lines: 2]
wordpress       [Status: 301, Size: 327, Words: 20, Lines: 10]
```

**Detailed Explanation**

The scan identified three interesting paths:

- `index.php` returned HTTP 200, indicating a successful response.
- `robots.txt` returned HTTP 200 and could contain information about paths the site owner does not want search engines to crawl.
- `wordpress` returned HTTP 301, indicating a redirect.

The discovery of `robots.txt` provided another useful enumeration opportunity.

### Step 2: Inspect robots.txt

**Command Used**

```bash
curl http://154.57.164.83:30616/robots.txt
```

**Observed Output**

```text
User-agent: *
Disallow: /admin-login-page.php
```

**Detailed Explanation**

The `robots.txt` file provides instructions to web crawlers.

- `User-agent: *` applies the following rule to all crawlers.
- `Disallow: /admin-login-page.php` tells compliant crawlers not to crawl that path.

This entry reveals the location of an administrative login page.

**Important Security Note**

The `Disallow` directive is not an access-control mechanism. It does not prevent users from requesting the path directly. Sensitive pages should be protected through proper authentication and authorization.

### Step 3: Inspect the Administrative Login Page

**Command Used**

```bash
curl http://154.57.164.83:30616/admin-login-page.php
```

The command retrieved the page's HTML source.

**Relevant HTML**

```html
<form name='login' autocomplete='off' class='form'
      action='' method='post'>

    <h1>Admin Panel</h1>

    <label for="username"><b>Username</b></label>
    <input name='username' placeholder='Username' type='text'>

    <label for="password"><b>Password</b></label>
    <input name='password' placeholder='Password' type='password'>

    <!-- TODO: remove test credentials admin:password123 -->

    <button type="submit" formmethod='post'>Login</button>
</form>
```

**Detailed Explanation**

Inspecting the HTML source revealed a developer comment containing test credentials:

```text
admin:password123
```

This is an example of sensitive information accidentally exposed in client-accessible source code.

HTML comments are not displayed as normal page content, but they remain visible to anyone who retrieves the source.

Exposing credentials in application source code is a security weakness. Test credentials should be removed before deployment, and secrets should never be embedded in publicly accessible client-side content.

### Step 4: Log In and Retrieve the Flag

Using the discovered credentials, the administrative login page was accessed:

| Field | Value |
|---|---|
| Username | `admin` |
| Password | `password123` |
| Page | `/admin-login-page.php` |

After logging in, the flag was obtained.

**Observed Flag**

```text
HTB{w3b_3num3r4710n_r3v34l5_53cr375}
```

**Short Answer**

`HTB{w3b_3num3r4710n_r3v34l5_53cr375}`

---

## Key Takeaways

1. **Content discovery:** FFUF can identify hidden files and directories using wordlists.
2. **HTTP status codes:** Status codes such as 200 and 301 help interpret discovered resources.
3. **robots.txt:** This file can reveal paths that are not intended for crawler indexing.
4. **Source inspection:** HTML comments can accidentally expose sensitive information.
5. **Credential security:** Test credentials must not be exposed in production application source code.
6. **Evidence-based enumeration:** Each discovery should be verified by inspecting the actual response.

---

[Back to Module Index](./README.md)
