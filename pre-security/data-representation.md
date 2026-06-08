# Data Representation

**Path:** Pre-Security → Software Basics
**Room:** [Data Representation](https://tryhackme.com/room/datarepresentation)
**Difficulty:** Easy · **Time:** ~45 min · **Completed:** 2026-06-06

## What the room covered

How computers turn 1s and 0s into colors and numbers. Two big ideas: RGB colour encoded as 24 bits = 3 bytes, and base systems (binary, decimal, hex, octal) with the math to convert between them.

Tasks: intro, Representing Colors (bits → bytes → hex colours), Numbers (binary ↔ decimal ↔ hexadecimal), conclusion.

## Key concepts

**Bit, byte, nibble.**
- **Bit** — one binary digit: 0 or 1. Two states.
- **Byte** (aka octet) — 8 bits. 2⁸ = **256** possible values (0 to 255).
- **Nibble** — 4 bits. Exactly one hex digit.

**RGB colour = 3 bytes.** Each channel (red, green, blue) gets one byte = 256 intensity levels. Total combinations: 256 × 256 × 256 = **16,777,216** colours.

**Hex is just shorthand for 4-bit groups.** Each nibble (4 bits) maps to one hex digit 0–F. A byte is two hex digits. Green channel value 10100011 in binary = A3 in hex. The colour 10100011 11101010 00101010 (24 bits) becomes `#A3EA2A`.

**Base systems are positional.** A number is each digit multiplied by the base raised to its position power.

Decimal 213:

```
213 = 2 × 10² + 1 × 10¹ + 3 × 10⁰
    = 200 + 10 + 3
```

Binary 1001:

```
1001 = 1 × 2³ + 0 × 2² + 0 × 2¹ + 1 × 2⁰
     = 8 + 0 + 0 + 1
     = 9
```

Hex 9BDF:

```
9BDF = 9 × 16³ + 11 × 16² + 13 × 16¹ + 15 × 16⁰
     = 36864 + 2816 + 208 + 15
     = 39,903
```

Same procedure for any base — octal (base 8) groups 3 bits per digit instead of 4.

**Quick lookup table.**

| Decimal | Hex | Binary |
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

## Mental model — why different bases exist

Computers physically can only represent two states (high/low voltage, north/south polarity, light on/off). So at the hardware level, everything is binary. Humans use decimal because we have ten fingers.

Hex (and octal) are a compromise. A byte in binary is `10100011` — eight characters, hard to read, easy to misread. The same byte in hex is `A3` — two characters, trivially convertible back to binary because each hex digit is exactly 4 bits. We use hex everywhere a human needs to look at raw bytes: colour codes, MAC addresses, memory addresses, file magic bytes, hash outputs, shellcode.

Binary, hex, and decimal are the same number expressed in different alphabets. The math doesn't change. The only thing that changes is which base you're more efficient at reading.

## Offensive angle

Encoding is everywhere in security. Knowing the bases means you can read what attackers and defenders are actually doing.

| Concept | Attacker / analyst use |
| --- | --- |
| Hex dumps | Reverse engineering binaries, packet inspection in Wireshark, forensic disk analysis. Every byte on disk is read as hex. |
| File magic bytes | Identify file type from the first few hex bytes: `4D 5A` = Windows PE (.exe), `7F 45 4C 46` = Linux ELF, `89 50 4E 47` = PNG. Used in malware analysis and to spot files with fake extensions. |
| Base64 / hex / URL encoding | Payload obfuscation. The same shellcode in raw bytes vs base64 vs URL-encoded looks completely different — useful for sneaking past signature-based WAFs and AV. |
| ASCII shellcode | Real exploit dev sometimes restricts payloads to printable ASCII (0x20–0x7E) so the bytes survive being passed through string-handling code. |
| Alternate IP encodings | `127.0.0.1` can be written in decimal (`2130706433`), hex (`0x7F000001`), or octal (`0177.0.0.1`). Classic SSRF / firewall bypass trick — the parser accepts it, the blocklist doesn't see it. |
| XOR encryption | Operates bit-by-bit. Knowing binary makes you able to break weak XOR by hand or recognise it from a hex dump. |
| Password hashes | MD5, SHA-1, SHA-256 etc. always output as hex strings. Length tells you the algorithm at a glance (32 chars = MD5, 40 = SHA-1, 64 = SHA-256). |
| Memory addresses | Exploit dev (`0xdeadbeef`, NOP sleds `\x90\x90\x90`). Every buffer-overflow tutorial assumes you can read hex fluently. |

The pattern: if you can't read hex and binary fluently, the actual artefacts of an intrusion (a packet capture, a memory dump, an obfuscated payload) just look like noise.

## What clicked

Hex isn't a separate system to memorise — it's just binary with the digits paired up. Seeing the table where every hex digit maps to exactly 4 bits made the whole thing collapse from "weird new base" to "shorthand." After that, `#A3EA2A` is just three pairs (`A3` `EA` `2A`) = three bytes = R, G, B intensities.

## What to revisit

Fast mental conversion. I can convert with the formula but it's slow. Worth drilling: every power of 2 up to 256 (1, 2, 4, 8, 16, 32, 64, 128), and the 0–F ↔ 0000–1111 table cold. That speed pays off the moment I'm staring at a hex dump in a forensics or exploit-dev room and don't want to break flow for arithmetic.
