**Password**: upsNCc7vzaRDx6oZC6GiR6ERwe1MowGB
**Tools**: [[ssh]], [[git]]

**Prev**: [[Level 26]]
**Next**: [[Level 28]]
## Solution
There is a git repository at `ssh://bandit27-git@localhost/home/bandit27-git/repo` via the port `2220`. The password for the user `bandit27-git` is the same as for the user `bandit27`.

Clone the repository and find the password for the next level.

Inside a temp repository created with `mktemp -d` we need use `git` to clone the repository via `ssh` on the port `2220`

```bash
git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo
```

password for the next level is in `README` file.