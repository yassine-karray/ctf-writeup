# Bandit — Full Writeups (Levels 0–33)

---

## Level 0 → 1

### Objective
SSH into the game server as bandit0.

### What I used
`ssh`

### Solution
```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
cat readme
```

### What I learned
Basic SSH syntax. The `-p` flag to specify a non-default port. Everything in OverTheWire starts here.

---

## Level 1 → 2

### Objective
Read a file named `-`.

### What I used
`cat`, path prefixing

### Solution
```bash
cat ./-
```

### What I learned
A filename starting with `-` gets interpreted as a flag by most commands. Prefixing with `./` tells the shell it's a path, not an option.

---

## Level 2 → 3

### Objective
Read a file with spaces in its name.

### What I used
`cat`, tab completion, quotes

### Solution
```bash
cat "spaces in this filename"
# or
cat spaces\ in\ this\ filename
```

### What I learned
Spaces in filenames need escaping or quoting. Tab completion handles this automatically  useful habit.

---

## Level 3 → 4

### Objective
Find a hidden file inside a directory.

### What I used
`ls -a`, `cat`

### Solution
```bash
cd inhere
ls -a
cat .hidden
```

### What I learned
`ls` doesn't show files starting with `.` by default. `-a` flag reveals them. Hidden files in Linux are just files with a dot prefix  no special permission magic.

---

## Level 4 → 5

### Objective
Find the only human-readable file among several files in `inhere/`.

### What I used
`file`

### Solution
```bash
cd inhere
file ./-file0*
```
Look for the one that says `ASCII text` — that's the one.
```bash
cat ./-file07
```

### What I learned
The `file` command detects file type by content, not extension. Wildcard `*` lets you run it on all files at once. Efficient.

---

## Level 5 → 6

### Objective
Find a file with specific properties: human-readable, 1033 bytes, not executable.

### What I used
`find`

### Solution
```bash
find ./inhere -type f -readable ! -executable -size 1033c
```

### What I learned
`find` is extremely powerful. `-size 1033c` means 1033 bytes (`c` = bytes). `!` negates a condition. You can chain multiple conditions in one command.

---

## Level 6 → 7

### Objective
Find a file somewhere on the server owned by user bandit7, group bandit6, 33 bytes.

### What I used
`find`

### Solution
```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

### What I learned
Searching from `/` floods output with permission errors. Redirecting stderr to `/dev/null` with `2>/dev/null` cleans it up. File ownership is searchable with `find`.

---

## Level 7 → 8

### Objective
Find the password next to the word "millionth" in a large file.

### What I used
`grep`

### Solution
```bash
grep "millionth" data.txt
```

### What I learned
`grep` searches file content by pattern. One line, instant result regardless of file size.

---

## Level 8 → 9

### Objective
Find the only line that appears once in a file full of duplicates.

### What I used
`sort`, `uniq`

### Solution
```bash
sort data.txt | uniq -u
```

### What I learned
`uniq -u` prints only unique lines — but it only detects adjacent duplicates, so you must `sort` first. Classic pipe pattern.

---

## Level 9 → 10

### Objective
Find a human-readable string preceded by `=` signs in a binary file.

### What I used
`strings`, `grep`

### Solution
```bash
strings data.txt | grep "=="
```

### What I learned
`strings` extracts printable text from binary files. Combined with `grep`, you can filter for specific patterns inside non-text files.

---

## Level 10 → 11

### Objective
Decode a base64-encoded file.

### What I used
`base64`

### Solution
```bash
base64 -d data.txt
```

### What I learned
Base64 is an encoding scheme, not encryption. Anyone with the encoded string can decode it instantly. `-d` flag decodes.

---

## Level 11 → 12

### Objective
Decode a ROT13-encoded file.

### What I used
`tr`

### Solution
```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

### What I learned
`tr` translates characters. ROT13 shifts each letter by 13 positions  applying it twice returns the original. Not encryption, just obfuscation.

---

## Level 12 → 13

### Objective
Decompress a file that has been compressed multiple times with different formats.

### What I used
`file`, `mv`, `gzip`, `bzip2`, `tar`, `xxd`

