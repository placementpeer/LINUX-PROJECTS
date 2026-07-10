# Passwordless SSH Login Setup (NodeB → NodeA)

## System Mapping

| Role   | IP Address    | Hostname               |
|--------|---------------|------------------------|
| Server | 172.25.25.12  | nodea.lab.example.com  |
| Client | 172.25.25.22  | nodeb.lab.example.com  |

## Setup Steps

| # | Explanation (Short) | Command | Expected Result |
|---|---|---|---|
| 1 | NodeB par SSH key pair generate karein | `ssh-keygen` (run on NodeB) | Prompts aayenge (Enter dabayein sabme) → `id_rsa` (private) aur `id_rsa.pub` (public) files `~/.ssh/` me ban jayengi |
| 2 | Generated public key ko NodeA par copy karein | `ssh-copy-id user@172.25.25.12` (run on NodeB) | NodeA ka password ek baar maangega → public key NodeA ke `~/.ssh/authorized_keys` me add ho jayegi |
| 2 (Alt) | Agar `ssh-copy-id` available na ho, manual copy | `cat ~/.ssh/id_rsa.pub \| ssh user@172.25.25.12 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"` | Same result — key manually append ho jayegi authorized_keys me |
| 3 | Permissions verify karein NodeA par | `ls -ld ~/.ssh` and `ls -l ~/.ssh/authorized_keys` | `.ssh` folder → `drwx------` (700), `authorized_keys` file → `-rw-------` (600) |
| 4 | SELinux context fix karein (agar RHEL/CentOS hai) | `restorecon -Rv ~/.ssh` (run on NodeA) | SSH-related SELinux labels correctly set ho jayenge, warnings/denials avoid honge |
| 5 | SSH config confirm karein NodeA par | `cat /etc/ssh/sshd_config \| grep -i pubkeyauth` | Output: `PubkeyAuthentication yes` (agar na ho to enable karke service restart karein) |
| 6 | SSH service restart karein (agar config change kiya ho) | `systemctl restart sshd` (run on NodeA) | SSH daemon naye config ke saath reload ho jayega |
| 7 | Passwordless login test karein | `ssh user@172.25.25.12` (run on NodeB) | Bina password maange directly NodeA ka shell prompt aa jayega ✅ |

## Notes

- Private key (`id_rsa`) kabhi share na karein — sirf public key (`id_rsa.pub`) NodeA par jaati hai.
- Passphrase-less key setup automation ke liye achha hai, par production servers ke liye extra security ke saath use karein (jaise `ssh-agent` ya restricted commands).

---

# Bidirectional Setup: NodeA → NodeB (Key Generation & Copy)

| # | Explanation (Short) | Command | Expected Result |
|---|---|---|---|
| 1 | NodeA par SSH key pair generate karein (agar already nahi hai) | `ssh-keygen` (run on NodeA) | `id_rsa` aur `id_rsa.pub` files `~/.ssh/` me ban jayengi on NodeA |
| 2 | NodeA ki public key ko NodeB par copy karein | `ssh-copy-id user@172.25.25.22` (run on NodeA) | NodeB ka password ek baar maangega → public key NodeB ke `~/.ssh/authorized_keys` me add ho jayegi |
| 3 | Test karein NodeA se NodeB par passwordless login | `ssh user@172.25.25.22` (run on NodeA) | Bina password maange directly NodeB ka shell prompt aa jayega ✅ |

Isse ab **bidirectional passwordless SSH** ho jayega:
- NodeB → NodeA
- NodeA → NodeB

---

# Restrict NodeB → NodeA to Require Password (Not Direct Passwordless)

## Method 1: Pubkey Authentication completely disable karein (sirf password se login)

| # | Explanation (Short) | Command | Expected Result |
|---|---|---|---|
| 1 | NodeA par sshd config edit karein | `vi /etc/ssh/sshd_config` (run on NodeA) | File open hogi editing ke liye |
| 2 | Pubkey auth disable karein (line 45 & 46) | Set: `PubkeyAuthentication no` and `PasswordAuthentication yes` | Ab key hone ke bawajood sirf password se hi login hoga |
| 3 | Confirm exact line numbers (optional) | `grep -n "PubkeyAuthentication\|PasswordAuthentication" /etc/ssh/sshd_config` | Exact line number dikhega jahan yeh parameters set hain |
| 4 | SSH service restart karein | `systemctl restart sshd` (run on NodeA) | New config apply ho jayega |
| 5 | Test karein | `ssh user@172.25.25.12` (run on NodeB) | Password prompt aayega, direct login nahi hoga ✅ |

## Method 2: Sirf specific user/IP ke liye pubkey disable karein (selective restriction)

| # | Explanation (Short) | Command | Expected Result |
|---|---|---|---|
| 1 | sshd_config me Match block add karein | Add at end of `/etc/ssh/sshd_config`:<br>`Match Host 172.25.25.22`<br>&nbsp;&nbsp;`PubkeyAuthentication no`<br>&nbsp;&nbsp;`PasswordAuthentication yes` | Sirf NodeB (172.25.25.22) se aane wale connection ke liye password mandatory hoga, baaki sab normal rahenge |
| 2 | SSH service restart karein | `systemctl restart sshd` (run on NodeA) | Config apply ho jayega |
| 3 | Test karein NodeB se | `ssh user@172.25.25.12` | Password maangega ✅ |

