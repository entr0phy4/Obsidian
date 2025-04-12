**Password**: 0qXahG8ZjOVMN9Ghs7iOWsCfZyXOUbYO
**Tools**: [[setuid]], [[netcat (nc)]]

## Solution
There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the [[Level 21]] (bandit21).

we need turn on a listen port with [[netcat (nc)]] in some port. then:
```bash
nc -nlvp 9001
```
now, we use the binary `suconnect` to establish a connection in that port
```bash
./suconnect 9001
```
then the connection  has been established we provide the password of the current level (bandit20) to netcat and we should receive the next password.