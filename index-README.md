# CTF Writeups

A collection of CTF challenges I've solved, with detailed writeups covering the vulnerability, exploitation steps, and remediation.

## Index

| Challenge | Category | Difficulty | Summary |
|---|---|---|---|
| [Flask Admin Panel — OTP Bypass](./flask-otp-bypass/README.md) | Web Exploitation | Medium | Cracked admin password hash, then bypassed 2FA by reading the OTP directly from a readable (signed-but-unencrypted) Flask session cookie. |

*(Add a new row here each time you add a writeup folder.)*

## Structure

Each challenge lives in its own folder:

```
ctf-writeups/
├── README.md                     ← this file
├── flask-otp-bypass/
│   └── README.md
├── <next-challenge-slug>/
│   └── README.md
```

## Categories

- **Web Exploitation** — auth bypass, session/cookie issues, injection, logic flaws
- **Crypto** — weak crypto, hash cracking, custom cipher breaks
- **Pwn / Binary Exploitation**
- **Forensics**
- **Reverse Engineering**
- **Misc**

## Disclaimer

All writeups document challenges solved in legal, intentionally vulnerable CTF environments (official competitions, TryHackMe, HackTheBox, picoCTF, self-hosted labs, etc.). Techniques described are for educational purposes only.

## About

Maintained by [your name/handle]. Feedback and corrections welcome via issues/PRs.
