
# All_Nodes

sudo dnf update -y
sudo dnf install epel-release -y
sudo systemctl status sshd

# Head Node

*Setup port forwarding*

**On Host Machine**
ssh-keygen -t ed25519 (If doesnt exit)

Windows: type %USERPROFILE%\.ssh\id_ed25519.pub | ssh kei@127.0.0.1
-p 2222 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
Linux: ssh-copy-id -p 2222 username@vm_ip_address

sudo hostnamectl set-hostname headnode

*screenshot output of hostname and cat /etc/os-release*

**On Compute 1 and 2**
sudo hostnamectl set-hostname compute1
sudo hostnamectl set-hostname compute2

*screenshot output of hostname and cat /etc/os-release*

***

# Setting up static ips

`ip a`

**On Head**
 ```Bash
    sudo nmcli con mod "Wired connection 1" connection.interface-name enp0s8
    sudo nmcli con mod "Wired connection 1" ipv4.addresses 10.0.0.1/24
    sudo nmcli con mod "Wired connection 1" ipv4.method manual
    sudo nmcli con up "Wired connection 1"
 ```

*Or, if adapter 1 set to interla*

 **On Head**
 ```Bash
    sudo nmcli con mod enp0s3 connection.interface-name enp0s3
    sudo nmcli con mod enp0s3 ipv4.addresses 10.0.0.1/24
    sudo nmcli con mod enp0s3 ipv4.method manual
    sudo nmcli con up enp0s3
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

# Set hpcuser up

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

# SSH and Key-setup

**On headnode, as hpcuser:**
```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "hpcuser@headnode"
# Press enter to accept defaults (no passphrase for cluster use)
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
Should return the hostname and uptime without asking for a password.

**What to record for deliverables:**
- Key type used (`ed25519` — note this in your SSH strategy doc)
- That keys were distributed via `ssh-copy-id`
- Save the successful SSH test output for `results.md`

---

# Configure proxyJump

**On your laptop, edit `~/.ssh/config`:**
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

**First, copy your laptop's public key to headnode:**
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub hpcuser@192.168.56.10
```

**Test ProxyJump:**
```bash
ssh compute1    
ssh compute2
```

**What to record for deliverables:**
- The full `~/.ssh/config` block (this IS your SSH strategy documentation)
- Key management note: private key stays on laptop only, public key distributed to nodes

---
# Enable passwordless proxyJump
`tail -1 ~/.ssh/authorized_keys > /tmp/laptop_key.pub`
```Bash
cat /tmp/laptop_key.pub | ssh hpcuser@compute1 'cat >> ~/.ssh/authorized_keys'
cat /tmp/laptop_key.pub | ssh hpcuser@compute2 'cat >> ~/.ssh/authorized_keys'
```

# Basline hardening
**On all nodes**
`sudo nano /etc/ssh/sshd_config`

```Bash
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPcasswords no
IgnoreRhosts yes 
```

---

sh-copy-id -i ~/.ssh/id_ed25519.pub -p 2225 hpcuser@127.0.0.1

---

# errors:

** enp0s8 does not have ipv4 so `sudo nmcli con mod enp0s8 ipv4.addresses 10.0.0.1/24` ** -> This lead us to a different solution and caused us to learn the following:
 -This taught us about the differences between connections and interfaces.
 -Difference between NIC and an interface?
 -Can virtual machines have multiple NICs?
 -Is enp0s8 the name of the NIC?
 -Why is the connection type on a virtual network ethernet?
 -What does enp0s3/enp0s8 actionally mean? (en = ethernet)
 -Why did the headnode face this issue and not the compute nodes?
 -DIfference between /etc/hosts and /etc/hostname
 ** solution **
 ```Bash
    sudo nmcli con mod "Wired connection 1" connection.interface-name enp0s8
    sudo nmcli con mod "Wired connection 1" ipv4.addresses 10.0.0.1/24
    sudo nmcli con mod "Wired connection 1" ipv4.method manual
    sudo nmcli con up "Wired connection 1"
 ```

---
 
 ** 100% packet loss upon ping headNode -c 3` **

 ** solution **
  Ensure all internal network adaptesr are set to the same name and all machines are on :[

---
  
  ** Proxyjump still asks for password**

**Solution**
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
    # No IdentityFile here – rely on forwarded agent

Host compute2
    HostName 10.0.0.3
    User hpcuser
    ProxyJump headnode
```

In headnode
`tail -1 ~/.ssh/authorized_keys > /tmp/laptop_key.pub`
```Bash
cat /tmp/laptop_key.pub | ssh hpcuser@compute1 'cat >> ~/.ssh/authorized_keys'
cat /tmp/laptop_key.pub | ssh hpcuser@compute2 'cat >> ~/.ssh/authorized_keys'
```

why wouldnt we want the personal user to be doing the sshing, since it may need to edit configs, install things and run sudo in the computes. why set it up for hpcuser

---

** systemctl --failed #Errors**
[kei@headnode ~]$ systemctl --failed
  UNIT           LOAD   ACTIVE SUB    DESCRIPTION                           
● mcelog.service loaded failed failed Machine Check Exception Logging Daemon

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
1 loaded units listed.
**Solution**
sudo systemctl stop mcelog
sudo systemctl disable mcelog
sudo systemctl mask mcelog
sudo systemctl reset-failed mcelog

[kei@compute2 ~]$ systemctl --failed
  UNIT                  LOAD   ACTIVE SUB    DESCRIPTION  
● dnf-makecache.service loaded failed failed dnf makecache

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
1 loaded units listed.
**Solution**
sudo systemctl reset-failed dnf-makecache
sudo systemctl disable --now dnf-makecache.timer
sudo systemctl mask dnf-makecache.service

--- 

