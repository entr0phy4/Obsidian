![[Pasted image 20250421015633.png]]
Scan with [[nmap]] looks [[port 80]] open.

![[Pasted image 20250421015857.png]]
if use [[curl]] we get this response where web app are loading `/js/app.c68eb462.js` file, using curl to see the content:
![[Pasted image 20250421020040.png]]
with grep we can try get some information filtering by key words.

```bash
curl -s http://horizontall.htb/js/app.c68eb462.js | grep -oP '".*?"' | grep http | sort -u
```
with this commands we can obtain this response:
![[Pasted image 20250421020918.png]]
so, we have a subdomain `http://api-prod.horizontall.htb`
remember add it to `/etc/hosts`

If we use [[gobuster]] to perform [[fuzzing]] to new subdomain we get a login panel in `/admin`

use [[searchsploit]] to get information about [[strapi CMS]]

*blah blah blah... elevate privileges*

if u user [[netstat]] can view port running a service in `localhost`
![[Pasted image 20250421021755.png]]
so [[port 8000]] are running, with service ??? let's investigate

using [[curl]] looks like a web page with [[laravel]]
![[Pasted image 20250421021939.png]]
so let's use chisel to do a [[port forwarding]], indeed, its a Laravel v8 (PHP v7.4.18) so we found this cve [[CVE-2021-3129]] that help us to execute remote commands, and ready.