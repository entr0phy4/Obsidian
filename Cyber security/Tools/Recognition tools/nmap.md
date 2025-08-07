A free and open-source network scanner tool. Nmap supports various scan types and protocols, including TCP, UDP, SYN and more.

---
[[Transmission Control Protocol (TCP)]]

```bash
nmap -sCV -p- --min-rate 5000 -vvv -Pn -n ip_target -oN output
```

---
[[User Datagram Protocol (UDP)]]

```bash
nmap -sU --top-ports 500 ip_target -oN output
```