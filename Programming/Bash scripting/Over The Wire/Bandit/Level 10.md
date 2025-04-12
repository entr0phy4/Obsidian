**Password**: FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey
**Tools**: [[base64]]

## Solution
The password for the [[Level 11]] is stored in the file **data.txt** which contains base64 encoded data

we can decode base64 data with [[base64]] and the parameter -d (decode)

```bash
cat data.txt | base64 -d | awk 'NF{print $NF}'
```