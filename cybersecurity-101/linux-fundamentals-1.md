# Linux Fundamentals Part 1

**Path:** Cyber Security 101 → Linux Fundamentals
**Room:** [Linux Fundamentals Part 1](https://tryhackme.com/room/linuxfundamentalspart1)
**Difficulty:** Info · **Time:** ~20 min · **Completed:** 2026-06-15

> Related: [linux-cli-basics.md](../pre-security/linux-cli-basics.md) covers the same core commands from the newer Pre-Security path. See also the [Linux commands cheat sheet](../references/linux-commands.md).

## What the room covered

What Linux is and where it runs (web servers, phones, cars, PoS systems, traffic controllers), the terminal as the primary interface, and the first batch of commands you need to actually get anywhere on a Linux box: `echo`, `whoami`, `ls`, `cd`, `cat`, `pwd`, `find`, `grep`, and the shell operators `&`, `&&`, `>`, `>>`.

## Key concepts

**Linux is a family, not one OS.** "Linux" is an umbrella term for many UNIX-based distributions — Ubuntu, Debian, Kali, CentOS, Fedora, Arch. They share the kernel; they differ in package manager, defaults, and target audience. Ubuntu is the room's choice.

**Talking to Linux.** No GUI is required. Everything happens through the terminal — a text-based interface where you type commands and read responses. In cyber security work, the terminal isn't optional; most tools are CLI-only.

**First batch of commands.**

| Command | What it does |
| --- | --- |
| `echo <text>` | Print text back. Wrap in double quotes if it contains spaces. |
| `whoami` | Print the current username. |
| `ls` | List contents of the current directory. |
| `cd <dir>` | Change to another directory. `cd ..` goes up one level. |
| `cat <file>` | Print a file's contents to the terminal. |
| `pwd` | Print the full path of the current directory ("print working directory"). |

**Searching.** Two tools for finding things fast without wandering around with `ls` and `cd`.

| Command | Use |
| --- | --- |
| `find -name <filename>` | Search current directory tree for a file by name. |
| `find -name "*.txt"` | Wildcard search — every file with a matching extension. |
| `grep "<value>" <file>` | Search inside a file's contents for a string. |
| `grep -R "<value>" <path>` | Recursive search through every file and subfolder. |
| `wc -l <file>` | Count lines in a file. Useful for measuring big log files. |

Combined: `find` locates the file; `grep` reads what's in it.

**Shell operators.** Small symbols that change how commands run and where output goes.

| Operator | Effect |
| --- | --- |
| `&` | Run the command in the background so the terminal stays free. |
| `&&` | Chain — run the next command only if the first succeeded. |
| `>` | Redirect output to a file, overwriting existing contents. |
| `>>` | Redirect output to a file, appending to existing contents. |

Example chain: `echo "hey" > welcome` creates a file named `welcome` containing "hey". Then `echo "hello" >> welcome` appends "hello" on a new line instead of overwriting.

## Mental model

Every Linux session is you asking the same three questions: **where am I** (`pwd`), **what's here** (`ls`), **how do I get somewhere else** (`cd`). Everything else builds on that loop — `cat` reads what you find, `find` and `grep` skip the manual walk when the tree is big, and shell operators glue results together.

Once this rhythm clicks, the terminal stops feeling foreign. Every pentest, every CTF, every SOC investigation starts by walking a filesystem the same way.

## Offensive angle

| Concept | Attacker use |
| --- | --- |
| `whoami` | First command after landing on a shell. Am I root? Regular user? Service account? Determines the escalation path. |
| `ls -la` | Show hidden files (anything starting with a dot). Config files, credentials, SSH keys, `.bash_history` all hide by default. |
| `cat /etc/passwd` | Enumerate all local users. Readable by everyone by default. Combined with a service account list, you know who to target. |
| `find / -perm -4000` | Find SUID binaries — files that run as their owner regardless of who executes them. Classic privilege escalation lead. |
| `find / -writable` | Find world-writable files. If a root cron script is writable by you, you own root. |
| `grep -R "password" /etc/` | Hunt hardcoded credentials across config files. Almost every pentest turns up at least one. |
| `grep -R "api_key\|token\|secret" ~/` | Same idea against a user's home directory. Development machines leak keys constantly. |
| `wc -l access.log` | Sizing a log file before deciding whether to grep it or exfiltrate the whole thing. Useful defensively too for spotting unusual volumes. |
| `>` and `>>` | Attackers use these to plant backdoors: append a rogue key to `~/.ssh/authorized_keys` (`>>`) or overwrite a legitimate script (`>`). |
| `&` | Run a reverse shell or persistence loop in the background so it survives your logout. |
| `.bash_history` | If a defender or another user has been sloppy, this file shows every command they ran. Credentials in command lines end up here. |

The pattern: attackers use the same commands as anyone else. What separates a pentester from a first-day sysadmin is knowing **which files leak what**, and knowing to look immediately.

## Practical — the room's chained workflow

Room's build-up in one flow:

```bash
# where am I, who am I
tryhackme@linux1:~$ pwd
/home/tryhackme
tryhackme@linux1:~$ whoami
tryhackme

# what's here
tryhackme@linux1:~$ ls
Desktop  Documents  Pictures  folder1

# find a file by name (don't know where it lives)
tryhackme@linux1:~$ find -name passwords.txt
./folder1/passwords.txt

# search inside a huge log for one specific IP
tryhackme@linux1:~$ grep "81.143.211.90" access.log

# save the result to a file, then append notes
tryhackme@linux1:~$ grep "81.143.211.90" access.log > suspicious.txt
tryhackme@linux1:~$ echo "followed up 2026-06-15" >> suspicious.txt
```

The same six-command loop — `pwd`, `whoami`, `ls`, `find`, `grep`, redirect — shows up in almost every real Linux workflow, offensive and defensive.

## What clicked

The redirection operators. `>` and `>>` turn any command that prints something into a file-writing tool without needing a text editor. `echo hey > welcome` is a two-word "create a file with this content in it." That's a shortcut I'll use forever — dropping notes mid-workflow, saving grep results for later, appending to logs. It also explains half of what shell scripts actually do.

## What to revisit

- `find` with more flags — `-perm`, `-mtime`, `-user`, `-size`. The room only touched `-name` but `find` is the tool for privilege-escalation enumeration.
- `grep` with regex — word boundaries, alternation (`|`), the `-E` extended flag. Simple string matching only takes you so far.
- Combining commands with pipes (`|`) — not covered here, but the natural next step after redirection. `cat file | grep pattern | wc -l` is the everyday pattern.