### What I tried first
This one broke me for a while. I saw `data.txt` and just tried `cat` — got garbage. Then I noticed it was a hex dump so I reversed it with `xxd -r`. Got a binary file. Ran `file` on it, said gzip. Decompressed it. Ran `file` again — bzip2. Decompressed that. tar. Then gzip again. Then bzip2 again. I genuinely lost count of how many layers there were. At some point I wasn't sure if I was going in circles or actually making progress. The mistake I kept making was forgetting to rename the file with the right extension before decompressing — `gzip -d` won't touch a file that doesn't end in `.gz`.

### Solution
```bash
mkdir /tmp/work && cp data.txt /tmp/work && cd /tmp/work
xxd -r data.txt > data.bin
# loop: file data.bin → rename → decompress → repeat
# gzip: mv data.bin data.bin.gz && gzip -d data.bin.gz
# bzip2: mv data.bin data.bin.bz2 && bzip2 -d data.bin.bz2
# tar: tar -xf data.bin
# keep going until file says ASCII text
```

### What I learned
`file` reads the actual file signature, not the extension  that's how you navigate this blind. Extensions are just labels, the magic bytes at the start of the file tell you the truth. Also learned to be methodical: run `file`, rename, decompress, repeat. Don't skip steps or you lose track of where you are. Real forensics work feels exactly like this.

---

## Level 13 → 14

### Objective
Use an SSH private key to log into the next level instead of a password.

### What I used
`ssh -i`

### Solution
```bash
ssh -i sshkey.private bandit14@localhost -p 2220
```

### What I learned
SSH supports key-based authentication. `-i` specifies the private key file. This is more secure than passwords and standard in professional environments.

---

## Level 14 → 15

### Objective
Submit the current level's password to port 30000 on localhost to get the next one.

### What I used
`ncat`

### Solution
```bash
cat /etc/bandit_pass/bandit14 | ncat localhost 30000
```

### What I learned
`ncat` (netcat) opens raw TCP connections. You can pipe data directly into a port. Fundamental networking tool.

---

## Level 15 → 16

### Objective
Same as previous but over SSL/TLS.

### What I used
`openssl s_client`

### Solution
```bash
openssl s_client -connect localhost:30001
# then paste the password
```

### What I learned
`openssl s_client` is the SSL/TLS equivalent of netcat. Encrypted connections work the same way underneath  just wrapped in TLS.

---

## Level 16 → 17

### Objective
Find which port on localhost (31000–32000) speaks SSL and returns credentials.

### What I used
`nmap`, `openssl s_client`

### Solution
```bash
nmap -sV localhost -p 31000-32000
# identify the SSL port that isn't just an echo server
openssl s_client -connect localhost:XXXXX
# submit password, get private key, save it, use it
```

### What I learned
`nmap -sV` probes ports and identifies services. Among multiple open ports, one gave back an RSA private key  saved it, `chmod 400`, used it for the next SSH login.

---

## Level 17 → 18

### Objective
Find the one line that differs between two files.

### What I used
`diff`

### Solution
```bash
diff passwords.old passwords.new
```

### What I learned
`diff` compares files line by line and shows what changed. The line only in `passwords.new` is the new password. Simple and powerful.

---

## Level 18 → 19

### Objective
`.bashrc` has been modified to log you out immediately on login.

### What I used
`ssh` with a command

### Solution
```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
```

### What I learned
SSH lets you pass a command directly  it runs the command and exits without ever spawning an interactive shell. Bypasses the malicious `.bashrc` entirely.

---

## Level 19 → 20

### Objective
Use a setuid binary to read a file you don't have permission to access.

### What I used
`./bandit20-do`

### Solution
```bash
ls -la bandit20-do   # setuid bit set, owned by bandit20
./bandit20-do cat /etc/bandit_pass/bandit20
```

### What I learned
Setuid binaries run with the permissions of their owner, not the caller. If a setuid binary owned by bandit20 runs a command, that command executes as bandit20. Core privilege escalation concept.

---

## Level 20 → 21

### Objective
Use a setuid binary that connects to a port, reads a line, and checks it against the current password.

### What I used
`ncat`, background processes, `&`

### Solution
```bash
echo "current_password" | ncat -l localhost 1234 &
./suconnect 1234
```

### What I learned
`&` runs a process in the background. You can set up a listener in one command and then run the binary that connects to it. Timing and process management matter.

---

## Level 21 → 22

### Objective
A cronjob is running as bandit22  find what it does and use it.

