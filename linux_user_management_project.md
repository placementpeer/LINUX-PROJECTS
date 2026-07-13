# Linux User & Group Management - Practice Project (Beginner Friendly)

Ye project aap apne Linux machine (VM, VirtualBox, ya cloud server) pe practice kar sakte hain.
Har topic ke saath: **basic terms**, **commands table**, aur **practice steps** diye gaye hain.

> ⚠️ Zyada tar commands `sudo` ya **root** user ke through chalti hain, kyunki user/group create karna
> ek "admin level" (system-level) kaam hai. Normal user ye kaam nahi kar sakta.

---

## 0. Pehle Kuch Basic Terms Samajh Lo

| Term | Simple Matlab |
|---|---|
| **User** | Ek account jisse koi insaan ya program Linux system login/use karta hai |
| **Root user** | Sabse powerful user, jiske paas system ke saare rights hote hain (Windows ke "Administrator" jaisa) |
| **Group** | Users ka ek collection, taaki permissions ek saath multiple logon ko di ja sake |
| **UID** | User ID - har user ko milne wala unique number |
| **GID** | Group ID - har group ko milne wala unique number |
| **Home directory** | User ka apna personal folder, jaise `/home/ravi` |
| **Shell** | Wo program jo commands leta hai aur run karta hai, jaise `/bin/bash` |
| **sudo** | Normal user ko temporarily root jaisi power dene wala command |
| **Terminal** | Wo window jaha hum commands type karte hain |

---

## 1. User Creation (Naya User Banana)

**Basic Terms:**
- `useradd` = user create karne ka low-level command
- `adduser` = same kaam karta hai but zyada friendly/interactive (Debian/Ubuntu me)
- `passwd` = user ka password set/change karna

| Command | Explanation (Hinglish) |
|---|---|
| `sudo useradd ravi` | "ravi" naam ka naya user banata hai (bina password, bina home folder by default kuch systems me) |
| `sudo useradd -m ravi` | `-m` matlab home directory bhi automatically bana do (`/home/ravi`) |
| `sudo useradd -m -s /bin/bash ravi` | `-s` se shell set karte hain, yaha bash shell diya |
| `sudo passwd ravi` | "ravi" user ke liye password set karta hai |
| `sudo adduser ravi` | Interactive tareeke se user banata hai, khud hi puchta hai password, full name, etc. |
| `id ravi` | "ravi" user ka UID, GID aur groups dikhata hai |
| `cat /etc/passwd` | System ke saare users ki list dikhata hai |
| `sudo userdel ravi` | "ravi" user ko delete karta hai (home folder delete nahi hota) |
| `sudo userdel -r ravi` | User delete karta hai AUR uska home folder bhi delete karta hai |
| `sudo usermod -l newname oldname` | User ka login name change karta hai |

### Practice Steps:
1. `sudo useradd -m -s /bin/bash testuser1`
2. `sudo passwd testuser1` → koi password set karo
3. `id testuser1` chala kar check karo user bana ya nahi
4. `cat /etc/passwd | grep testuser1` se entry dekho

---

## 2. Password Policy (Password Ke Rules)

**Basic Terms:**
- Password policy matlab: password kitna lamba ho, kab expire ho, kitni baar galat try allowed ho — ye sab rules
- Ye rules mostly `/etc/login.defs` file aur `PAM` (niche explain kiya hai) se control hote hain

| Command / File | Explanation (Hinglish) |
|---|---|
| `sudo nano /etc/login.defs` | Ye file open karo, isme default password rules hote hain (jaise PASS_MAX_DAYS) |
| `PASS_MAX_DAYS 90` | Password max 90 din tak valid rahega, fir change karna padega |
| `PASS_MIN_DAYS 0` | Password minimum kitne din baad dobara change kar sakte ho |
| `PASS_MIN_LEN 8` | Password ki minimum length (kuch systems me ye PAM se control hota hai) |
| `PASS_WARN_AGE 7` | Expiry se 7 din pehle warning dikhana shuru karega |
| `sudo chage -l ravi` | "ravi" user ki current password policy/status dikhata hai |
| `sudo chage -M 30 ravi` | "ravi" ka password max 30 din me expire hoga |
| `sudo chage -m 1 ravi` | Password minimum 1 din baad hi change ho sakta hai |
| `sudo chage -W 5 ravi` | Expiry se 5 din pehle warning milegi |

