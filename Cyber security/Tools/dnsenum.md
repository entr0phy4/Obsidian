Multi-threaded perl script to enumerate [[Domain Name System (DNS)]] information of a domain and to discover non-contiguous IP blocks.
```bash
dnsenum --dnsserver 10.10.10.224 -f /usr/share/seclist/Discovery/DNS/subdomains-top1million-110000.txt realcorp.htb --threads 51
```