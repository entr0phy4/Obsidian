**Password**: 2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ
**Tools**: [[Bash scripting]], [[strings]], [[file]]
## Solution
The password for the [[Level 5]] is stored in the only human-readable file in the **inhere** directory. 

1. We can check filetype of all files with [[file]]
```shell
file ./-file0*
```
And with this we get this:
![[Pasted image 20250408153636.png]]
so, file `-file07` is the unique human readable, if we read that get the password for the next level.

2. With `strings` loop and [[Bash scripting]]:
```bash
for file in $(ls ./inhere); do strings ./inhere/$file; done
```