### What I used
`crontab`, `cat`

### What I tried first
I had no idea what cronjobs were when I hit this level. I knew something was running automatically but I didn't understand the mechanism. Spent a while researching  ended up going deep into how cron works, cron syntax, and then stumbled onto something interesting: zombie and ghost processes. Scripts that run invisibly in the background, leaving no obvious trace. That context actually made the level click — the cronjob is essentially a ghost script, running on a schedule, doing its thing whether you're watching or not.

### Solution
```bash
ls /etc/cron.d/
cat /etc/cron.d/cronjob_bandit22
cat /usr/bin/cronjob_bandit22.sh
# script writes the password to a temp file in /tmp
cat /tmp/t7O6lds9S0RqQh9sMcdr_beS8sZMFzqm  # filename from reading the script
```

### What I learned
Cronjobs run scripts automatically on a schedule  they don't need anyone logged in. If a script runs as another user and writes output to a world-readable location, you can just read it. Always check `/etc/cron.d/`. Also: understanding ghost/zombie processes gave me a much better mental model of what's happening on a Linux system at any given time beyond what you can see.

---

## Level 22 → 23

### Objective
Another cronjob but the temp filename is dynamically generated  you have to figure out what it computes.

### What I used
`cat`, `echo`, `md5sum`, `cut`

### What I tried first
Same approach as before  find the script, read it. But this time the filename wasn't hardcoded. The script was doing some computation to generate it. I read through it carefully and saw it was running `echo I am user bandit23 | md5sum` to get the hash, then using that as the filename. Once I understood the logic I just ran the same command myself.

### Solution
```bash
cat /usr/bin/cronjob_bandit23.sh
# understand what it computes
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
# get the hash, then:
cat /tmp/<that_hash>
```

### What I learned
Read scripts carefully and actually understand what they're computing  don't just skim. You can replicate any deterministic logic to predict output. The ability to read someone else's script and understand its behavior is a real skill, both for offense and defense.

---

## Level 23 → 24

### Objective
Write your own script, drop it in a directory the cronjob watches, and have it execute as bandit24.

### What I used
`bash scripting`, `chmod`, cronjob execution

### What I tried first
This was the first level where I had to write something instead of just read. Took me a bit to understand that I needed to make the output directory world-writable too  my first attempt the script ran fine but wrote to a directory I couldn't read from. Small detail that cost me time.

### Solution
```bash
mkdir /tmp/mydir
cat > /tmp/mydir/myscript.sh << 'EOF'
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/mydir/pass
EOF
chmod 777 /tmp/mydir/myscript.sh
chmod 777 /tmp/mydir
cp /tmp/mydir/myscript.sh /var/spool/bandit24/foo/
# wait for the cronjob to execute
cat /tmp/mydir/pass
```

### What I learned
Writable cron directories are a real privilege escalation vector. Permissions on the output location matter as much as the script itself  I learned that the hard way. If a privileged process can execute arbitrary code dropped in a directory, that directory is a security boundary.

---

## Level 24 → 25

### Objective
Brute-force a 4-digit PIN combined with the password against a daemon on port 30002.

### What I used
`bash scripting`, `ncat`

### Solution
```bash
# generate all combinations and pipe to ncat
for i in $(seq 0000 9999); do
  echo "password $i"
done | ncat localhost 30002
```

### What I learned
Simple brute force with a shell loop. The daemon accepts input line by line  piping all 10000 combinations at once is faster than connecting 10000 times. Scripted attacks are more efficient than manual ones.

---

## Level 25 → 26

### Objective
bandit26's shell is not `/bin/bash`  it's a custom binary that exits immediately.

### What I used
`/etc/passwd`, `more`, `vi`, shell escape

### What I tried first
SSHed in with the private key and got immediately disconnected. First thought was a connection problem — wrong port, wrong key, something network-related. Tried a few more times, same result. Then I actually read what was happening and noticed it wasn't a connection error, it was a clean exit. Something was terminating the session on purpose. Checked `/etc/passwd` and saw the shell wasn't bash — it was a custom binary called `showtext`. It runs `more` on a text file then exits. The terminal size trick I had to look up — didn't figure that one myself. But once I understood why it works (more enters interactive mode when output doesn't fit the screen) it made complete sense.

