**Password**: EeoULMCra2q0dSkYj561DX7s1CpBuOBt
**Tools**: [[cron]]

**Prev**: [[Level 20]] 
**Next**: [[Level 22]]
## Solution
A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

we can start check inside **/etc/cron.d** directory
```bash
ls -la /etc/cron.d/
```
![[Pasted image 20250410090044.png]]
thinking we are in the `bandit21` we should check `cronjob_bandit22`
![[Pasted image 20250410090305.png]]
we found the cronjob are executing a script `/usr/bin/cronjob_bandit22`
![[Pasted image 20250410090514.png]]
basically, the script are saving the password of `bandit22` in a temp directory `/tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv`

```bash
cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```
and we got the password of the next level.