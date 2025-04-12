**Password**: FO5dwFsc0cbaIiH0h8J2eUks2vdTDwAn
**Tools**: [[ssh]]

### Solution
The password for the [[Level 14]] is stored in **/etc/bandit_pass/bandit14 and can only be read by user bandit14**. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. **Note:** **localhost** is a hostname that refers to the machine you are working on

using the sshkey.private we can connect to bandit14.

```bash
ssh -i sshkey.private bandit14@localhost -p 2220
```