### Solution
```bash
cat /etc/passwd | grep bandit26
# shell is /usr/bin/showtext — runs `more` on a file then exits
# shrink your terminal window vertically so more is forced into interactive mode
ssh -i bandit26.sshkey bandit26@localhost -p 2220
# inside more: press v to open vi
# inside vi:
:set shell=/bin/bash
:shell
```

### What I learned
Always check what shell a user actually has — `/etc/passwd` is the first place to look. Program behavior depends on environment: `more` acts completely differently based on terminal size, which is not obvious at all. `vi` can spawn a shell via `:shell`, making it a classic escape tool. Also learned that "connection problem" is almost never the real answer — read what's actually happening before assuming it's infrastructure.

---

## Level 26 → 27

### Objective
Escape from bandit26's restricted environment and read bandit27's password.

### What I used
Setuid binary (same pattern as level 19)

### Solution
```bash
# already inside as bandit26 from previous level's vi escape
./bandit27-do cat /etc/bandit_pass/bandit27
```

### What I learned
Once inside, it's the same setuid pattern from level 19. Recognizing repeated patterns across levels speeds everything up.

---

## Level 27 → 28

### Objective
Clone a git repo and find the password.

### What I used
`git clone`

### Solution
```bash
mkdir /tmp/git27 && cd /tmp/git27
git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo
cd repo
cat README
```

### What I learned
Git repos can be hosted over SSH. You can clone them the same way you SSH in. Password was sitting in plain text in the README  real repos sometimes have credentials committed by mistake.

---

## Level 28 → 29

### Objective
Password was in the repo but got "fixed"  it's still in git history.

### What I used
`git log`, `git show`

### Solution
```bash
git clone ssh://bandit28-git@localhost:2220/home/bandit28-git/repo
cd repo
git log
git show HEAD~1
```

### What I learned
Git history is permanent. Deleting sensitive data from a file and committing doesn't erase it  it's still in every previous commit. This is a real and common credential leak vector.

---

## Level 29 → 30

### Objective
Password is not in main  it's in another branch.

### What I used
`git branch`, `git checkout`

### Solution
```bash
git clone ssh://bandit29-git@localhost:2220/home/bandit29-git/repo
cd repo
git branch -a
git checkout dev
cat README.md
```

### What I learned
`git branch -a` shows all branches including remote ones. Sensitive data is sometimes committed to non-main branches and forgotten. Always enumerate branches.

---

## Level 30 → 31

### Objective
Password is hidden in a git tag.

### What I used
`git tag`, `git show`

### Solution
```bash
git clone ssh://bandit30-git@localhost:2220/home/bandit30-git/repo
cd repo
git tag
git show secret
```

### What I learned
Git tags are another place to hide (or leak) data. `git tag` lists them, `git show <tag>` reveals the content. Always check tags during repo forensics.

---

## Level 31 → 32

### Objective
Push a specific file to the remote repo to get the password back.

### What I used
`git add -f`, `git commit`, `git push`

### Solution
```bash
git clone ssh://bandit31-git@localhost:2220/home/bandit31-git/repo
cd repo
cat README.md   # tells you: push file key.txt with content "May I come in?"
echo "May I come in?" > key.txt
git add -f key.txt   # -f because .gitignore blocks *.txt
git commit -m "push"
git push origin master
# server responds with the password
```

### What I learned
`.gitignore` blocks files from being staged — `-f` force-adds them anyway. The server-side hook reads the pushed content and responds. Git hooks are a real attack/defense surface.

---

## Level 32 → 33

### Objective
Every command you type gets converted to uppercase  nothing works.

### What I used
`$0` special shell variable

### Solution
```bash
$0
# drops into /bin/sh
cat /etc/bandit_pass/bandit33
```

### What I learned
`$0` holds the name of the current shell/script. It's not a command  it's a variable reference, so the uppercase filter doesn't affect it. Invoking it spawns a new shell. Special shell variables are powerful and often overlooked.

---

## Level 33 → END

### Objective
Read the final congratulations message.

### Solution
```bash
cat /etc/bandit_pass/bandit33
cat README.txt
```

### What I learned
Bandit complete. The wargame covers the full foundation: file navigation, permissions, encoding, networking, privilege escalation, shell escaping, and git forensics. Everything here maps to real-world security concepts.