### Practice Steps:
1. `sudo chage -l testuser1` chala kar current policy dekho
2. `sudo chage -M 30 testuser1` se max 30 days set karo
3. Dobara `sudo chage -l testuser1` chala kar confirm karo

---

## 3. Group Management (Groups Banana Aur Manage Karna)

**Basic Terms:**
- **Primary group** = user ka default/main group (usually user ke naam jaisa hi group)
- **Secondary group** = extra groups jinme user add hota hai extra permissions ke liye

| Command | Explanation (Hinglish) |
|---|---|
| `sudo groupadd developers` | "developers" naam ka naya group banata hai |
| `groups ravi` | "ravi" konse groups me hai, ye dikhata hai |
| `cat /etc/group` | System ke saare groups ki list dikhata hai |
| `sudo usermod -aG developers ravi` | "ravi" ko "developers" group me add karta hai (`-a` = append, purane groups delete nahi honge) |
| `sudo gpasswd -d ravi developers` | "ravi" ko "developers" group se remove karta hai |
| `sudo groupdel developers` | "developers" group ko delete karta hai |
| `sudo usermod -g developers ravi` | "ravi" ka PRIMARY group change karke "developers" karta hai |

> ⚠️ Important: `usermod -aG` me hamesha `-a` (append) use karo. Agar `-a` bhool gaye,
> to user ke purane saare secondary groups remove ho jayenge!

### Practice Steps:
1. `sudo groupadd cloudteam`
2. `sudo usermod -aG cloudteam testuser1`
3. `groups testuser1` se confirm karo ki group add hua

---

## 4. Sudo (Temporary Root Power Dena)

**Basic Terms:**
- `sudo` = "SuperUser DO" — normal user ko permission deta hai root jaisa command chalane ki
- `/etc/sudoers` = ye file decide karti hai kaun kaun sudo use kar sakta hai
- `visudo` = `/etc/sudoers` file ko SAFELY edit karne ka command (isse syntax error catch ho jata hai)

| Command | Explanation (Hinglish) |
|---|---|
| `sudo whoami` | Agar output "root" aaye, matlab sudo kaam kar raha hai |
| `sudo visudo` | `/etc/sudoers` file ko safely edit karne ke liye kholta hai |
| `ravi ALL=(ALL:ALL) ALL` | Ye line `/etc/sudoers` me dalne se "ravi" ko full sudo access milta hai |
| `sudo -l` | Current user ke paas kaunse sudo permissions hain, dikhata hai |
| `sudo -u username command` | Kisi specific user ke naam se command run karta hai |

### Practice Steps:
1. `sudo visudo` open karo (dhyan se, galat edit se system lock ho sakta hai)
2. Check karo ki `%wheel` ya `%sudo` group ki entry already hai
3. `sudo -l` chala kar apni permissions dekho

---

## 5. Wheel Group (Special Admin Group)

**Basic Terms:**
- **Wheel group** ek special group hai (mostly RedHat/CentOS/Fedora me use hota hai) —
  jo bhi user isme hota hai, use sudo/root access mil jata hai
- Ubuntu/Debian me isi kaam ke liye **"sudo" group** use hota hai

| Command | Explanation (Hinglish) |
|---|---|
| `grep wheel /etc/group` | Check karta hai wheel group exist karta hai ya nahi |
| `sudo usermod -aG wheel ravi` | "ravi" ko wheel group me add karta hai (RedHat/CentOS systems) |
| `sudo usermod -aG sudo ravi` | "ravi" ko sudo group me add karta hai (Ubuntu/Debian systems) |
| `%wheel ALL=(ALL) ALL` | Ye line `/etc/sudoers` me hoti hai jo wheel group ko sudo power deti hai |

