**Password**: Yz9IpL0sBcCeuG7m9uQFt8ZNpS4HZRcN
**Tools**: [[git]]

**Prev**: [[Level 27]]
**Next**: [[Level 29]]

## Solution
There is a git repository at `ssh://bandit28-git@localhost/home/bandit28-git/repo` via the port `2220`. The password for the user `bandit28-git` is the same as for the user `bandit28`.

Clone the repository and find the password for the next level.

At clone the repository and print `README.md` the password is crossed out with `xxxxxxx` but if is a repository we can see the history of logs with [[git]]
```bash
git log -p
```
![[Pasted image 20250410140404.png]]