## Method 3: Authorized_keys se key temporarily remove/comment karein

| # | Explanation (Short) | Command | Expected Result |
|---|---|---|---|
| 1 | NodeA par authorized_keys file edit karein | `vi ~/.ssh/authorized_keys` (run on NodeA) | File open hogi |
| 2 | NodeB ki public key wali line ko comment (#) karein ya delete karein | Line ke start me `#` lagayein ya line delete karein | Key ab invalid ho jayegi |
| 3 | Test karein | `ssh user@172.25.25.12` (run on NodeB) | Ab password maangega ✅ |

## Method 4: Extra layer add karein — 2FA (Password + OTP) using Google Authenticator

| # | Explanation (Short) | Command | Expected Result |
|---|---|---|---|
| 1 | Google Authenticator PAM module install karein | `yum install google-authenticator -y` (RHEL/CentOS) or `apt install libpam-google-authenticator -y` (Ubuntu/Debian) | Package install ho jayega on NodeA |
| 2 | User ke liye authenticator setup karein | `google-authenticator` (run as target user on NodeA) | QR code aur secret key milega |
| 3 | PAM config me add karein | `/etc/pam.d/sshd` me add: `auth required pam_google_authenticator.so` | PAM ab OTP verification enforce karega |
| 4 | sshd_config me enable karein | Set: `ChallengeResponseAuthentication yes` and `AuthenticationMethods publickey,keyboard-interactive` | Login ke liye key + OTP dono chahiye honge |
| 5 | SSH restart karein | `systemctl restart sshd` | Config apply ho jayega |
| 6 | Test karein | `ssh user@172.25.25.12` (NodeB se) | Key match hone ke baad bhi OTP code maanga jayega ✅ |

## Method 5: Client-side force karein password use

| # | Explanation (Short) | Command | Expected Result |
|---|---|---|---|
| 1 | NodeB se explicitly password authentication force karein | `ssh -o PreferredAuthentications=password user@172.25.25.12` | SSH client key ko ignore karke sidha password maangega (agar server side PasswordAuthentication enabled hai) |

### Konsa method use karna chahiye?

| Requirement | Best Method |
|---|---|
| Sabke liye password mandatory karna hai | Method 1 |
| Sirf NodeB ke liye restrict karna hai, baaki normal rahe | Method 2 |
| Sirf temporarily disable karna hai (testing ke liye) | Method 3 |
| Extra security chahiye (password + OTP dono) | Method 4 |
| Sirf client side se test/force karna hai | Method 5 |

---

# Key Storage Location

| Key Type | Location | Description |
|---|---|---|
| Private Key | `~/.ssh/id_rsa` | Secret key — kabhi share nahi karni |
| Public Key | `~/.ssh/id_rsa.pub` | Shareable key — target server ke `authorized_keys` me copy hoti hai |

**Root user path:** `/root/.ssh/id_rsa` and `/root/.ssh/id_rsa.pub`
**Normal user path:** `/home/username/.ssh/id_rsa` and `/home/username/.ssh/id_rsa.pub`

Copied public key target server par yahan store hoti hai: `~/.ssh/authorized_keys`

---

# Verify Keys Are Generated (Both Nodes)

| # | Explanation (Short) | Command | Expected Result |
|---|---|---|---|
| 1 | NodeA par key files check karein | `ls -la ~/.ssh/` (run on NodeA) | `id_rsa` aur `id_rsa.pub` dono files dikhni chahiye |
| 2 | NodeB par key files check karein | `ls -la ~/.ssh/` (run on NodeB) | `id_rsa` aur `id_rsa.pub` dono files dikhni chahiye |
| 3 | Sirf public key exist karti hai ya nahi | `cat ~/.ssh/id_rsa.pub` | Output `ssh-rsa AAAA...` se start hoga |
| 4 | Sirf private key exist karti hai ya nahi | `cat ~/.ssh/id_rsa` | Output `-----BEGIN OPENSSH PRIVATE KEY-----` se start hoga |
| 5 | Fingerprint verify karein | `ssh-keygen -lf ~/.ssh/id_rsa.pub` | Fingerprint aur bit-length dikhega |
| 6 | Combined quick check (dono files ek saath) | `[ -f ~/.ssh/id_rsa ] && [ -f ~/.ssh/id_rsa.pub ] && echo "Both keys exist" || echo "Keys missing"` | `Both keys exist` print hoga agar dono files present hain |

### Authorized_keys me copy hui key check karne ke liye

| # | Explanation (Short) | Command | Expected Result |
|---|---|---|---|
| 1 | NodeA par authorized_keys check karein (NodeB ki key aayi ya nahi) | `cat ~/.ssh/authorized_keys` (run on NodeA) | NodeB ki public key ki line dikhni chahiye |
| 2 | NodeB par authorized_keys check karein (NodeA ki key aayi ya nahi) | `cat ~/.ssh/authorized_keys` (run on NodeB) | NodeA ki public key ki line dikhni chahiye |
