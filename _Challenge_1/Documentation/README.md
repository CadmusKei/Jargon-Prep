# Jargon 

## Environment
- Hypervisor: VirtualBox
- OS: Rocky Linux 9.7
- Nodes: 1 head node, 2 compute nodes

## Run Order
1. Provision 3 VMs and install Rocky
2. Run system updates and set hostnames
3. Temporarily enable NAT for compute1, compute2 for SSH during setup
3. Configure static IPs and /etc/hosts on all nodes 
4. Create hpcuser on all nodes 
5. Generate SSH keys on headnode and distribute to compute nodes 
6. Configure ProxyJump on laptop (Host)
7. Remove the NAT adapters on compute nodes. 
7. Apply baseline hardening on all nodes

## Key Commands

### All_Nodes

sudo dnf update -y
sudo dnf install epel-release -y
sudo systemctl status sshd

### Head Node

*Setup port forwarding*

**On Host Machine**
ssh-keygen -t ed25519 (If doesnt exist)

Windows: `type %USERPROFILE%\.ssh\id_ed25519.pub | ssh kei@127.0.0.1 -p 2222 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"`
Linux: `ssh-copy-id -p 2222 username@vm_ip_address`

sudo hostnamectl set-hostname headnode

*screenshot output of hostname and cat /etc/os-release*

**On Compute 1 and 2**
sudo hostnamectl set-hostname compute1
sudo hostnamectl set-hostname compute2

*screenshot output of hostname and cat /etc/os-release*

***

### Setting up static ips

`ip a`

**On Head**
 ```Bash
    sudo nmcli con mod "Wired connection 1" connection.interface-name enp0s8
    sudo nmcli con mod "Wired connection 1" ipv4.addresses 10.0.0.1/24
    sudo nmcli con mod "Wired connection 1" ipv4.method manual
    sudo nmcli con up "Wired connection 1"
 ```

**On compute**
```Bash
sudo nmcli con mod enp0s3 ipv4.addresses 10.0.0.2/24
sudo nmcli con mod enp0s3 ipv4.method manual
sudo nmcli con up enp0s3
```

**Edit /etc/hosts on ALL three nodes:**
```bash
sudo nano /etc/hosts
```
**Add these lines to the file on every node:**
```Bash
10.0.0.1 headnode
10.0.0.2 compute1
10.0.0.3 compute2
```

**Verify reachability (run from headnode):**
```bash
ping -c 3 compute1
ping -c 3 compute2
```

***

### Set hpcuser up

**Run on ALL three nodes:**
```bash
# Create group and user with matching UID/GID (use same numbers on all nodes)
sudo groupadd -g 1001 hpcgroup
sudo useradd -m -u 1001 -g 1001 -s /bin/bash hpcuser
sudo passwd hpcuser   # set a password
```

**Verify:**
```bash
id hpcuser
```
Should show `uid=1001(hpcuser) gid=1001(hpcgroup)` on all nodes.

---

### SSH and Key-setup

**On headnode, as hpcuser:**
```bash
ssh-keygen -t ed25519 -C "hpcuser@headnode"
```

**Copy public key to compute nodes:**
```bash
ssh-copy-id hpcuser@compute1
ssh-copy-id hpcuser@compute2
```

**Test passwordless SSH:**
```bash
ssh hpcuser@compute1 "hostname && uptime"
ssh hpcuser@compute2 "hostname && uptime"
```

---

### Configure proxyJump

**On host, edit `~/.ssh/config`:**
```Bash
Host headnode
    HostName 127.0.0.1
    Port 2225
    User hpcuser
    IdentityFile ~/.ssh/id_ed25519

Host compute1
    HostName 10.0.0.2
    User hpcuser
    ProxyJump headnode

Host compute2
    HostName 10.0.0.3
    User hpcuser
    ProxyJump headnode
```

**copy laptop's public key to headnode:**
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub hpcuser@192.168.56.10
```

**Test ProxyJump:**
```bash
ssh compute1    
ssh compute2
```

---

# Basline hardening

`sudo nano /etc/ssh/sshd_config`

```Bash
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
IgnoreRhosts yes 
```

```Bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --list-all
```

```Bash
systemctl --failed
```
---

