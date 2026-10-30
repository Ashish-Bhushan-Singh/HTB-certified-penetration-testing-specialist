# Section 06: Basic Tools

**Module:** 02 — Getting Started  
**Topic:** Service Banner Grabbing  
**Status:** Completed — answer documented

---

## Questions & Answers

### Question 1: Grab the Server Banner

**Question**

Apply what you learned in this section to grab the banner of the provided server and submit it as the answer.

**Command Used**

```bash
netcat 154.57.164.72 31612
```

**Observed Output**

```text
SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.1
```

**Detailed Explanation**

The `netcat` (`nc`) utility can establish a network connection to a specified IP address and port. In this exercise, the connection was made to `154.57.164.72` on port `31612`.

The server returned the banner:

`SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.1`

This banner provides information about the service listening on the remote port:

| Component | Meaning |
|---|---|
| `SSH-2.0` | The service identifies its protocol version as SSH version 2.0. |
| `OpenSSH_8.2p1` | The server identifies its SSH implementation as OpenSSH 8.2p1. |
| `Ubuntu-4ubuntu0.1` | The remaining string identifies the Ubuntu package build or revision associated with the software. |

**Why Banner Grabbing Matters**

Banner grabbing is a basic reconnaissance technique used to identify services and gather information about their software versions. This information can help an authorized security tester decide which technologies to investigate further.

However, a banner alone does not prove that a service is vulnerable. The reported version and package information should be verified against other available evidence before drawing conclusions.

**Short Answer**

```text
SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.1
```

---

[Back to Module Index](./README.md)
