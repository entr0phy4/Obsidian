A proxy server is an application act as intermediary when a client request consume o resource that is found in a server.
Instead connect us directly to the server who provides a source as a file or web application, **the client direct the request to the proxy server who is evaluate the request and performs the network request by us.*

### Differences with with a [[Virtual Private Network]]
- A VPN encrypt and encapsulate the traffic, a proxy, no.
- A VPN works in operative system level and redirect all the traffic into the network, the proxy works a application level.
- The VPN connections is usually more stable than a proxy
- The VPN services is usually paid, the proxy services are free.
- The VPN connections can be more slow than a proxy because require encrypt and encapsulate the data.

### Types of proxy servers: Open proxy
- **Forwarding proxy**: Anyone can access of internet.
- **Anonymous proxy:** This server reveals his identity as proxy server but not reveals the client source IP address.
- **Transparent proxy:** This server not only identified as proxy server, but with, the support of the HTTP header fields as [[X-Forwarded-for]], reveals also the client source IP address.

[[Privoxy]]