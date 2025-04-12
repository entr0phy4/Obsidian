**Password**: dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr
**Tools**: [[tr]]

## Solution
The password for the [[Level 12]] is stored in the file **data.txt**, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.

If we print the content of data.txt we get this:
![[Pasted image 20250408195852.png]]
So we start rotate 13 positions from G using tr tool for the translation.

```bash
cat data.txt | tr '[G-ZA-Fg-za-g]' '[T-ZA-St-za-s]'
```
