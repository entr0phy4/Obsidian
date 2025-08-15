# Transmission Control Protocol (TCP)

## Overview
TCP is a connection-oriented, reliable transport layer protocol that provides ordered, error-checked delivery of data between applications running on hosts communicating via an IP network.

## Key Features
- **Reliable delivery**: Guarantees data arrives uncorrupted and in order
- **Connection-oriented**: Establishes connection before data transfer
- **Flow control**: Manages data transmission rate between sender and receiver
- **Congestion control**: Prevents network overload
- **Full-duplex communication**: Bidirectional data flow

## TCP Header Structure
```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## TCP Flags
- **SYN**: Synchronize sequence numbers (connection establishment)
- **ACK**: Acknowledgment field significant
- **FIN**: Finish, no more data (connection termination)
- **RST**: Reset connection
- **PSH**: Push data to application immediately
- **URG**: Urgent pointer field significant

## Connection Management

### Three-Way Handshake (Connection Establishment)
```
Client                    Server
  |                         |
  |------- SYN --------->   |  Step 1: Client requests connection
  |                         |
  |<--- SYN+ACK --------    |  Step 2: Server acknowledges and responds
  |                         |
  |------- ACK --------->   |  Step 3: Client acknowledges
  |                         |
  |    Connection Established|
```

### Four-Way Handshake (Connection Termination)
```
Client                    Server
  |                         |
  |------- FIN --------->   |  Step 1: Client initiates close
  |                         |
  |<------ ACK ----------   |  Step 2: Server acknowledges
  |                         |
  |<------ FIN ----------   |  Step 3: Server initiates close
  |                         |
  |------- ACK --------->   |  Step 4: Client acknowledges
  |                         |
  |    Connection Closed    |
```

## TCP States
- **LISTEN**: Server waiting for connection requests
- **SYN_SENT**: Client has sent SYN, waiting for response
- **SYN_RECEIVED**: Server received SYN, sent SYN+ACK
- **ESTABLISHED**: Connection is open and data can flow
- **FIN_WAIT_1**: Client sent FIN, waiting for ACK
- **FIN_WAIT_2**: Client received ACK for FIN, waiting for server FIN
- **CLOSE_WAIT**: Server received FIN, waiting to send its FIN
- **CLOSING**: Both sides have sent FIN simultaneously
- **LAST_ACK**: Server sent FIN, waiting for final ACK
- **TIME_WAIT**: Client waiting to ensure server received final ACK
- **CLOSED**: Connection is closed

## Flow Control
TCP uses a sliding window mechanism to control data flow:

```bash
# Window size determines how much unacknowledged data can be sent
Sender Window = Receiver's Advertised Window - Unacknowledged Data

# Example:
# If receiver advertises window size of 8192 bytes
# and 1024 bytes are unacknowledged
# Sender can send 7168 more bytes before waiting for ACK
```

## Common TCP Ports
```bash
20/21   - FTP (File Transfer Protocol)
22      - SSH (Secure Shell)
23      - Telnet
25      - SMTP (Simple Mail Transfer Protocol)
53      - DNS (Domain Name System)
80      - HTTP (Hypertext Transfer Protocol)
110     - POP3 (Post Office Protocol)
143     - IMAP (Internet Message Access Protocol)
443     - HTTPS (HTTP Secure)
993     - IMAPS (IMAP Secure)
995     - POP3S (POP3 Secure)
```

## TCP vs UDP Comparison
| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable delivery | Best-effort delivery |
| Ordering | Ordered delivery | No ordering guarantee |
| Flow Control | Yes | No |
| Congestion Control | Yes | No |
| Header Size | 20+ bytes | 8 bytes |
| Speed | Slower (more overhead) | Faster (less overhead) |
| Use Cases | HTTP, HTTPS, FTP, SSH | DNS, DHCP, streaming |

## TCP Configuration

### Linux TCP Parameters
```bash
# View TCP settings
sysctl net.ipv4.tcp_congestion_control
sysctl net.core.rmem_max
sysctl net.core.wmem_max

# Common TCP tuning parameters
net.core.rmem_max = 16777216          # Max receive buffer
net.core.wmem_max = 16777216          # Max send buffer
net.ipv4.tcp_rmem = 4096 87380 16777216  # TCP receive buffer
net.ipv4.tcp_wmem = 4096 16384 16777216  # TCP send buffer
net.ipv4.tcp_congestion_control = bbr    # Congestion control algorithm
```

### Monitoring TCP Connections
```bash
# Show all TCP connections
netstat -t
ss -t

# Show listening TCP ports
netstat -tl
ss -tl

# Show TCP statistics
netstat -s | grep -A 20 "Tcp:"
ss -s

# Monitor specific connection
tcpdump -i any -n tcp port 80
```

## Security Considerations

### TCP SYN Flood Protection
```bash
# Enable SYN cookies
echo 1 > /proc/sys/net/ipv4/tcp_syncookies

# Reduce SYN timeout
echo 1 > /proc/sys/net/ipv4/tcp_syn_retries

# Limit half-open connections
echo 1024 > /proc/sys/net/ipv4/tcp_max_syn_backlog
```

### Common TCP Attacks
- **SYN Flood**: Overwhelm server with SYN requests
- **Connection Hijacking**: Take over established connection
- **RST Attack**: Force connection termination
- **Sequence Number Prediction**: Predict next sequence number

## Troubleshooting TCP Issues

### Common Problems
```bash
# Connection refused
telnet hostname port  # Test if port is open

# Slow connections
# Check window scaling
tcpdump -i any -n -S tcp and host hostname

# High retransmissions
ss -i  # Show interface statistics

# Connection timeouts
# Check firewall rules
iptables -L -n
```

### Performance Analysis
```bash
# Monitor TCP retransmissions
grep -i retrans /proc/net/snmp

# Check connection queue
ss -l  # Show listen queue sizes

# Analyze packet capture
tcpdump -i any -w capture.pcap tcp
wireshark capture.pcap
```

## Related Protocols
- [[User Datagram Protocol (UDP)]] - Alternative transport protocol
- [[Internet Protocol (IP)]] - Network layer protocol
- [[Internet Control Message Protocol (ICMP)]] - Error reporting

## Related Topics
- [[Network programming]]
- [[Socket programming]]
- [[Network troubleshooting]]
- [[Performance tuning]]
- [[Network security]]
