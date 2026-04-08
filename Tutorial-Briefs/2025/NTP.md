# Network Time Protocol

NTP let's you to synchronise the time across all the computers in your network. This is important for HPC clusters as some applications require that system time be accurate between different nodes (imagine receiving a message 'before' it was sent). You will configure the NTP service through `chronyd` on your head node and then connect your compute nodes as its clients.

1. Install `chrony` on both your head and compute nodes

   ```bash
   sudo dnf install chrony
   ```

1. Head Node
   * Edit the file `/etc/chrony.conf`

     Modify the `allow` declaration to include the internal subnet of your cluster (uncomment or remove the "#" in front of `allow` if it's there, otherwise this is ignored).

   ```bash
   allow 10.0.0.1/24
   ```
   * Start and enable the `chronyd` service
   ```bash
   sudo systemctl enable chronyd

   # The service may be automatically started and enabled after installtion
   # Thus you may need to restart is for changes to take effect.
   sudo systemctl restart chronyd
   ```
   * Verify the NTP synchronization status and that there are no clients connected as yet
   ```bash
   sudo chronyc tracking
   sudo chronyc clients
   ```

1. Compute Node
   * Edit the file `/etc/chrony.conf`

     Comment out (add a "#" in front of) all the `pool` and `server` declarations and add this new line to the file:
   ```bash
   server <headnode_ip>
   ```
   * Restart and enable the `chronyd` service
   ```bash
   sudo systemctl enable chronyd
   sudo systemctl restart chronyd
   ```
   * Verify the sources of the NTP server
   ```bash
   #
   sudo chronyc sources
   ```

