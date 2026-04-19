# Results — Challenge 1

## Node Reachability
| Source | Target | Method | Result |
|---|---|---|---|
| headnode | compute1 | ping | Success |
| headnode | compute2 | ping | Success |
| laptop | headnode | SSH direct | Success |
| laptop | compute1 | SSH ProxyJump | Success |
| laptop | compute2 | SSH ProxyJump | Success |

## Node Inventory
| Hostname | Role | IP | OS | Uptime |
|---|---|---|---|---|
| headnode | Head/Login Node | 10.0.0.1 | Rocky Linux 9.7 | 44 minutes |
| compute1 | Compute Node | 10.0.0.2 | Rocky Linux 9.7 | 2 minutes |
| compute2 | Compute Node | 10.0.0.3 | Rocky Linux 9.7 | 2 minutes|

## Essential Service Checks
| Node | sshd | firewall | Failed Units |
|---|---|---|---|
| headnode | Active | Active | None |
| compute1 | Active | Active | None |
| compute2 | Active | Active | None |
