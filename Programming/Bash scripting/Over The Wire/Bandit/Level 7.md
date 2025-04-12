**Password**: morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj
**Tools**: [[grep]], [[awk]]

The password for the [[Level 8]] is stored in the file **data.txt** next to the word *millionth*.

We can use grep to filter by a word, so:
```bash
grep 'millionth' data.txt | awk '{print $2}'
```
using [[awk]] and a pipe we can print only the password 