### Practice Steps:
1. Apna OS check karo: Ubuntu hai to `sudo` group use hoga, CentOS/Fedora hai to `wheel` group
2. `grep -E 'wheel|sudo' /etc/group` chala kar dekho konsa group hai
3. `sudo usermod -aG sudo testuser1` (Ubuntu) ya `wheel` (CentOS) chalao

---

## 6. Expiry (User Account Ki Expiry Date)

**Basic Terms:**
- Account expiry matlab: ek fix date ke baad us account se login hi nahi ho payega
- Ye password expiry se alag hai — password expiry sirf password change karwati hai, account expiry pura login band kar deti hai

| Command | Explanation (Hinglish) |
|---|---|
| `sudo chage -E 2026-12-31 ravi` | "ravi" ka account 31 Dec 2026 ko expire ho jayega |
| `sudo chage -E -1 ravi` | Expiry hata deta hai (account kabhi expire nahi hoga) |
| `sudo chage -l ravi` | Account expiry date sameet saari info dikhata hai |
| `sudo usermod -e 2026-12-31 ravi` | Yehi kaam `usermod` se bhi kar sakte ho |
| `sudo passwd -l ravi` | Account ko turant LOCK kar deta hai (login band) |
| `sudo passwd -u ravi` | Locked account ko wapas UNLOCK karta hai |

### Practice Steps:
1. `sudo chage -E 2026-08-01 testuser1` se ek expiry date set karo
2. `sudo chage -l testuser1` se confirm karo
3. `sudo chage -E -1 testuser1` se expiry hata do

---

## 7. PAM (Pluggable Authentication Modules)

**Basic Terms:**
- PAM ek system hai jo decide karta hai **login/authentication kaise hoga** — password check, fingerprint, retry limits, sab PAM handle karta hai
- Iski config files `/etc/pam.d/` folder me hoti hain
- Har service (login, sudo, ssh) ki apni alag PAM config file hoti hai

| Command / File | Explanation (Hinglish) |
|---|---|
| `ls /etc/pam.d/` | Saari PAM config files ki list dikhata hai (login, sudo, sshd, etc.) |
| `cat /etc/pam.d/common-password` | Password related PAM rules dikhata hai (Debian/Ubuntu) |
| `pam_pwquality.so` | Ek PAM module jo password strength (length, complexity) check karta hai |
| `pam_unix.so` | Basic Linux password authentication karne wala module |
| `sudo nano /etc/pam.d/common-password` | Password quality rules edit karne ke liye file open karna |
| `retry=3` | PAM config me ye line batati hai ki galat password 3 baar try kar sakte ho |
| `minlen=8` | PAM config me minimum password length 8 set karta hai |

> ⚠️ PAM files edit karte time bahut careful raho — galat edit se pura login system
> (even root ka bhi) block ho sakta hai. Hamesha backup lo:
> `sudo cp /etc/pam.d/common-password /etc/pam.d/common-password.bak`

### Practice Steps:
1. `ls /etc/pam.d/` chala kar files dekho
2. `cat /etc/pam.d/common-password` (ya `system-auth` CentOS me) chala kar padho
3. Sirf padhne ke liye — pehli baar edit mat karo jab tak confident na ho

---

## 8. UMASK (Default File/Folder Permissions)

**Basic Terms:**
- Jab bhi koi naya file ya folder banta hai, usko kuch default permissions milti hain
- **umask** decide karta hai ki default permission se kitni permission "hata" di jaye
- File ki max default permission: `666` (rw-rw-rw-), Folder ki max default: `777` (rwxrwxrwx)
- Formula: **Final Permission = Default Permission − Umask**

