[el@headnode ~]$ sudo systemctl --failed
  UNIT                               LOAD   ACTIVE SUB    DESCRIPTION
● NetworkManager-wait-online.service loaded failed failed Network Manager Wait Online
LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit act

Solution
sudo systemctl disable NetworkManager-wait-online.service
sudo systemctl reset-failed NetworkManager-wait-online.service