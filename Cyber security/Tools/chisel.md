A fast TCP/UDP tunnel over HTTP.

On host machine start a chisel server.

```bash
sudo ./chisel server --reverse -p <port_of_server>
```

On target machine spawn a client.

```bash
./chisel client <ip_chisel_server>:<port_chisel_server>
```

---
### Error `'GLIBC_2.3x'` not found.

```bash
CGO_ENABLE=0 GOOS=linux GOARCH=amd64 go build -o . -ldflags "-s -w"
```

**Flags**
`-s` Delete symbol table.
`-w` Delete debugging information.

---
Can use [[upx]] to compress the binary resultant 