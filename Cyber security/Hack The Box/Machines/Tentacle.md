![[Pasted image 20250416163528.png]]
with a ping we can know the target machine is a **Linux** by [[ttl]] equal 63.
do a scan with [[nmap]] we can know the open ports and then try to get to know the services running in that ports.
![[Pasted image 20250416163825.png]]
And find this. 
on [[port 3128]] we found a proxy [[squid-http]], searching in the browser `10.10.10.224:3128` 
![[Pasted image 20250416174244.png]]
we pay attention to `j.nakazawa` possible user and `realcorp.htb` domain and `srv01.realcorp.htb` subdomain.
add that to `/etc/hosts`
![[Pasted image 20250416174432.png]]

we can use [[dnsenum]] to effect a brute force attack to find possibles name servers
```bash
dnsenum --dnsserver 10.10.10.224 -f /usr/share/seclist/Discovery/DNS/subdomains-top1million-110000.txt realcorp.htb --threads 51
```
and we found interesting things:
![[Pasted image 20250416175153.png]]
with [[awk]] we can treat that output to save in `/etc/hosts` 
```bash
awk '{print $5 " " $1}'
```

and deleting the useless lines we have:
![[Pasted image 20250416180108.png]]
We can use [[proxychains]] to jump into proxy and scan.
first we need edit `/etc/proxychains.conf` adding the type of proxy IP address of the proxy and port.
```/etc/proxychains.conf
http    10.10.10.224    3128
```
with this ready can use proxychains to try scan around the proxy `127.0.0.1`. We can implement a script to scan ports using [[/dev/tcp]] instead nmap.
```bash
#!/bin/bash
target_host=$1

for port in $(seq 1 65535); do
	proxychains -q timeout 1 bash -c "echo '' > /dev/tcp/$target_host/$port" 2>/dev/null && echo "[+] PORT $port OPEN" &
done
wait
```
and execute
![[Pasted image 20250416183835.png]]
we found another proxy inside! then add too in `/etc/proxychains`
```/etc/proxychains.conf
http    127.0.0.1    3128
```
and try to get `10.197.243.77`
![[Pasted image 20250416190159.png]]
now, we can try aim to another target added to `/etc/hosts` (**wpad.realcorp.htb**), but not works, if we try again jumping into 10.197.243.77:3128 proxy we have access and can scan open ports to **wpad.realcorp.htb**
![[Pasted image 20250416192128.png]]
the first thing that catches your attention is [[port 80]] open, a web page.

If we use [[curl]] to get that page obtain:
![[Pasted image 20250416194129.png]]
searching in your browser about [[WPAD]] we found interesting information (https://blog.alcancelibre.org/staticpages/index.php/como-wpad) 
config of wpad must be save as **wpad.dat**
so we try get that config
```bash
proxychains -q curl wpad.realcorp.htb/wpad.dat
```
![[Pasted image 20250416194459.png]]
and we got!
the [[subred]] `10.241.251.0` we did not have it, so, let's scan it.

we can implement with [[Bash scripting]] a tool to scan most frequently ports in all subred.
```bash
#!/bin/bash

subred=$1
for port in 21 22 25 80 88 443 445 8080 8081; do
  for i in $(seq 1 255); do
    proxychains -q bash -c "echo '' > /dev/tcp/$subred.$i/$port" 2>/dev/null && echo "[+] $subred.$i:$port " &
  done
done
wait
```
at execute we get:
![[Pasted image 20250416201154.png]]
[[port 25]] is for [[Simple Mail Transfer Protocol (SMTP)]]
and if we use nmap to get more info about the service got
```bash
proxychains -q nmap -sCV -sT -Pn -v -n 10.241.251.113 -p25
```
![[Pasted image 20250416201720.png]]
we have a version of smtp service so can use [[searchsploit]] to find a exploit
![[Pasted image 20250416204002.png]]
`linux/remote/47984.py` allows you remote code execution so
`searchsploit -m linux/remote/47984.py`
