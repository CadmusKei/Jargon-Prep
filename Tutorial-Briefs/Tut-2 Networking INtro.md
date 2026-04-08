
# Tutorial: Networking Foundation & Cluster Access

## Part 1: Visualising the Cluster (GNS3)

Before touching the command line, it is crucial to understand what a subnet looks like physically. In this exercise, you will build a miniature virtual cluster and test how subnet masks dictate traffic.

**Tasks:**

1.  Open GNS3 and drag **one virtual switch** and **three Virtual PCs (VPCs)** onto your workspace. Connect all three VPCs to the switch.
    
2.  We will treat VPC-1 as your Head Node and VPC-2 and VPC-3 as Compute Nodes.
    
3.  Using the `10.0.0.0/24` subnet discussed in the lecture, assign the following IP addresses in the VPC consoles:
    
    -   **VPC-1 (Head Node):** `10.0.0.1/24`
        
    -   **VPC-2 (Compute 1):** `10.0.0.2/24`
        
    -   **VPC-3 (Compute 2 - Rogue Node):** Deliberately configure this node on the wrong subnet by assigning it `10.0.1.2/24`.
        

**Validation:**

-   From VPC-1, run `ping 10.0.0.2`. (This should succeed and measure the round-trip time) .
    
-   From VPC-1, run `ping 10.0.1.2`. (This should fail).
    
-   _Take a screenshot of the failed ping. This illustrates exactly why calculating your CIDR ranges correctly is critical for cluster communication._


## Part 2: Cluster Muscle Memory (Virtual Machines)

Now we apply these concepts to actual Linux environments. You will configure persistent static IPs on two virtual machines and establish the secure, passwordless SSH access required for seamless cluster management.

**Prerequisites:**

-   Two Linux VMs running in VirtualBox/VMware.
    
-   **CRITICAL:** Ensure both VMs have an adapter set to **"Internal Network"** in your hypervisor network settings so they can communicate.
    
-   We will refer to these as `head-node` and `compute-node`.
    

### Step 1: Verify the Physical Layer

Before configuring anything, ensure your interfaces are recognized by the OS and identify their names. Run the following on both VMs:
```
# The -br (brief) flag makes the output much easier to read.
# Look for your internal network adapter (e.g., eth1, enp0s8)
ip -br link

```

### Step 2: Assign Static IPs

Using `nmcli` (NetworkManager Command Line Interface), set persistent static IPs so your nodes always know where to find each other. _Note: Replace `eth1` with the actual interface name you found in Step 1._

**On `head-node`:**
```
sudo nmcli dev modify eth1 ipv4.addresses 10.0.0.1/24 ipv4.method manual
sudo nmcli dev connect eth1

```

**On `compute-node`:**
```
sudo nmcli dev modify eth1 ipv4.addresses 10.0.0.2/24 ipv4.method manual
sudo nmcli dev connect eth1

```

**Validation:** From your `head-node`, verify Layer 3 reachability:
```
ping -c 4 10.0.0.2

```

### Step 3: Configure Passwordless SSH

In a distributed computing environment, your head node needs to execute commands on compute nodes instantly, without prompting for a password.

1.  **Generate the Key Pair (On `head-node` only):**
    ```
    # Press Enter to accept the default file location and leave the passphrase empty
    ssh-keygen -t ed25519
    
    ``` 
    Note: This creates an elliptic curve key pair, which is highly secure and efficient.
    
2.  **Distribute the Public Key:** Copy your new key to the compute node. You will be prompted for the compute node's password one last time.
    ```
    ssh-copy-id user@10.0.0.2
    
    ```
3.  **Verify Seamless Access:**
    ```
    # This should log you directly into the compute node without a password prompt
    ssh user@10.0.0.2
    
    ```
----------

## Part 3: Submission & Troubleshooting

**Submission:** Once you have successfully logged into your compute node via SSH without a password, run `ip -br addr` to show the compute node's IP address alongside your active SSH session. Upload this screenshot to your team's Discord channel.

**Common Pitfalls & Troubleshooting:** If things are not working as expected, systematically work through these layers:

-   **Pings are failing:**
    
    -   Check the physical layer first! Ensure both VMs are set to the same internal network name in your hypervisor settings.
        
    -   Verify your subnet masks. Did you accidentally type `/8` instead of `/24`?
        
    -   Run `ip -br addr` on both machines to ensure the IPs actually applied properly.
        
-   **SSH still asks for a password:**
    
    -   Ensure you ran `ssh-copy-id` from the exact user account you are trying to log in with. Keys are user-specific, not system-wide.
        
    -   Check the permissions on the compute node's `~/.ssh` directory (should be `700`) and the `authorized_keys` file (should be `600`). Strict permissions are required for SSH to trust the keys.
