# Encoding & Data Representation Reference

Combined reference covering numerical bases (binary, hex, decimal) and character encoding (ASCII, Unicode, UTF-8/16, URL encoding, base64). Use as quick lookup during CTFs, forensics, web pentest, and exploit development.

## Numerical bases

### Bit / byte / nibble

- **Bit** — one binary digit (0 or 1). Two states.
- **Nibble** — 4 bits = one hex digit.
- **Byte** (aka octet) — 8 bits = two hex digits. 2⁸ = **256** possible values (0–255).

### Hex ↔ binary ↔ decimal

| Dec | Hex | Binary |
| --- | --- | --- |
| 0 | 0 | 0000 |
| 1 | 1 | 0001 |
| 2 | 2 | 0010 |
| 3 | 3 | 0011 |
| 4 | 4 | 0100 |
| 5 | 5 | 0101 |
| 6 | 6 | 0110 |
| 7 | 7 | 0111 |
| 8 | 8 | 1000 |
| 9 | 9 | 1001 |
| 10 | A | 1010 |
| 11 | B | 1011 |
| 12 | C | 1100 |
| 13 | D | 1101 |
| 14 | E | 1110 |
| 15 | F | 1111 |

### Powers of 2 (drill these)

| Exponent | Value |
| --- | --- |
| 2⁰ | 1 |
| 2¹ | 2 |
| 2² | 4 |
| 2³ | 8 |
| 2⁴ | 16 |
| 2⁵ | 32 |
| 2⁶ | 64 |
| 2⁷ | 128 |
| 2⁸ | 256 |
| 2¹⁰ | 1,024 (1 KB) |
| 2¹⁶ | 65,536 |
| 2²⁰ | 1,048,576 (1 MB) |
| 2³² | 4,294,967,296 (max unsigned 32-bit) |

### File magic bytes (first bytes of a file)

| Magic (hex) | File type |
| --- | --- |
| `4D 5A` | Windows PE (`.exe`, `.dll`) — "MZ" |
| `7F 45 4C 46` | Linux ELF binary — `\x7FELF` |
| `89 50 4E 47 0D 0A 1A 0A` | PNG |
| `FF D8 FF` | JPEG |
| `47 49 46 38` | GIF ("GIF8") |
| `25 50 44 46` | PDF ("%PDF") |
| `50 4B 03 04` | ZIP / DOCX / XLSX / JAR ("PK") |
| `52 61 72 21` | RAR ("Rar!") |
| `CA FE BA BE` | Java class file |

## ASCII printable range

- `0x20` = space
- `0x21`–`0x2F` = punctuation (`! " # $ % & ' ( ) * + , - . /`)
- `0x30`–`0x39` = digits 0–9 (`0x30` = '0')
- `0x41`–`0x5A` = uppercase A–Z (`0x41` = 'A')
- `0x61`–`0x7A` = lowercase a–z (`0x61` = 'a')
- `0x7E` = ~

Control chars worth knowing:
- `0x00` = NUL (string terminator in C)
- `0x07` = BEL (terminal bell)
- `0x09` = TAB
- `0x0A` = LF (Unix newline)
- `0x0D` = CR (Windows uses CRLF = `0x0D 0x0A`)
- `0x7F` = DEL

## Unicode quick-reference

Format: `U+XXXX` (hex code point).

- `U+0041` = 'A'
- `U+0061` = 'a'
- `U+2615` = ☕
- `U+2658` = ♘
- `U+1F525` = 🔥 (astral plane, needs surrogate pair in UTF-16)

UTF byte sizes:
- **UTF-8** — 1–4 bytes per char. ASCII range stays 1 byte (backwards-compatible).
- **UTF-16** — 2 or 4 bytes (4 = surrogate pair for code points above `U+FFFF`).
- **UTF-32** — always 4 bytes per code point.

## URL encoding (common)

| Char | Encoded |
| --- | --- |
| space | `%20` |
| `!` | `%21` |
| `"` | `%22` |
| `#` | `%23` |
| `$` | `%24` |
| `%` | `%25` |
| `&` | `%26` |
| `'` | `%27` |
| `/` | `%2F` |
| `:` | `%3A` |
| `;` | `%3B` |
| `<` | `%3C` |
| `=` | `%3D` |
| `>` | `%3E` |
| `?` | `%3F` |
| `@` | `%40` |

Double-encoding: `%25` decodes to `%`, so `%2525` decodes to `%25` then to `%`. Classic WAF bypass.

## Base64

- 64-character alphabet: A–Z, a–z, 0–9, `+`, `/`
- Padding character: `=`
- 3 input bytes → 4 output characters
- Output length always a multiple of 4
- Recognise: alphanumeric string ending in `=` or `==`, length divisible by 4, uses `+` / `/` (or `-` / `_` for URL-safe base64)

Byte-length math: encoded length = ceil(input_bytes / 3) × 4. So 100 base64 chars decode to ~75 bytes.

## Encoding-confusion attack surface (cheat list)

- **Charset mismatch** — WAF reads as UTF-8, backend reads as Latin-1. Payloads that look harmless to one look weaponised to the other.
- **Unicode normalisation** — `é` as one code point vs `e` + combining accent. Filters checking one form miss the other. NFKC can also collapse exotic chars into ASCII equivalents that bypass blocklists.
- **Homoglyphs** — lookalike letters from different scripts (Cyrillic `а` vs Latin `a`). Phishing domains, social engineering filenames.
- **Overlong UTF-8** — same character encoded with extra bytes. Legacy parsers accepted these; `/` as `%c0%af` was the classic IIS directory traversal.
- **Null byte injection** — `%00` truncates many C-string-based parsers, hides what comes after.
- **CRLF injection** — `%0d%0a` smuggles new HTTP headers (response splitting) or new log lines (log injection).
- **Double / triple URL encoding** — `%2525`. Decodes one layer at the WAF, the rest at the app.

## Use this reference for

Quick lookup during:
- Hex dump reading (forensics, malware analysis)
- File-type identification when extensions lie
- Web pentest filter-bypass work
- CTF crypto / stego challenges
- Exploit dev (shellcode constraints, memory addresses)
- Reading packet captures and base64-encoded C2 traffic
