# HTB — Keeper Writeup

**Author:** Vibhav Chennamadhava
**Difficulty:** Easy
**OS:** Linux
**Platform:** Hack The Box

---

## Overview

## 🧭 Introduction

Keeper is one of those HTB machines that looks simple on the surface but actually teaches a lot of **real-world security lessons**. There were no crazy exploits or advanced reverse engineering involved. Instead, this box focused on **misconfigurations, human mistakes, and poor credential handling** — things that happen all the time in real environments.

In this write-up, I’ll walk through how I approached the machine, the hurdles I faced, and how everything connected step by step.

---

## Methodology / Steps

## 🔍 Enumeration — Figuring Out What’s There

The first thing I did was run an Nmap scan to see what services were exposed. The scan showed only two open ports:

* **Port 22 (SSH)**
* **Port 80 (HTTP)**

At first, SSH wasn’t very useful since I didn’t have any credentials. That pushed me toward the web service on port 80, which is usually a good starting point anyway.

---

## 🌐 Web Enumeration — First Small Roadblock

When I opened the IP address in my browser, I noticed the site redirected me to `tickets.keeper.htb`. The page didn’t load correctly at first, which was confusing. After a bit of trial and error, I realized the issue wasn’t the site itself — it was **DNS resolution**.

To fix this, I added the domain to my `/etc/hosts` file. Once that was done, the site loaded normally. This was my first reminder that **web applications often depend on hostnames**, not just IP addresses.

---

## 🔐 Request Tracker & Default Credentials

The web application turned out to be **Request Tracker (RT)**, an open-source ticketing system. After identifying the software, I did a quick check to see if it had default credentials.

Surprisingly (but realistically), logging in with:

* **Username:** root
* **Password:** password

worked immediately.

This step felt almost too easy, but it highlighted a major real-world issue: **systems are often deployed and never properly hardened**.

---

## 👤 Finding Real Credentials (Foothold)

Once inside the RT dashboard, I started looking around instead of rushing ahead. Under the user management section, I found another user named **lnorgaard**. While checking the user details, I noticed something careless but critical — a **plaintext password** stored inside a comment field.

Trying this password over SSH worked, and I finally had shell access as a normal user.

At this point, the box shifted from web exploitation to Linux enumeration.

---

## 🏠 User Enumeration — Something Interesting Appears

Inside the user’s home directory, I found a ZIP archive. After extracting it, I noticed two files that immediately stood out:

* A **KeePass database file (.kdbx)**
* A **memory dump (.dmp)**

At first, I wasn’t sure what to do with them. But seeing both files together was a big hint that KeePass had been running and that its memory might contain sensitive data.

---

## 🧠 KeePass Exploitation — The Biggest Hurdle

KeePass is usually very secure, so I initially assumed this might be a dead end. After some research, I learned about **CVE-2023-32784**, which affects how KeePass handles master passwords in memory.

In simple terms, KeePass doesn’t fully clean up memory while a password is being typed. If a memory dump is taken, parts of the master password can still be recovered.

Using a public proof-of-concept tool on the memory dump,
which was this -> https://github.com/vdohney/keepass-password-dumper

I was able to recover most of the master password. The output wasn’t perfect, though — first character was missing or slightly off, this was another small hurdle.

To solve this, I used common sense and a quick Google search, which helped me correct the phrase and successfully unlock the KeePass database.

---

## 🔓 Extracting the Keys

Once the database was open, things became much clearer. Inside, I found stored credentials and **SSH private keys**, including one belonging to the **root user**.

The key was in PuTTY format, which didn’t work directly with OpenSSH. Converting it and fixing the file permissions took a bit of tweaking, but once done, the key worked perfectly.

---

## 🚀 Privilege Escalation — No Exploit Needed

Instead of exploiting a binary or abusing sudo permissions, privilege escalation here was very straightforward:

* Use the recovered **root SSH private key**
* Log in directly as root

This was a good reminder that **credential exposure is often more dangerous than software vulnerabilities**.

---

## 🏁 Root Access & Final Thoughts

After logging in as root, grabbing the final flag confirmed full system compromise.

What made Keeper interesting wasn’t technical difficulty, but how realistic the attack path felt. Every step depended on human mistakes:

* Default credentials
* Passwords stored in comments
* Sensitive memory dumps left behind
* SSH keys stored insecurely

---

## Findings

- Initial compromise came from weak operational practices (default credentials and plaintext password handling).
- Post-exploitation artifacts (KeePass dump + memory dump) enabled recovery of high-value credentials.
- Privilege escalation was achieved through key exposure rather than local kernel/service exploitation.

## Lessons Learned

* Enumeration is everything — rushing would have missed key details
* Small misconfigurations can chain into full compromise
* Memory dumps can completely bypass encryption
* Password managers still require secure environments
* Real-world hacking is often about patience and observation

---

## 🎯 Closing Thoughts

Keeper was a great example of how attackers don’t always need advanced exploits. Sometimes, all it takes is careful enumeration, basic research, and connecting the dots. For anyone new to HTB, this box is a perfect lesson in **thinking like an attacker while understanding real-world security failures**.


## Screenshots

Any supporting images for this writeup should be stored in [`assets/`](assets/).
