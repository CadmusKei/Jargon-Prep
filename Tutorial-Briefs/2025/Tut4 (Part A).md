# Tutorial 4 - Networking: Part A

This tutorial is set up to being able to run your benchmark across multiple machines, utilizing each machine resources together to achieve a faster benchmark. In order for that to happen, we need a way for machines to communicate with each other.

Part A is just build to getting the devices connected and communicating on a network.

### Tasks

1. Add an "Internal Network" network adapter to your Virtual Machine. (or the equivalent if you are using another virtual machine)
2. Enter and give the vm an IP address for the "Internal Network" interface that is on the 10.0.0.0/24 subnet. (most likely the last interface when you run `ip -br link`)
3. Confirm by checking the output of `ip -br addr`
4. Spin up another virtual machine but this time only having an "Internal Network" adapter. (Will call these vm1 and vm2)
5. Enter via the GUI terminal and repeat step 2 & 3.
6. If everything is working, you should be able to `ping` your other virtual machine using the IP address you gave it.

### Deliverables

Two screenshots of you being able to ping each virtual machine from each other. Screenshot needs to display the output of these commands:

1. `ip -brief addr`
2. `ping <other-vm-ip> -c 4`
