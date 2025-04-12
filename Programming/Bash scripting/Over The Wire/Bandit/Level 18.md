**Password**: x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO
**Tools**: [[ssh]]

## Solution
The password for the [[Level 19]] is stored in a file **readme** in the homedirectory. Unfortunately, someone has modified **.bashrc** to log you out when you log in with SSH.

we found a little bit of delay when we try connect via [[ssh]], and the server log out us, but we can try execute command in this time, i.e. spawn a bash.

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 bash
```

or read directly file **readme** file.

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme
```