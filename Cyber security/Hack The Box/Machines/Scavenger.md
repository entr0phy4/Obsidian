![[Pasted image 20250501040142.png]]

with [[nmap]] we get this information about open ports, we can try connect to [[port 21]] with [[ftp]] as user `anonymous` but we need credentials.

if we try connect to [[port 43]] [[WHOIS]] is running, so let's try with [[netcat (nc)]]

```bash
nc 10.10.10.155 43
```

and if we send a simple quote we get this error:

![[Pasted image 20250501005133.png]]

So we can try with events [[SQL Injection]].

![[Pasted image 20250501005230.png]]

Testing with different query's we get:

![[Pasted image 20250501142026.png]]

new domains.