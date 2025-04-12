**Password**: 7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4
**Tools**: [[xxd]], [[7z]], [[Bash scripting]], [[grep]], [[tail]]

## Solution
The password for the [[Level 13]] stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work. Use mkdir with a hard to guess directory name. Or better, use the command “mktemp -d”. Then copy the datafile using cp, and rename it using mv (read the manpages!)

if we read **data.txt** file we found a hexdumped here, using [[xxd]] we can revert this with option `-r` and redirect the result to a new file called content

```bash
xxd -r data.txt > content
```

with [[file]] we can know what type of file is content and we found a `.gzip` file. Like the problem says, content.gzip has another compressed inside, and another and another.
then we going to write a tool in bash to unzip all files unzippable.

first, we need obtain the name of the zip inside the current file, using [[7z]] we can list contents of a zip with `l` flag.

```bash
7z l content.zip
```

but the result its a little confused, we just need the column "Name", then, let's treat the output.

![[Pasted image 20250409132943.png]]

```
7z l content.gzip | grep "Name" -A 2 | tail -n 1 | awk 'NF{print $NF}'
```

with [[grep]] we get only the `Name` column and with params `-A 2` gets two lines down from Name.
with [[tail]] says only wan see the last row of that and with [[awk]] get only the last value of that row.

![[Pasted image 20250409133419.png]]
All right! with this we can get only the name of the zip inside the zip.
with this we can start the script.

```bash
#!/bin/bash

current_file=$(7z l $1 | grep "Name" -A 2 | tail -n 1 | awk 'NF{print $NF}')
7z x $! >/dev/null 2>&1

while true; do
	7z l $current_file >/dev/null 2>&1
	if [ "$(echo $?)" == "0" ]; then
		next_file=$(7z l $current_file | grep "Name" -A 2 | tail -n 1 | awk 'NF{print $NF}')
		7z x $current_file >/dev/null 2>&1 && current_file=$next_file
	else 
		exit 1
	fi
done
```

with this we save the name of the zip in a var called current_file, and basically we first unzip that, after we check if the current_file can be unzipped, if yes the code going save the name of the zip inside on new variable called next_file, before equal current_file with next_file we unzip the current_file to in the next round work with the zip recently unzipped.

with this script we got unzip all files in chains until there are no more file to unzip.