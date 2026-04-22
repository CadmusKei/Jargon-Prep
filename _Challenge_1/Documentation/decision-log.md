# Decision Log — Challenge 1

| Decision | Options | Choice | Reason |
|---|---|---|---|
| Hypervisor | VirtualBox, VMWare | VirtualBox | Ease, based off team's past experiences |
| OS | Rocky Linux 9, Ubuntu | Rocky Linux 9 | Common in HPC environments|
| SSH key type | RSA 4096, ed25519 | ed25519 | More secure and modern default |
| IP assignment | DHCP, static | Static | IPs must not change on reboot (stability) |
| External access | Direct IP per node, ProxyJump | ProxyJump | Compute nodes unexposed; single entry point is more secure |
| Firewall | nftables, firewalld, ufw | firewalld | Default, sufficient for baseline hardening |