| Command | Explanation (Hinglish) |
|---|---|
| `umask` | Current umask value dikhata hai (jaise `0022`) |
| `umask 0027` | Naya umask set karta hai (sirf current session ke liye) |
| `touch file1.txt && ls -l file1.txt` | Naya file banao aur uski permission dekho (umask ka effect check karne ke liye) |
| `mkdir folder1 && ls -ld folder1` | Naya folder banao aur uski permission dekho |
| `sudo nano /etc/profile` | Yaha umask set karo taaki ye SABHI users ke liye permanent ho |
| `umask -S` | Umask ko symbolic form me dikhata hai (jaise `u=rwx,g=rx,o=rx`) |

### Example (samajhne ke liye):
```
File default        = 666 (rw-rw-rw-)
Umask                = 022
Final Permission     = 666 - 022 = 644 (rw-r--r--)

Folder default       = 777 (rwxrwxrwx)
Umask                = 022
Final Permission     = 777 - 022 = 755 (rwxr-xr-x)
```

### Practice Steps:
1. `umask` chala kar current value dekho (usually `0022`)
2. `touch testfile.txt` → `ls -l testfile.txt` chala kar permission dekho
3. `umask 077` set karo
4. `touch testfile2.txt` → `ls -l testfile2.txt` — is baar permission zyada strict hogi
5. Difference compare karo dono files ki permission me

---

## 9. Full Practice Project (Sab Kuch Ek Saath)

Ye ek complete scenario hai jisse aap sab topics ek saath practice kar sakte ho.
Har step ek pehle wale par based hai — order me hi karo.

| Step | Task | Command Hint |
|---|---|---|
| 1 | "devuser" naam ka user banao, home dir aur bash shell ke saath | `sudo useradd -m -s /bin/bash devuser` |
| 2 | Uska password set karo | `sudo passwd devuser` |
| 3 | Password max 60 din me expire ho, ye set karo | `sudo chage -M 60 devuser` |
| 4 | "devteam" naam ka group banao | `sudo groupadd devteam` |
| 5 | "devuser" ko "devteam" group me add karo | `sudo usermod -aG devteam devuser` |
| 6 | "devuser" ko sudo/wheel group me bhi add karo | `sudo usermod -aG sudo devuser` |
| 7 | Confirm karo sudo access mil gaya | `sudo -l -U devuser` |
| 8 | Account ki ek expiry date set karo (6 mahine baad) | `sudo chage -E 2027-01-13 devuser` |
| 9 | `chage -l` se poori policy verify karo | `sudo chage -l devuser` |
| 10 | Umask check karo aur ek file banake permission verify karo | `umask` then `touch test.txt && ls -l test.txt` |
| 11 | Sab kuch clean karne ke liye user delete karo | `sudo userdel -r devuser` |
| 12 | Group bhi delete karo | `sudo groupdel devteam` |

---

## 10. Quick Revision Table (Sab Commands Ek Jagah)

| Category | Command | Kaam Kya Karta Hai |
|---|---|---|
| User | `useradd -m -s /bin/bash name` | User + home dir + shell banata hai |
| User | `passwd name` | Password set/change karta hai |
| User | `userdel -r name` | User + home dir delete karta hai |
| Policy | `chage -l name` | Password policy dikhata hai |
| Policy | `chage -M days name` | Max password age set karta hai |
| Group | `groupadd name` | Naya group banata hai |
| Group | `usermod -aG group name` | User ko group me add karta hai |
| Group | `groups name` | User ke groups dikhata hai |
| Sudo | `visudo` | Sudoers file safely edit karta hai |
| Sudo | `sudo -l` | Apni sudo permissions dikhata hai |
| Wheel | `usermod -aG wheel/sudo name` | Admin group me add karta hai |
| Expiry | `chage -E date name` | Account expiry date set karta hai |
| Expiry | `passwd -l name` | Account lock karta hai |
| PAM | `ls /etc/pam.d/` | PAM config files dikhata hai |
| Umask | `umask` | Current default permission mask dikhata hai |

---

### 💡 Tip:
Sab commands ek **test/practice VM** pe try karo (VirtualBox, VMware, ya cloud free-tier),
apne main/production system pe nahi — kyunki galat user/sudo/PAM config se login problems ho sakti hain.

Happy Practicing! 🐧
