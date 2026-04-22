# Challange-1 errors:

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

[el@compute2 ~]$ systemctl --failed
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
