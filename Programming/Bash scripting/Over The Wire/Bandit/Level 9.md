**Password**: 4CKMh1JI91bUIZZPXDqGanal4xvAg0JM
**Tools**: [[strings]], [[grep]], [[tail]], [[awk]]

## Solution
The password for the [[Level 10]] is stored in the file **data.txt** in one of the few human-readable strings, preceded by several "=" characters.

We can use [[strings]] to print only human-readable files in data.txt and piping grep to filter by multiples "=" characters.

```bash
strings data.txt | grep "==="
```

we can improve the output to get only the password with [[tail]] and [[awk]]
```bash
strings data.txt | grep "===" | tail -n 1 | awk '{print $2}'
```