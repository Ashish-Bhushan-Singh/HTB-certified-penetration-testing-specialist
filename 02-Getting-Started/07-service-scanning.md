# Section 07: Service Scanning

**Module:** 02 — Getting Started  
**Topics:** Nmap Service Detection, Non-Default Ports, SMB Enumeration  
**Status:** Answers documented

---

## Questions & Answers

### Question 1: Identify the Service on Port 8080

**Question**

Perform an Nmap scan of the target. What does Nmap display as the version of the service running on port `8080`?

**Command Used**

```bash
nmap -sC -sV 10.129.132.119
```

**Relevant Scan Output**

```text
8080/tcp open  http  Apache Tomcat
|_http-title: Apache Tomcat
```

**Detailed Explanation**

The Nmap command uses two options:

- `-sC` runs Nmap's default set of NSE scripts.
- `-sV` enables service and version detection to help identify the software running on open ports.

The scan reports port `8080` as open and identifies the service as `http`, with the software name `Apache Tomcat`.

Apache Tomcat is a web server and servlet container commonly used to run Java-based web applications. Port `8080` is often used for HTTP services as an alternative to the default HTTP port, `80`.

The supplied output identifies the software as **Apache Tomcat**, but it does not display a specific Tomcat version number. Therefore, the answer should match the service identification shown by Nmap.

**Short Answer**

`Apache Tomcat`

---

### Question 2: Identify the Non-Default Telnet Port

**Question**

Perform an Nmap scan of the target and identify the non-default port that the Telnet service is running on.

**Relevant Scan Output**

```text
2323/tcp open  telnet  Linux telnetd
```

**Detailed Explanation**

The scan identifies the Telnet service as running on TCP port `2323`.

Telnet conventionally uses TCP port `23`. In this scan, it is available on port `2323`, which is a non-default port for Telnet.

Identifying the port associated with a service helps a penetration tester understand the target's exposed network services and determine which services may require further investigation.

**Short Answer**

`2323`

---

### Question 3: Retrieve the Flag from the SMB Share

**Question**

List the SMB shares available on the target host. Connect to the available share as the `bob` user. Once connected, access the folder called `flag` and submit the contents of the `flag.txt` file.

**Target**

```text
10.129.132.119
```

#### Step 1: Enumerate Available SMB Shares

**Command Used**

```bash
smbclient -N -L //10.129.132.119
```

**Observed Output**

```text
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
users           Disk
IPC$            IPC       IPC Service (gs-svcscan server (Samba, Ubuntu))
```

**Explanation**

The `smbclient` utility allows users to interact with SMB shares.

- `-L` lists the shares exposed by the target.
- `-N` attempts the connection without prompting for a password.

The scan revealed three shares: `print$`, `users`, and `IPC$`. Listing the shares does not necessarily mean that their contents are accessible without authentication.

#### Step 2: Connect to the `users` Share as `bob`

**Command Used**

```bash
smbclient -U bob //10.129.132.119/users
```

The command prompts for the password associated with the `bob` account. After connecting, the following command lists the share's contents:

```text
smb: \> ls
```

**Relevant Output**

```text
.       D
..      D
flag    D
bob     D
```

**Explanation**

The directory listing shows a folder named `flag` within the `users` share.

The earlier unauthenticated attempt did not successfully list the share contents. The supplied evidence shows that the authenticated session as `bob` could list the directory.

#### Step 3: Open the `flag` Directory

Attempting to retrieve `flag` directly produced this message:

```text
NT_STATUS_FILE_IS_A_DIRECTORY opening remote file \flag
```

This indicates that `flag` is a directory rather than a regular file.

The correct approach is to change into that directory and list its contents:

```text
smb: \> cd flag
smb: \flag\> ls
```

**Relevant Output**

```text
.          D
..         D
flag.txt   N   33
```

The listing confirms that `flag.txt` is present and has a reported size of 33 bytes.

#### Step 4: Download and Read `flag.txt`

From the SMB session, the file was downloaded using:

```text
smb: \flag\> get flag.txt
```

After exiting the SMB session, the local file was read with:

```bash
cat flag.txt
```

**Observed Flag**

```text
dceece590f3284c3866305eb2473d099
```

**Detailed Explanation**

This exercise demonstrates a basic SMB enumeration and file retrieval workflow:

1. Enumerate the target's available SMB shares.
2. Connect to the relevant share using the specified account.
3. List the share's contents to identify directories.
4. Navigate into the target directory.
5. Download the required file.
6. Read the downloaded file and submit its contents.

**Short Answer**

`dceece590f3284c3866305eb2473d099`

---

## Key Takeaways

- **Nmap service detection:** The `-sV` option helps identify the software associated with open ports.
- **Non-default ports:** Services can operate on ports other than their conventional defaults.
- **SMB enumeration:** Share discovery and access to share contents are separate steps.
- **Authentication:** A share may be discoverable but still require credentials to access its contents.
- **Directory navigation:** A directory must be entered before retrieving a file inside it.
- **Evidence-based reporting:** Record the actual command output and distinguish observed results from assumptions.

---

[Back to Module Index](./README.md)
