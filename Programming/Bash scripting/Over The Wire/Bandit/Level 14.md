**Password**: MU4VWeTyJk8ROof1qqmcBPaLh7lDCPvS
**Tools**: [[netcat (nc)]], [[telnet]]

### Solution
The password for the [[Level 15]] can be retrieved by submitting the password of the current level to **port 30000 on localhost**.

the pass of the current level are in /etc/bandit_pass/bandit14 so we can send that in the port 30000:

```bash
echo 8xCjnmgoKbGLhHFAZlGE5Tmu4M2tKJQo | nc localhost 30000
```

we  use the pipes to redirect the output of the echo to send to localhost:30000 using [[netcat (nc)]]

another way is using [[telnet]]
```bash
telnet localhost 30000
```

after establishing the connection send the password of bandit14.

