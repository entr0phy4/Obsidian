![[Pasted image 20250421192012.png]]
[[nmap]] reports [[port 80]] and [[port 8000]] open. But also show a `.git` is exposed.

we can use tools as [[GitHack]] to download the code of that repository.
```bash
githack http://10.10.11.134
```
![[Pasted image 20250421192309.png]]
now, we get access to some of source code.
in `server.py` we some routes and show us that app it's made with [[Flask]] framework.

![[Pasted image 20250421192807.png]]
we need also a `secret` to get a [[Json Web Token (JWT)]] to can build a cookie `auth` to log in as admin.

In another file `track_api_CR_148.py`
![[Pasted image 20250421193003.png]]
we found that information about [[Amazon Web Services (AWS)]] to connect to the server we need the `aws_access_key_id` and `aws_secret_access_key`
Searching on the logs of git:

we found the commit `7cf92a7a09e523c1c667d13847c9ba22464412f3`
![[Pasted image 20250421193305.png]]
where we can see the keys, so
with [[aws-cli]] tool we can configure the connection:
![[Pasted image 20250421193532.png]]
we know the function implemented is [[lambda]], then can list lambda functions:
![[Pasted image 20250421193756.png]]
and we get one.

with `get-function` we can check deeply that function
![[Pasted image 20250421193905.png]]
and get a location of the function. So use [[wget]] to download it.
![[Pasted image 20250421194132.png]]
we found a file called code but [[file]] tool say us that is a Zip data, to use [[7z]] to decompress and get a python file `lambda_function.py`
![[Pasted image 20250421194247.png]]
where we found a `secret` value.

can be useful to us to create a `jwt` cookie, let's try with [[PyJWT]]

```bash
pip install PyJWT
```

![[Pasted image 20250421194937.png]]
let's try if this works for us like a `auth` cookie

![[Pasted image 20250421195031.png]]
`welcome Admin` say's us that works.

in `/order` we found we can control the render ![[Pasted image 20250421200555.png]]
with that select, so let's use [[Burpsuite]] to intercept that request and try a [[Server Side Template Injection (SSTI)]].

![[Pasted image 20250421200913.png]]
so we could execute commands.
let's get a interactive shell.

once inside we found this cron task are executing this script.
![[Pasted image 20250421212032.png]]

the parameter `-h` of [[tar]] allow the tar binary follow [[symlinks]].

```bash
#!/bin/bash

while true; do
  if [ -e /opt/backups/checksum ]; then
    rm -f /opt/backups/checksum
    echo "[+] File deleted."
    ln -s -f /root/.ssh/id_rsa /opt/backups/checksum
    echo "[+] Symlink created."
    break
  fi
done
```

