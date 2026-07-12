# Bash Scripting vs Ansible — Complete Notes

## 1. Core Idea (One Line Mein)

| | Bash Script | Ansible |
|---|---|---|
| Kya hai | Commands ki ek file, jo terminal commands hi hain | Automation tool jo "desired state" define karta hai |
| Approach | **Imperative** (HOW to do — step by step) | **Declarative** (WHAT to achieve — end result) |
| Best for | 1 machine, chhote/quick tasks | Multiple servers, enterprise scale |
| Idempotent? | Nahi (khud manage karna padta hai) | Haan (by default) |

---

## 2. Bash Scripting — Practical Example

**Scenario:** Ek naya Linux server setup karna hai — user banana, Nginx install karna, firewall open karna.

```bash
#!/bin/bash
# server_setup.sh

# 1. User banana
if id "devops" &>/dev/null; then
    echo "User already exists, skipping..."
else
    useradd -m devops
    echo "User created"
fi

# 2. Package install
if ! rpm -q nginx &>/dev/null; then
    yum install -y nginx
    systemctl enable --now nginx
fi

# 3. Firewall rule
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload
```

Run karne ka tareeka:
```bash
chmod +x server_setup.sh
./server_setup.sh
```

### Real-Time Problem
Agar aapki company ke paas **4 servers** hain — `web1, web2, web3, web4` — toh:
```bash
scp server_setup.sh user@web1:/tmp/
ssh user@web1 "chmod +x /tmp/server_setup.sh && /tmp/server_setup.sh"

scp server_setup.sh user@web2:/tmp/
ssh user@web2 "chmod +x /tmp/server_setup.sh && /tmp/server_setup.sh"
```
...aur isi tarah web3, web4 ke liye bhi manually karna padega.

**Pain points:**
- Har server ke liye manual `scp` + `ssh` — time-consuming aur error-prone.
- Script dobara run hui toh duplicate actions ka risk (isliye upar `if id`, `rpm -q` jaise checks likhne pade — yeh khud handle karna padta hai).
- Koi centralized logging/reporting nahi — kaunsa server pass/fail hua, khud track karna padega.
- 50 servers ho jayein toh yeh approach practically break ho jaati hai.

---

## 3. Ansible — Same Task, Ansible Way

**Setup ek baar (Passwordless SSH):**
```bash
ssh-keygen -t rsa
ssh-copy-id user@web1
ssh-copy-id user@web2
ssh-copy-id user@web3
ssh-copy-id user@web4
```

**Inventory file** (`inventory.ini`) — batao kaunse servers manage karne hain:
```ini
[webservers]
web1 ansible_host=192.168.1.11
web2 ansible_host=192.168.1.12
web3 ansible_host=192.168.1.13
web4 ansible_host=192.168.1.14
```

**Playbook** (`setup.yml`) — bas end result batao:
```yaml
---
- name: Setup web servers
  hosts: webservers
  become: yes

  tasks:
    - name: Create devops user
      user:
        name: devops
        state: present

    - name: Install and start Nginx
      yum:
        name: nginx
        state: present

    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Open firewall port 80
      firewalld:
        port: 80/tcp
        permanent: yes
        state: enabled
        immediate: yes
```

**Run karo — ek command, saare 4 servers ek saath:**
```bash
ansible-playbook -i inventory.ini setup.yml
```

### Kya faayda hua?
- Ek hi command se `web1` se `web4` tak parallel execution.
- Agar dobara run karo, Ansible khud check karega — user already exists ho ya nginx already installed ho, woh dubara nahi karega (koi if-else likhne ki zaroorat nahi). Isko **idempotency** kehte hain.
- Har task ka output milta hai: `ok` (already correct), `changed` (kuch change hua), `failed` (error) — isse pata chal jaata hai kis server par kya hua.

---

## 4. Real-Time Scenario Comparison

### Scenario A: Ek Developer Apni Laptop Setup Kar Raha Hai
- **Use Bash.** Ek machine, one-time task, Ansible install karna overkill hoga.

### Scenario B: Company Mein 100 Servers Par Security Patch Deploy Karna Hai
- **Use Ansible.** Ek playbook, ek command, saare 100 servers par ek saath, aur report bhi milta hai kaunsa server successfully patch hua.

### Scenario C: CI/CD Pipeline Mein Quick Log Cleanup Script
- **Use Bash.** Chhota, single-purpose task — jaise `find /var/log -mtime +7 -delete`. Ansible ka overhead yahan zaroorat nahi.

### Scenario D: New Employee Onboarding — 20 Machines Par Same User Account + SSH Key + VPN Config Chahiye
- **Use Ansible.** Ek playbook likh do, sabhi 20 machines par ek command se apply ho jayega, aur agle naye employee ke liye bhi reuse ho jayega.

---

## 5. Quick Summary Table

| Feature | Bash Scripting | Ansible |
|---|---|---|
| Best suited for | Single machine, small tasks | Many servers, large-scale tasks |
| Thinking style | Imperative (poora procedure batao) | Declarative (bas result batao) |
| Reaching servers | Manual scp + ssh, ek-ek karke | Ek central workstation se sab manage |
| Re-run safety | Khud if/for likhna padta hai | By default idempotent |
| Setup requirement | Kuch nahi, bash already hota hai | ssh-keygen, ssh-copy-id, sudo access |
| Learning curve | Kam (agar shell commands aati hain) | Thoda zyada (YAML + modules seekhne padte hain) |
| Reporting/Visibility | Khud likhna padta hai | Built-in (ok/changed/failed per host) |

---

## 6. Ek Line Mein Yaad Rakhne Ke Liye
> **Bash = "Mujhe khud jaake, khud karke aana hai."**
> **Ansible = "Mujhe bas batana hai kya chahiye, baaki system khud sambhal lega — chahe 1 server ho ya 1000."**
