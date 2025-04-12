**Password**: HWasnPhtq9AVKe0dmk45nxy20cvUa6EG
**Tools**: [[find]], [[xargs]], [[Redirections]]

## Solution
The password for the [[Level 7]] is stored somewhere on the server an has all of the following properties:
- owned by user bandit7
- owned by group bandit6
- 33 bytes in size
- 
with [[find]] we can filter fulfilling all that properties and with [[xargs]] can read immediately result file. we need redirect the error message to nothing.

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```