**Password**: tRae0UfB9v0UzbCdn9cY0gQnds9GF58Q
**Tools**: [[md5sum]], [[cron]]

**Prev**: [[Level 21]]
**Next**: [[Level 23]]
## Solution
A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in **/etc/cron.d/** for the configuration and see what command is being executed.

we follow the steps of the `cronjob_bandit23` and found are executing a script `/usr/bin/cronjob_bandit23.sh`
![[Pasted image 20250410093350.png]]
the script basically create a temp file when save the password of the next level, we can just need found the name of file are being created and then read it.

first we can just execute the script and get the md5 string and read the file.
![[Pasted image 20250410102054.png]]
