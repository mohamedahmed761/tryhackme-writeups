# Cryptography Concepts

**Path:** Pre-Security → Attacks and Defenses
**Room:** [Cryptography Concepts](https://tryhackme.com/room/cryptographyconcepts)
**Difficulty:** Easy · **Time:** ~60 min · **Completed:** 2026-06-15

## What the room covered

What cryptography actually is and why the padlock in your browser matters. Three real ideas: **symmetric encryption** (one shared key, fast, has a key-distribution problem), **asymmetric encryption** (two mathematically linked keys, slow, solves the distribution problem), and the **hybrid model** that HTTPS uses to get the best of both.

The Caesar cipher is used as a learning example to show how algorithm + key + plaintext → ciphertext. Caesar is broken in milliseconds by a computer — the point is the pattern, not the cipher itself.

## Key concepts

**The four core terms.**

| Term | What it is |
| --- | --- |
| Plaintext | A readable message. `HELLO`. |
| Ciphertext | The scrambled version. `KHOOR`. |
| Key | The secret value that controls scrambling. With Caesar, just a number. With AES, 128/256 bits of random data. |
| Algorithm | The public recipe. Everyone can know it. Security comes from the key being secret. |

Kerckhoffs's principle in one line: **the algorithm is public; only the key is secret.** Real-world ciphers (AES, RSA, ChaCha20) are openly published and reviewed by thousands of cryptographers. If your security depends on hiding the algorithm, you've already lost.

**Symmetric encryption — one key, both directions.**
- Same key encrypts and decrypts. Both parties need it.
- Fast. Designed for bulk data — files, disk, network traffic.
- The classic: AES (128 or 256 bit), ChaCha20.
- Catch: how do you share the key safely in the first place? → **key distribution problem**.

**Asymmetric encryption — two mathematically linked keys.**
- **Public key** — give it to anyone. Used to encrypt to you.
- **Private key** — you keep this. Used to decrypt what was encrypted to you.
- Anything encrypted with one key can only be decrypted with the other.
- The math binding them is one-way: deriving the private from the public would take thousands of years on a normal computer.
- The classic: RSA, ECDH, Ed25519.
- Trade-off: much slower than symmetric. Not used for bulk data.

**HTTPS is the hybrid.** Asymmetric for the handshake (agree on a shared secret without an eavesdropper learning it), symmetric for the actual data flow. Best of both worlds.

**Certificates and CAs.** Anyone can hand you a public key — the question is whether it really belongs to the site you think you're talking to. A **certificate** is a public key signed by a **Certificate Authority** (CA) that the browser already trusts. The browser checks the signature chain on every HTTPS connection; mismatch or expiry triggers the warning page.

## Mental model — cryptography as three different jobs

Most people use "encryption" to mean any kind of cryptographic operation. The room sorts out two of them; this section adds a third (hashing) because the cyber security work that follows will only make sense if you keep them straight.

1. **Symmetric encryption** — keep data secret, fast. "Two people share one key." Good for whole-disk encryption, VPN tunnels, the bulk-data phase of HTTPS.
2. **Asymmetric encryption** — share secrets safely with someone you've never met. "Two keys, one public, one private." Good for kicking off a connection (HTTPS handshake), signing software, exchanging keys.
3. **Hashing** — not in this room, but you'll meet it everywhere next. **No key. One-way. Same input → same fixed-length output.** Good for password storage and integrity checks.

Real systems chain these three constantly. HTTPS uses asymmetric to establish a session, symmetric for bulk data, and hashing inside both for integrity. A password reset email is hashing (server-side) plus asymmetric (the TLS connection delivering the link).

## Encryption vs hashing — the distinction

This is the single concept beginners get wrong most often. It matters because the two are used for different things and have different failure modes.

| | Encryption (symmetric or asymmetric) | Hashing |
| --- | --- | --- |
| Direction | Two-way. Encrypt now, decrypt later. | One-way. You cannot reverse a hash to recover the input. |
| Needs a key? | Yes. The key is the secret. | No. Anyone can hash anything. |
| Output length | Roughly the size of the input. | Fixed length, regardless of input size. SHA-256 always = 256 bits. |
| Used for | Hiding data so only authorised people can read it. | Proving data wasn't tampered with. Storing passwords safely. |
| Examples | AES, ChaCha20, RSA | MD5 (broken), SHA-1 (broken), SHA-256, bcrypt, Argon2 |

**Why hashing is one-way matters.** When a server stores your password, it doesn't store the password itself — it stores the hash. When you log in, the server hashes what you typed and compares the result. If the database leaks, attackers get the hashes, not the passwords. They then have to crack the hashes (try inputs until one matches), which is the entire battle of credential dumping.

**Why "hashing your data to protect it" is wrong.** If you hash a secret to send it, the receiver can't read it either. That's not encryption — that's destruction.

**Encryption with hashing inside.** Modern protocols combine both: encryption for confidentiality, hash-based MACs (HMAC) for integrity. The encryption hides the data; the hash proves it wasn't modified in transit.

The shorthand to remember: **encryption = read-protection (two-way). Hashing = tamper-detection / fingerprinting (one-way).**

## Offensive angle

Crypto is one of the most leveraged areas in offence — not because attackers break the math, but because they break the implementation. Algorithms are usually solid; the way humans wire them up usually isn't.

| Concept | Attacker move |
| --- | --- |
| Weak algorithms | DES, MD5, SHA-1, RC4 are dead. Old systems still use them. Identify by hash length / cipher name, then crack offline (Hashcat, John). |
| Caesar / classical ciphers | CTF staples. Try all 25 shifts, look for English letter frequency, look for the word "flag" / "the". Cyberchef does it instantly. |
| Stolen keys | If you find the symmetric key, you have all past and future traffic. Keys leaked in source code (`config.py`, `.env`, GitHub) are a top recon target. |
| Key distribution failure | Pre-shared symmetric keys sent over insecure channels (email, Slack) = adversary captures everything. Classic blue-team misstep. |
| Reused passwords / hashes | Hash from one breach gets thrown at every other login. "Credential stuffing." |
| Unsalted password hashes | If the server hashes passwords without a unique salt per user, rainbow tables crack them in seconds. `bcrypt` and `argon2` salt automatically; raw `md5` / `sha256` of a password is broken practice. |
| Hashcat / John the Ripper | Throw a wordlist at captured hashes. The same dictionary attack from the Become a Hacker room, but against the hashed form of passwords. |
| HTTPS cert errors | Self-signed cert, expired cert, name mismatch → user clicks through. Attacker on a public Wi-Fi runs a fake AP and presents their own cert hoping the user accepts. |
| Cert authority compromise | If a trusted CA gets popped, the attacker can mint certificates for any domain. DigiNotar 2011 is the canonical case. |
| TLS downgrade | Force the connection to use an old, broken cipher suite (POODLE, BEAST, FREAK). Defence is configuring servers to disable everything below TLS 1.2. |
| Side channels | Even perfect math can leak via timing, power consumption, error messages. The padding-oracle attack on CBC mode is the classic. |
| Padding oracle / Bleichenbacher | The server tells you whether the padding was valid; that single bit leaks enough to recover the plaintext. |
| Hashing for IDs | Apps that hash a user ID and use it as a session token — if the hash is reversible / predictable, you walk into other accounts. |

**The repeating pattern:** attackers don't break AES, they steal the key. They don't break SHA-256, they grab the hash database and crack the weak passwords. Crypto failures are almost always implementation failures, not math failures.

## Practical — the Caesar walkthrough

The room used Caesar cipher with key = 3.

```
Encrypt HELLO (shift +3):
  H → K
  E → H
  L → O
  L → O
  O → R
  HELLO → KHOOR

Decrypt KHOOR (shift -3):
  K → H, H → E, O → L, O → L, R → O → HELLO
```

The room's puzzles practised both directions:
- `CYBER` with key 5 → `HDGJW`
- `FVZCYR PNRFNE PVCURE` with key 13 (ROT13) → `SIMPLE CAESAR CIPHER`

That's literally all there is to Caesar. Useful to learn the encryption / key / algorithm vocabulary, useless for real secrecy.

## What clicked

The **algorithm-public, key-secret** framing. Before, I thought encryption was about hiding *how* the encryption worked. It's the opposite: the whole world is invited to study AES and try to break it — and after 25 years they still haven't. The math is exposed; the secret is the key. That's why leaked keys and bad key handling cause every real-world crypto failure, not broken ciphers.

The hybrid HTTPS flow also clicked: asymmetric is too slow for bulk data but fast enough to share one symmetric key, and symmetric is fast enough to do everything else. The padlock in the browser is both of them working in sequence.

## What to revisit

- **Hashing in depth** — the room skipped it. Need to drill: salt, bcrypt vs argon2, common hash lengths by algorithm, how Hashcat / John actually work in practice.
- **TLS handshake step-by-step** — ClientHello → ServerHello → cert → key exchange → finished. The room shows the concept; understanding the actual byte sequence helps for web pentesting later.
- **Common modes of operation** — ECB / CBC / GCM. ECB leaks patterns (the infamous penguin image), CBC enables padding-oracle attacks, GCM is the modern default. Worth knowing why.
- **Real cipher names and key sizes** — AES-128 vs AES-256, RSA-2048 vs ECDSA P-256, ChaCha20-Poly1305. Recognising which crypto a target uses tells you what attacks are even on the table.
