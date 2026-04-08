## Tutorial 4: Part B
You'll notice that your vm2 won't have access to the network the network. In an actual HPC system, you'll only have the head node connected directly to the internet with the compute nodes routing all traffic through it. This allows the head node to control traffic from each compute node, i.e. firewall control, instead of having to setup on each individual compute node.

In order for us to route our internet traffic through our head node, we will have to preform NAT (or IP Masquerading in this case) for our internal network IPs. To manage our firewall for this, We will be using [nftables](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page) which is a stateful firewall.

From here on, I will refer to vm1 as our head node and vm2 as our compute node.

### Tasks

- Head node:
    1. Install the `nftables` package.
    2. Flush any existing rulesets: `sudo nft flush ruleset`
    3. With `nft`, add an "inet" table called `my_nat`.
    4. Add a chain called `my_masquerade` to our `my_nat` table where:
        1. type is "nat",
        2. hook is "postrouting" and
        3. priority is "srcnat".
    5. Add a rule to our `my_masquerade` chain that when the output interface equals our public facing interface (the interface connected to the internet), masquerade.
    6. Confirm by running `sudo nft list ruleset`
    7. Restart and enable the `nftables` service.
- Compute node:
    1. Ssh into your compute node via your head node using "Proxy Jump".
    2. Add a default route via your head node IP.
    3. If all is successful, you should be able to `ping google.com` successfully.

### Deliverables

Screenshots of:
1. head node - the output of `sudo nft list ruleset` and `sudo systemctl status nftables`
2. compute node - the output of `ip route` and `ping google.com -c 4`
