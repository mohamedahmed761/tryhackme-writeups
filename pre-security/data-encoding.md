# Data Encoding

**Path:** Pre-Security → Software Basics
**Room:** [Data Encoding](https://tryhackme.com/room/dataencoding)
**Difficulty:** Easy · **Time:** ~45 min · **Completed:** 2026-06-06

## What the room covered

How computers turn bytes into readable characters. Three layers: ASCII (7-bit English-only), ISO-8859 (8-bit regional patches for European languages), and Unicode (universal standard with UTF-8/16/32 as the actual byte-level encodings).

Tasks: intro, ASCII, Unicode, conclusion.

Builds directly on the Data Representation room — a byte is just a number, character encoding is the agreement about which number means which symbol.

## Key concepts

**ASCII — the 7-bit foundation.** American Standard Code for Information Interchange, 1963. 128 codes (0–127) cover English letters, digits, punctuation, and control characters. Each character has a fixed number; the alphabet is in order, so once you know one letter's code you can derive the rest.

Key landmarks:
- `0x30` = `0` (digits start at 48 decimal)
- `0x41` = `A` (uppercase starts at 65)
- `0x61` = `a` (lowercase starts at 97)
- `0x07` = BEL, `0x0A` = newline (LF), `0x7F` = DEL

**"TryHackMe" on disk.** Saved as ASCII, the string `TryHackMe\n` is just these bytes:

```
Hex:    54 72 79 48 61 63 6b 4d 65 0a
ASCII:  T  r  y  H  a  c  k  M  e  \n
```

Every text file is this — a stream of bytes plus an agreement on what each byte means.

**ISO-8859 — 8-bit regional patches.** ASCII used 7 of 8 bits. The 8th bit gave 128 more codes, but not enough for all of Europe in one standard. So multiple ISO-8859 variants exist:
- ISO-8859-1 (Latin-1): Western European — German, French, Spanish, etc.
- ISO-8859-2 (Latin-2): Central/Eastern European — Polish, Czech, Romanian, etc.

If you save a file as Latin-1 and open it as Latin-2, non-English characters render wrong. Ø saved in Latin-1 reads as Ř in Latin-2. This is the classic "mojibake" problem.

**Unicode — one universal code point per character.** Covers every script in the world plus emoji. Format: `U+XXXX` where XXXX is a hex code point.

- `U+0041` = A
- `U+03A9` = Ω
- `U+3042` = あ (Hiragana "a")
- `U+9F8D` = 龍 ("dragon")
- `U+2615` = ☕
- `U+2658` = ♘ (white knight)
- `U+1F525` = 🔥

Unicode 17 defines ~157,000 characters, ~4,000 of them emoji.

**UTF — how Unicode is actually stored.**

| Encoding | Bytes per char | Notes |
| --- | --- | --- |
| UTF-8 | 1–4 (variable) | Backwards-compatible with ASCII (`U+0000`–`U+007F` are 1 byte). Dominant on the modern web. |
| UTF-16 | 2 or 4 | Common Latin/Cyrillic/Hanzi in 2 bytes, emoji/rare scripts use a surrogate pair (4 bytes). |
| UTF-32 | 4 (fixed) | Every code point gets 4 bytes. Simple, wasteful, rarely used in transit. |

Example — the fire emoji 🔥 (`U+1F525`):
- UTF-8: `F0 9F 94 A5` (4 bytes)
- UTF-16: `D83D DD25` (surrogate pair, 4 bytes)
- UTF-32: `0001F525` (4 bytes)

## Mental model — character encoding as a layered translation

1. **Bytes on disk** — raw numbers. No meaning yet.
2. **Character set** (Unicode, or the older ASCII / ISO-8859) — says which number stands for which symbol. Just a lookup table.
3. **Encoding** (UTF-8, UTF-16, ISO-8859-1) — says how to actually pack those numbers into bytes on disk or wire.

ASCII conflates layers 2 and 3 — the code IS the byte. Unicode splits them: code points are abstract identifiers (`U+XXXX`), and UTF-8/16/32 are different ways to serialise them. That's why a file can be saved "in UTF-8" or "in UTF-16" — same Unicode content, different bytes on disk.

Get the encoding wrong on either end and you get mojibake (もじばけ) — the bytes are read by the wrong lookup table.

## Offensive angle

Character encoding is one of the highest-yield abuse surfaces in security — nearly every input filter, normaliser, and parser has to make assumptions about it, and those assumptions break in interesting ways.

| Concept | Attacker / analyst use |
| --- | --- |
| ASCII range | Recognise traffic at a glance — bytes between 0x20 and 0x7E in a hex dump are likely plain text. Used for shellcode constraints (printable-ASCII shellcode) and for spotting strings in binaries (`strings` does this). |
| Control characters | `\0` (NULL) terminates C strings — inject a null byte and many parsers truncate. `\r\n` is HTTP line break — inject those and you get HTTP response splitting. |
| ISO-8859 vs UTF-8 | Charset confusion. If a WAF normalises as UTF-8 and the backend reads as Latin-1, payloads can survive sanitisation that wouldn't match in the WAF's view. |
| Unicode homoglyphs | `paypaʟ.com` — letters that look identical but have different code points. Used in phishing domains (IDN homograph attacks) and in social-engineering filenames. |
| Unicode normalisation | Same character has multiple representations (`é` as a single code point vs `e` + combining accent). Filters that check one form miss the other. `NFKC` normalisation can also flatten characters into ASCII in ways that bypass blocklists. |
| UTF-8 overlong encoding | Same character encoded with more bytes than necessary. Many old parsers accepted overlong forms, letting attackers smuggle `/` past path filters as `%c0%af` (classic IIS directory traversal). |
| Surrogate pairs (UTF-16) | Astral plane characters split across two 16-bit values. Some parsers mishandle one half of the pair, leading to truncation or injection. |
| URL encoding | Same byte expressed as `%XX`. Double-encoding (`%2525` decodes to `%25` then `%`) is the classic WAF bypass. |
| Base64 / hex / etc. | Payload obfuscation in C2 traffic, in malware droppers, and in CTF challenges. Recognise base64 by character set (A–Z, a–z, 0–9, +, /) and trailing `=` padding. |
| Encoding-aware regex | An attacker who knows the target system's encoding picks payloads that match the application but evade the filter. "`<script>`" might be blocked while its UTF-7 equivalent slips through into a Content-Type-misconfigured page. |

The meta-point: any time data crosses a boundary between systems (browser → server, server → database, log file → SIEM), there's a place where one side might decode it differently than the other. That gap is the attack surface.

## What clicked

The split between *character set* and *encoding*. I'd always treated "Unicode" and "UTF-8" as the same thing. They're not — Unicode is the catalogue of characters (each gets a number), UTF-8/16/32 are competing ways to serialise that number into actual bytes. "Save as UTF-8" vs "Save as UTF-16" produces totally different byte sequences for the same Unicode text. Once that landed, mojibake and charset-confusion bugs stopped being mysterious — they're just decoder/encoder mismatches.

## What to revisit

The actual UTF-8 byte structure (how the leading bits of each byte signal continuation), and UTF-16 surrogate pairs. The room shows the results but not the rules — worth drilling before any web exploit room where overlong encoding or surrogate-pair tricks might come up.
