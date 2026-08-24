# CTF Writeups

A collection of CTF challenges I've solved, with detailed writeups covering the vulnerability, exploitation steps, and remediation.

## Structure

Writeups are organized by **platform → category → challenge**:

```
ctf-writeups/
├── README.md                          ← this file (index)
├── tryhackme/
│   └── web/
│       └── no-fa/
│           └── README.md
├── htb/
│   ├── web/
│   │   └── <challenge-slug>/
│   │       └── README.md
│   └── crypto/
│       └── <challenge-slug>/
│           └── README.md
├── picoctf/
│   └── forensics/
│       └── <challenge-slug>/
│           └── README.md
```

- **platform** — where the challenge was hosted (e.g. `tryhackme`, `htb`, `picoctf`, `local` for self-hosted/practice labs)
- **category** — challenge type (`web`, `crypto`, `pwn`, `forensics`, `reversing`, `misc`)
- **challenge-slug** — folder named after the challenge (e.g. `no-fa`), containing a `README.md` writeup

## Index

| Platform | Category | Challenge | Difficulty | Summary |
|---|---|---|---|---|
| cylabacademy | Web | [No FA](./cylabacademy/web/no-fa/README.md) | Medium | Cracked admin password hash, then bypassed 2FA by reading the OTP directly from a readable (signed-but-unencrypted) Flask session cookie. |

*(Add a new row here each time you add a writeup.)*

## Categories

- **Web** — auth bypass, session/cookie issues, injection, logic flaws
- **Crypto** — weak crypto, hash cracking, custom cipher breaks
- **Pwn** — binary exploitation
- **Forensics**
- **Reversing**
- **Misc**

## Disclaimer

All writeups document challenges solved in legal, intentionally vulnerable CTF environments (official competitions, TryHackMe, HackTheBox, picoCTF, self-hosted labs, etc.). Techniques described are for educational purposes only.
