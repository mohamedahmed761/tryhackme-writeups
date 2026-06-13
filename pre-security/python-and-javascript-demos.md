# Python and JavaScript Demos

**Path:** Pre-Security → Software Basics
**Rooms:** Python Demo, JavaScript Demo
**Completed:** 2026-06-13

## What the rooms covered

First exposure to programming syntax through a guess-the-number game. Same logic implemented in both Python and JavaScript to show how the same problem looks in different languages.

Core concepts: variables, input/output, type conversion, conditionals (if/else), loops (until correct guess), and incrementing counters.

## Key concepts

**Variables** — named containers for data (e.g., `secret = 10`, `tries = 0`).

**Input/output** — `input()` in Python and `prompt()` in JavaScript read user input. `print()` in Python and `console.log()` in JavaScript display output.

**Type conversion** — `input()` returns text (string), so `int(text)` converts it to a number for comparison. Without conversion, you can't do maths on user input.

**Conditionals** — `if/elif/else` checks the guess against the secret and branches the program accordingly.

**Loops and counters** — the program loops until the user guesses correctly. A counter variable (`tries`) increments by 1 each round to track attempts.

**Comments** — `#` in Python, `//` in JavaScript. Notes for humans, ignored by the computer.

## Mental model — why two languages?

The same logic (input → check → output → repeat) works in any language. Python and JavaScript differ in syntax but not in concept. Variables, conditionals, loops, and functions exist in every modern language.

For cyber security: Python is the dominant scripting language for offensive tools, automation, exploit dev, and CTF solving. JavaScript matters for web security — it runs in browsers, powers most web apps, and is the language of cross-site scripting (XSS) attacks. Knowing both lets you read attack code in either environment.

## What clicked

Same logic, different syntax. Once you see the structure — input → check → branch → loop — porting between languages is mostly looking up how each one spells the same idea. Python `print()` is JavaScript `console.log()`. `if/elif/else` is `if/else if/else`. A `while` loop is a `while` loop. After the first translation the rest is muscle memory.

## What to revisit

Writing both versions of the guess-the-number game from scratch without looking at the demos. I can read the code now; typing it blind in Python then in JavaScript would force the syntax differences into memory before I hit rooms that expect me to modify or write code.
