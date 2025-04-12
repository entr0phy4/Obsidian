**Password**: kSkvUpMQ7lBYyCM4GBPvCvT1BfWRy0Dx
**Tools**: [[openssl]], [[Nmap]]
## Solution
The credentials for the [[Level 17]] can be retrieved by submitting the password of the current level to **a port on localhost in the range 31000 to 32000**. First find out which of these ports have a server listening on them. Then find out which of those speak SSL/TLS and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

first we scan ports opens for range in 31000 and 32000, and search services talks with ssl encryption 
```bash
nmap -v --open -n -T5 -p31000-32000 127.0.0.1
```
after try with the ports opens we found the port `31790` then connect with [[openssl]]
```
openssl s_client -quiet -connect 127.0.0.1:31790
```
and when the connection is established send the password of bandit16 and you get a private ssh key, can try connect to bandit17 with that.

```bash
ssh -i private_key_of_openssl bandit17@localhost -p 2220
```
the you got!, can read the password of the level in /etc/bandit_pass/bandit17.
