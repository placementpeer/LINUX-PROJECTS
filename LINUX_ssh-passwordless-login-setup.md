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
