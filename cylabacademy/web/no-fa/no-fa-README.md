# No FA — 2FA Bypass via Client-Readable Flask Session

**Platform:** Cylabacademy
**Category:** Web Exploitation
**Difficulty:** Medium
**Tags:** `flask` `session-cookie` `weak-hashing` `2fa-bypass` `information-disclosure`

## Summary

This challenge chains together three separate weaknesses in a Flask admin panel to reach an authenticated flag: a weakly hashed credential store, a Flask session cookie that is signed but not encrypted, and a low-entropy OTP generated for a second-factor check. None of the individual bugs is exotic, but combined they defeat the app's 2FA protection without ever forging a cookie or brute-forcing the OTP space — living up to the challenge's name, "No FA."

## Exploit Chain

```text
Leaked users.db
      ↓
Unsalted SHA-256 hash
      ↓
RockYou / John
      ↓
admin password
      ↓
Admin login
      ↓
OTP generated
      ↓
OTP placed in Flask session cookie
      ↓
Decode readable session payload
      ↓
Recover OTP
      ↓
Submit OTP
      ↓
Authenticated as admin
      ↓
Flag
```

## Step-by-Step

### 1. Credential recovery

`users.db` was leaked/accessible and stored the admin password as an **unsalted SHA-256** hash. Because there was no salt, the hash was crackable against a precomputed wordlist rather than needing to be reversed mathematically (SHA-256 cannot be "inverted" — it was **cracked via an offline dictionary attack using RockYou**, with John the Ripper / hashcat).

```bash
john --wordlist=rockyou.txt --format=Raw-SHA256 hash.txt
```

This recovered the admin plaintext password, which was used to log in through the normal login form.

### 2. OTP issued for second factor

After password login, the app generated a 4-digit OTP using random.randint(1000, 9999) and stored it in the Flask session. Because Flask's default session is client-side, the OTP was included in the session cookie sent to the browser.

### 3. Reading the session cookie

This is the core vulnerability, and it's worth stating precisely:

> **Flask's default session cookie (`itsdangerous`-signed) provides integrity, not confidentiality.**

Flask serializes the session data and encodes it into the cookie, then signs the resulting value using SECRET_KEY. The signature provides integrity and authenticity, not confidentiality. Therefore, knowing the SECRET_KEY is not necessary to decode and read an existing session cookie; it would only be necessary to create or modify a cookie that Flask would accept as valid. Anyone holding the cookie can decode its serialized payload and, where compression is used, decompress it to recover the session data. `SECRET_KEY` prevents an attacker from **forging a modified cookie** that the server will accept; it does nothing to stop an attacker from **reading the existing payload**, since no encryption step ever occurs.

The Flask session cookie uses a serialized representation of the session
data. In this case, the payload was compressed, so it had to be decoded
and decompressed before the session contents could be read.

The important point is that no SECRET_KEY was required to read the
existing payload. The key would be required to create a modified,
validly signed session cookie.

{
  "logged": "false",
  "otp_secret": "9767",
  "otp_timestamp": 1787586541.2186317,
  "username": "admin"
}

The OTP was plainly visible in the decoded payload — no signature-forging or key recovery required.

### 4. Submitting the recovered OTP

The recovered OTP was submitted to the 2FA endpoint, authenticating the session as admin and exposing the flag.

## Vulnerability Classification

| Weakness | Role in the chain | Severity |
|---|---|---|
| Unsalted SHA-256 password storage | Enabled offline dictionary cracking of the admin credential | Primary |
| Session cookie confidentiality (info stored in a signed-but-unencrypted cookie) | Allowed direct reading of the OTP value, bypassing 2FA entirely | **Primary — this is the actual 2FA bypass** |
| 4-digit `random.randint()` OTP space | Low entropy, has a small search space and, based on the provided code, no visible rate limiting or attempt counter; however, it was not necessary to brute-force the OTP because the OTP was directly exposed in the session.

The write-up deliberately keeps this ordering: the OTP was **read**, not **guessed**. The small keyspace is a real weakness worth flagging for defenders, but it should not be described as brute-forced since that isn't what was demonstrated.

## Root Causes

1. Passwords hashed with unsalted SHA-256 instead of a slow, salted KDF (bcrypt/argon2/scrypt).
2. Sensitive second-factor material, such as the OTP, should not be stored in Flask's client-side session cookie. Store the OTP and its verification state server-side, for example in a server-side session store or database, and keep only an opaque session identifier in the client cookie.
3. OTP generated with a small, guessable keyspace and no rate-limiting on the verification endpoint (defense-in-depth gap, not the primary hole).

## Remediation

- Store passwords with a salted, slow hash function (argon2id preferred).
- Never place secrets (OTPs, tokens, roles used for authorization decisions) inside the default Flask session cookie. Use a server-side session backend (Redis, database) so the client only holds an opaque session ID.
- If sensitive data must live client-side, encrypt it, don't just sign it.
- Increase OTP entropy (6+ digits) and enforce rate-limiting / lockout on the verification endpoint.

## Disclaimer

This write-up documents a CTF challenge solved in an isolated, intentionally vulnerable lab environment. The techniques described (offline hash cracking, session payload inspection) are standard web-security assessment techniques and are documented here for educational purposes only.
