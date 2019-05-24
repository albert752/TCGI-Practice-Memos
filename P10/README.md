# Practice 10 Memo TCGI
## RIP
### Exercice 1
1. We run the mentioned pings and these are the results:
```
root@r3:~# ping -c10 192.168.2.1
PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
64 bytes from 192.168.2.1: icmp_req=1 ttl=64 time=0.688 ms
64 bytes from 192.168.2.1: icmp_req=2 ttl=64 time=0.325 ms
64 bytes from 192.168.2.1: icmp_req=3 ttl=64 time=0.299 ms
64 bytes from 192.168.2.1: icmp_req=4 ttl=64 time=0.291 ms
64 bytes from 192.168.2.1: icmp_req=5 ttl=64 time=0.286 ms
64 bytes from 192.168.2.1: icmp_req=6 ttl=64 time=0.452 ms
64 bytes from 192.168.2.1: icmp_req=7 ttl=64 time=0.517 ms
64 bytes from 192.168.2.1: icmp_req=8 ttl=64 time=0.369 ms
64 bytes from 192.168.2.1: icmp_req=9 ttl=64 time=0.326 ms
64 bytes from 192.168.2.1: icmp_req=10 ttl=64 time=0.305 ms

--- 192.168.2.1 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9023ms
rtt min/avg/max/mdev = 0.286/0.385/0.688/0.126 ms
```

```
root@r3:~# ping -c10 192.168.3.1
connect: Network is unreachable
```

```
root@r3:~# ping -c10 192.168.4.11
connect: Network is unreachable
```
Due to the fact that no default gateway has been sat, we
can only reach the networks connected directly to the router. 
This is why only the eth2 from r1 responds our request, the
other pings do not even get routed.

2. After running the following commands:

```
root@r3:~# vtysh

Hello, this is Quagga (version 0.99.20.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

r3# configure terminal
r3(config)# router rip
r3(config-router)# version 1
r3(config-router)# network 192.168.1.0/24
r3(config-router)# network 192.168.2.0/24
```

In SimNet2 we can see 4 packets. Two IGMP memebership join for the m/c RIP
group and two RIP frames. We'll ignore the m/c frames, the have been 
generated before setting the RIP version and do not correspond to V1.

* **RIP response:** It is sent periodicaly, every 25secs
	* Addrs:
		* L2 addr: Src: fe:fd:00:00:03:02 (fe:fd:00:00:03:02), Dst: Broadcast (ff:ff:ff:ff:ff:ff)
		* L3 addr: Src: 192.168.2.3, Dst: 192.168.2.255
		* L4 addr: User Datagram Protocol, Src Port: 520, Dst Port: 520
	* We can see as expected taht the frames are sent to the broadcast addrs
		and from/to port 520 UDP
	* Payload:
		* Its is advertising the network 192.168.1.0 with metric 0, no mask
			specified.

3. After running `neighbor 192.168.2.1` we set the @IP as a neighbor so unicast
   RIP resposes will be sent to him directly. Due to the fact that eth2 is not
   running RIP, we can see that ICMP error from `192.168.2.1` reporting port unreachable.

4. If we now kill eth3 interface and capture for two minutes on SimNet3, we can
   see that:
	* During the fist two minutes, the metric changes from 1 to 16, so to
	   infinity.
	* After the two minutes, no routes are advertised due to the garbage
		collection.

5. After restoring the link with eth3, we can see the following on SimNet2:
	* We can see a new response with metric 1. It is instantanious so we think
		that it might be set up as triggered.

6. When we remove the net 192.168.1.0/24 for the R3, it inmediatly generates a
   response for this net with metric 16. After 2 minutes, the messages stop.

7. 

## Issues
* Is adding an interface the same as adding its network?
