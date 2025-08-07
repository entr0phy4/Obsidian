The first step to start to experiment is have a network card in `monitor mode`, we can do this with `airmon-ng`.
with `ifconfig` of `net-tools` package we can display all network interfaces in our device.
![[Pasted image 20250412203237.png]]
I going to use `wlo1`.
To set `wlo1` in monitor mode is easiest using airmon-ng
```bash
sudo airmon-ng start wlo1
```
![[Pasted image 20250412203521.png]]
if we check again `ifconfig` we now have a "new" network interface called `wlo1mon`
![[Pasted image 20250412203617.png]]