**Password**: qp30ex3VLz5MDG1n91YowTv4Q8l7CDZL
**Tools**: [[git]]

**Prev**: [[Level 29]]
**Next**: [[Level 31]]

## Solution
There is a git repository at `ssh://bandit30-git@localhost/home/bandit30-git/repo` via the port `2220`. The password for the user `bandit30-git` is the same as for the user `bandit30`.

Clone the repository and find the password for the next level.

In this level we need check `tags` of git.
```bash
git tag
```

![[Pasted image 20250410141949.png]]
and read it
```bash
git show secret
```
and u got the password