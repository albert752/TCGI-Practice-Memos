# TCGI Command and config file summary
## Bridge control and VLAN
`brctl show` shows current bridge config
`brctl addbr <brname>` create bridge
`brctl addif <brname> <iface>` add iface to the bridge
**TIP:** To enter in fw state, briges MUST have an @IP

`vconfig add <iface> <tag_id>` tag the iface
**!** Now to add the iface to the brige the iface names is `<iface>.<tag>`

**Do not forget to enable (up) the ifaces!**

`brctl delif <bridge> <iface>` delete iface from bridge
`brctl showmacs <bridge>` shows the learned @MACs

## Static routing and ARP
`route add -net <netid>/<mask> gw <gw>` add an entry
`echo 1 > /proc/sys/net/ipv4/ip_forward` enable routing
`arp -v` check the ARP cache
`arp -s <@IP> <@MAC> temp` add an entry to the ARP cache

## APPS
`cat /etc/services` list of all services
`netstat -tnlp` active tcp sockets, t for tcp, l for listening
`nc -l -p 12345` netcat server @ port 1234
`nc <@IP> <port>` netcat client
`lsof -a -p <PID> -d0-10` list of open files from 0 to 10 of PID
`sudo scp file.txt root@<@IP>:/path` send the file tp @IP to path

**Other app commands can be found on the PPT and Documents**

## DNS
`dig <domain name>` get dns info
`/etc/resolv.conf` name server  info:

```
nameserver 10.0.0.21
search example.com
```
**!** All bind config files are located @ `/etc/bind/`

## DHCP
`/var/lib/dhcp3/dhclient.leases` in a DHCP server, all the lease info
`dhclient <iface>` activate dhcp on iface
`dhclient -r <iface>` release the @IP

## APACHE
`/etc/init.d/apache2 start` start the daemon

## iptables
List of examples for firewall:
* `iptables -t filter -A INPUT -p ICMP -j DROP` add drop input ICMPS
* `iptables -t filter -D INPUT -p ICMP -j DROP` remove previous config
* `iptables -t filter -A INPUT -p ICMP --icmp-type echo-request -j DROP`

* `iptables -t filter -A FORWARD -p ICMP -s 192.168.1.0/24 --icmp-type echo-request -j ACCEPT`

* `Rint:~# iptables -t filter -A FORWARD -p ICMP --icmp-type echo-request -j DROP`
* `iptables -t filter -A FORWARD -s 192.168.1.0/24 -p tcp --syn -j ACCEPT`
* `iptables -t filter -A FORWARD -p tcp --syn -j DROP`
* `-A FORWARD -s 172.16.1.5/32 -p udp -j ACCEPT`
* `-A FORWARD -d 172.16.1.5/32 -p udp -j ACCEPT`
* `-A FORWARD -p udp -j DROP`

List of examples for NAT:
* `iptables -t nat -A POSTROUTING -o eth2 -j SNAT --to 10.0.2.2`
* `iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j DNAT --to 172.16.1.2`

## TUNNEL
Example of the creation fo an IPIP tunnel between two ifaces 198.51.100.2 and
192.0.2.2 with inherited ttl and no nomptudisc policy:

```
R2:~# ip tunnel add tunnel0 mode ipip local 198.51.100.2 remote 192.0.2.2 ttl inherit nopmtudisc dev eth2
R2:~# ifconfig tunnel  1.2.3.4
R2:~# route add -net 192.168.0.0/24 dev tunnel0
```

```
R1:~# ip tunnel add tunnel0 mode ipip local 192.0.2.2 remote 198.51.100.2 ttl inherit nopmtudisc dev eth2
R1:~# ifconfig tunnel0 4.3.2.1
R1:~# route add -net 172.16.1.0/24 dev tunnel0
```

`ip route show cache` to see the tunnel info

`host3:~# ip route add 192.168.0.0/24 via 172.16.1.1 advmss 936` advertise another MSS
`iptables -t mangle -A FORWARD -o tunnel0 -p tcp --syn -j TCPMSS --set-mss 936`
change the MSS vai NAT. (MSS payload of TCP max size)

## IP Multicast
`cat /proc/net/igmp` show mc info
`netstat -gn ` show m/c info
`/proc/net/ip_mr_vif` this file contains the interfaces that are involved in multicast operations in the multicast router, and some statistics of usage
`/proc/net/ip_mr_cache` this file shows the contents of the Multicast Forwarding Cache.

Configuration of m/c:
```
R1:~# echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts
R1:~# smcroute -d
R1:~# smcroute -a tunnel0 0.0.0.0 232.43.211.234 eth1
R1:~# smcroute -j tunnel0 232.43.211.234
```
1. Enable m/c
2. Start the daemon
3. Add the route from tunnel0 from any @IP to @IP fw to eth1 (multiple ifaces
   can be stated)

`smcroute -k` kill the deamon

`mcsender -t3 232.43.211.234:1234` starts a sender with TTL 3 to m/c:port
`emcast 232.43.211.234:1234` starts a listener of m/c:port

This block checks the m/c connection form the server to the host:
```
server:~# ssmpingd
host4:~# ssmping 172.16.1.3
```

## Disclaimer
You may find some error or typos, if you do please report them.

## Author

* **Albert Azemar i Rovira** - *Initial work* -
	[albert752](https://github.com/albert752)

## License

This project is licensed under the MIT License - see the
[LICENSE.md](./LICENSE.md) file for details
