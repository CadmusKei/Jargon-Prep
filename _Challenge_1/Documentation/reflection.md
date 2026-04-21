# How we did it

We created a network of 3 Rocky Linux virtual machines with a headnode, and two compute nodes. 
The three nodes were part of an internal network with the static IPs 10.0.0.1, 10.0.0.2 and 10.0.0.3, all
under the hostnames headnode, compute1, and compute2 respectively. 
We then enabled a second adapter for the compute nodes for NAT. This was followed by creating the hpcuser
across all machines with the same UID. We then configured for passwordless SSH, and proxyjump. Lastly we ran 
standard baseline hardening procedures. Examples include disabling root SSH and password authentication. 

# Why we chose this approach

**NAT Adapter on compute nodes:** This was to allow temporary SSH access from the host machine while configuring 
the nodes, before removing the external exposure post-setup.
**Static IPs:** These specific IPs were chosen for simplicity. Dynamic IPs would not work for this network as 
consistent IPs are critical. 
**Global hpc user:** A shared user with a matching UID/GID across all nodes is the default for HPC. It ensures 
consistent file ownership and permissions when accessing shared filesystems, and allows job schedulers and 
other services to run tasks as the same user on any node without errors.
**Passwordless SSH:** Chosen for speed in further tasks. Also enables compatibility for automation systems later on. 
**ProxyJump:** Passwordless ProxyJump was chosen for quick access of the compute nodes from the host machine. 
This approach also ensures the compute nodes can maintain their enclosure in the internal network. 
**Baseline hardening:** Done for additional security. Disabling root SSH and password authentication reduces entry points. 


# Challenges and failures encountered

1. enp0s8 had no IPv4 configuration on the headnode
Running `sudo nmcli con mod enp0s8 ipv4.addresses 10.0.0.1/24` failed because the connection profile on the headnode was named "Wired connection 1" rather than the interface name enp0s8. The compute nodes happened to use matching connection and interface names, so the same command worked for them but not for the headnode.

2. 100% packet loss when pinging between nodes
After configuring static IPs, pings between nodes failed. This was caused by the internal network adapter the VMs not being set to the same internal network.

3. ProxyJump still prompted for a password
Even after setting up passwordless SSH from headnode to the compute nodes, connecting via ProxyJump from the laptop still asked for a password. The issue was that the laptop's public key had only been copied to the headnode. 

4. mcelog.service failed on headnode
`systemctl --failed` showed mcelog.service in a failed state. This service is for hardware machines, which is not applicable for a virtual machine. Which causes an error. 

5. dnf-makecache.service failed on compute2
`systemctl --failed` showed dnf-makecache.service failing. After NAT adapters were removed, the compute nodes lost internet access. This error was a routine check with mirrors 
and showed that it could not reach the mirrors. 


# How we recovered and what we would do differently

1. enp0s8 IPv4 issue
Instead of modifying the connection by interface name, we identified the correct connection profile name using nmcli con show and modified "Wired connection 1" directly. 
```bash
sudo nmcli con mod "Wired connection 1" connection.interface-name enp0s8
sudo nmcli con mod "Wired connection 1" ipv4.addresses 10.0.0.1/24
sudo nmcli con mod "Wired connection 1" ipv4.method manual
sudo nmcli con up "Wired connection 1"
```
What we'd do differently: Identify the correct interface names and available connections for smooth setup. 

2. 100% packet loss 
We verified all VM network adapters were on an internal network with the same network name.

What we'd do differently: Ensure these steps before attempting to assign static IPs and pinging in the internal network.

3. ProxyJump password prompt 
We extracted the laptop's public key from the headnode's authorized_keys and manually appended it to each compute node's authorized_keys file via the headnode.
```bash
tail -1 ~/.ssh/authorized_keys > /tmp/laptop_key.pub
cat /tmp/laptop_key.pub | ssh hpcuser@compute1 'cat >> ~/.ssh/authorized_keys'
cat /tmp/laptop_key.pub | ssh hpcuser@compute2 'cat >> ~/.ssh/authorized_keys'
```
What we'd do differently: Include sending the laptop's public key to all nodes as an explicit step in the run order, not just the headnode. 

4. mcelog.service failure 
We stopped, disabled and masked this so it can no longer start or fail.
```bash
sudo systemctl stop mcelog
sudo systemctl disable mcelog
sudo systemctl mask mcelog
sudo systemctl reset-failed mcelog
```
What we'd do differently: Add masking of mcelog to our regular steps for setup of VMs. 

5. dnf-makecache.service failure 
We stopped, disabled and masked this so it can no longer start or fail.
```bash
sudo systemctl reset-failed dnf-makecache
sudo systemctl disable --now dnf-makecache.timer
sudo systemctl mask dnf-makecache.service
```

What we'd do differently: Disable and mask dnf-makecache on compute nodes as part of the NAT removal step.
