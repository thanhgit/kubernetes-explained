# Delve into network
- Netfilter manages the rules that define network communication for the Linux kernel
- These rules permit, deny, route, modify and forward packages
- It organizes these rules into tables according to their purpose

## Netfilter table
### The filter table
- It is responsible for filtering packets
- It consists of 3 chains:
    - Forward chain: filter packets to other server
    - Input chain: filter incomming packets 
    - Output chain: filter outgoing packets
- With the rules:
    - ACCEPT
    - DROP
    - REJECT
    - REFERENCE (refer to a other chain)

### The NAT table
- These rules control network address translation
- They modify the source and destination address for the packet, changing `how the kernel routes the packet`
- It consists of 2 type:
    - Pre-routing chain: change incomming packet IP
    - Post-routing chain: change source packet IP

### The mangle table
- The `headers of packets` which go through this table are altered, changing the way the packet behaves
- Netfilter might shorten the TTL (time to live), redirect it to a different address, or change the number of network hops, TOS (type of service) and MARK

### Raw table
- This table marks packets to `bypass` the iptables stateful connection tracking

### Security table
- This table sets SELinux security contect marks on packets 
- Settings the marks affects how SELinux handle the packets
- The rules in this table set marks on a per-packet or per-connection basis
- A rule defines a set of conditions, and if the packet matches those conditions, an action is taken. For example:
    - Bock all connections originating from a specific IP address
    - Block connections to a NIC
    - Allow all HTTP/HTTPS connections
    - Block connections to specific ports
- 5 default chains that each packet go through:
    - PREROUTING
    - INPUT
    - FORWARD
    - OUTPUT
    - POSTROUTING

## Netfilter modules
### Iptables
- It is a firewall, which is configured and analysis packets and provide NAT engineering
- Filter packets based on MAC and flags in TCP header
- It has ability prevent attack by DDoS
- Command:
    - `-t <table>`: filter, nat, mangle, tables
    - `-j <target>`: jump to a target chain
    - `-A`: adding new rule >> iptables chains
    - `-p <protocol-type>`: icmp, tcp, udp and all
    - `-s <ip-address>`: source IP
    - `-d <ip-address>`: destination IP
    - `-i <interface-name>`: input NIC receive packet
    - `-o <interface-name>`: output NIC go out packet
- Tips:
    - Preventing Syn flooding:
    ```bash
    iptables -A FORWARD -p tcp --syn -m limit --limit 1/s -j ACCEPT
    ```
    - Preventing scan port:
    ```bash
    iptables -A FORWARD -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s -j ACCEPT
    ```

    - Preventing Ping of Death:
    ```bash
    iptables -A FORWARD -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT
    ```
    - Allow the packet which is established connection, continue go through firewall
    ```bash
    iptables -A FORWARD -m state  --state ESTABLISHED,RELATED -j ACCEPT
    ```
    - Preventing fake internal IP from outside
    ```bash
    iptables -t nat -A PREROUTING -i eth0 -s 10.0.0.0/8 -j DROP
    ```
    - Change address from internal to external (SNAT)
    ```bash
    iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to 203.162.0.10
    ```
    - Change address of webserver from external to internal (DNAT)
    ```bash
    iptables -t nat -A PREROUTING -d 203.162.0.9 -p tcp --dport 80 -j DNAT --to 10.0.0.10
    ```
    - Setup Transparent proxy by forwarding port 80 to squid server proxy 10.0.0.9
    ```bash
    iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to 10.0.0.0:31328
    ```
    - Routing internal MAC machine, which has NIC: 00:C7:8F:72:14 go out
    ```bash
    iptables -A FORWARD -m state --state NEW -m mac --mac-source 00:C7:8F:72:14 -j ACCEPT
    ```
    - Priority throughput against website
    ```bash
    iptables -A PREROUTING  -t mangle -p tcp --sport 80 -j TOS --set-tos Maximize-Throughput
    ```

### NAT
- It is used to translation IP network

### Connection tracking
- It allow kernel keep track all logic or session connections
- Using raw table to update state of connections