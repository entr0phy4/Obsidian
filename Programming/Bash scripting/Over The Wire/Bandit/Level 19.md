**Password**: cGWpMaKXVwDUNgPAVJbWYuGHVn9zl3j8
**Tools**: [[setuid]]

## Solution
To gain access to the [[Level 20]], you should use the [[setuid]] binary in the homedirectory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used the setuid binary.

we can use a [[setuid]] binary `bandit20-do` to execute commands being the user `bandit20`

```bash
./bandit20-do whoami
# bandit20
./bandit20-do cat /etc/bandit_pass/bandit20
# password
```