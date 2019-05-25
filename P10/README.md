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
		* Its is advertising the network 192.168.1.0 with metric 1, no mask
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

7. By capturing on SimNet 2, 3, 4 and 6 we can see that, when we run `network
   192.168.0.0/16` on r1:
	* We capture RIP messages on SimNets 2, 3 and 4.
	* The first message is the usual request when we start the RIP protocol.
	* Then, the following ones advertise the complementary two nets that are
		connected to the router for each iface that fit the specified net id.
	* The metric is obviosly 1.
	* Regarding the addres:
		* They all have as src the ip/mac of the iface
		* They all have as dst the broadcast addr.
		* The port is set always to the 520/UDP
	* We can not see any messages on SimNet6 because its @IP does not fit the
		provided NetID, hence RIP has not been activated for that network in
		particular.
8. I think it does not make sense because there aren't any routers connected.
   But, if we don't know the topology of the net (we don't need it to run RIP,
   it is not mandatory), it would be a good idea to send them just in case a
   router gets connected or exist in fisrt place. The again we could use the
   passive mode and wait for a request, lowering the charge of the net.

9. After deleting the network, we don't see any RIP messages, only two IGMP
   leave group. Then, before 2 minutes, when we restore for nets 3.0 and 2.0, we
   can see two requests and the normal responses advertising 0.3 on SimNet2

10. After adding network 5.0, we cannot see any messages from r3 on SimNets 2
	and 3 because those have not been aded to the RIP protocol of r3.
	* After adding the 2.0 net to r3, we can see a request from r3 followed
		from a respons from r1 advertising net 3.0 and also a response from r3
		advertising net 5.0.
	* We can see that both routers have learned the routes:
```
		r1# show ip rip
Codes: R - RIP, C - connected, S - Static, O - OSPF, B - BGP
Sub-codes:
      (n) - normal, (s) - static, (d) - default, (r) - redistribute,
      (i) - interface

     Network            Next Hop         Metric From            Tag Time
C(i) 192.168.2.0/24     0.0.0.0               1 self              0
C(i) 192.168.3.0/24     0.0.0.0               1 self              0
R(n) 192.168.5.0/24     192.168.2.3           2 192.168.2.3       0 02:52

```

```
r3# show ip rip
Codes: R - RIP, C - connected, S - Static, O - OSPF, B - BGP
Sub-codes:
      (n) - normal, (s) - static, (d) - default, (r) - redistribute,
      (i) - interface

     Network            Next Hop         Metric From            Tag Time
C(i) 192.168.2.0/24     0.0.0.0               1 self              0
R(n) 192.168.3.0/24     192.168.2.1           2 192.168.2.1       0 02:56
C(i) 192.168.5.0/24     0.0.0.0               1 self              0
```
.
	* In R1, triggered updates has been set up because whenever we change the
		state of any link, r1 generates a ressponse advertising the new status.
	* We cannot see any response taht matches the poissoned revers behabiour.
	* Split horizon is activated because r3 is not anouncing 0.3 to r1.

11. 
	* When we ping from r3 to 192.168.3.1:
		* The ping works flawlesly
		* It works because r3 has learned the route from r1.
	* When we ping from r3 to 192.168.4.11:
		* It does not work
		* Nwtwork unreachable
		* The net is not inside the rip domain nor it's a direct route.
	* If we examine the RIB and FIB of r3:
**THE RIB FOR RIP**
```
		r3# show ip rip
Codes: R - RIP, C - connected, S - Static, O - OSPF, B - BGP
Sub-codes:
      (n) - normal, (s) - static, (d) - default, (r) - redistribute,
      (i) - interface

     Network            Next Hop         Metric From            Tag Time
C(i) 192.168.2.0/24     0.0.0.0               1 self              0
R(n) 192.168.3.0/24     192.168.2.1           2 192.168.2.1       0 02:42
C(i) 192.168.5.0/24     0.0.0.0               1 self              0

```
**FIB/RIB FOR FOR ALL PROTOCOLS**
```
r3# show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP, O - OSPF,
       I - ISIS, B - BGP, > - selected route, * - FIB route

C>* 127.0.0.0/8 is directly connected, lo
C>* 192.168.0.0/24 is directly connected, eth4
C>* 192.168.1.0/24 is directly connected, eth3
C>* 192.168.2.0/24 is directly connected, eth2
R>* 192.168.3.0/24 [120/2] via 192.168.2.1, eth2, 00:12:11
C>* 192.168.5.0/24 is directly connected, eth1
```

**THE FIB**
```
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.0.0     0.0.0.0         255.255.255.0   U     0      0        0 eth4
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth3
192.168.2.0     0.0.0.0         255.255.255.0   U     0      0        0 eth2
192.168.3.0     192.168.2.1     255.255.255.0   UG    2      0        0 eth2
192.168.5.0     0.0.0.0         255.255.255.0   U     0      0        0 eth1
```

We can see that the route to network 4 has not been added but for network 3 it
has.

12. Now we are going to add the net 4 with eth4 in passive mode:
```
r1(config-router)# passive-interface eth4
r1(config-router)# network 192.168.4.0/24
```
	* While capturing on SimNWt4 we only see a RIP request but no responses as
		expected.
	* When we try again the ping, it works.
	* The route to the net 4 has been added to both RIB and FIB like the 3.

13. The RIP responses generated by r222 on SimNet3 advertise networks 0 and 1
	with metric 2 and network 5 with metric 1 as expected.
```
r222# show ip rip
Codes: R - RIP, C - connected, S - Static, O - OSPF, B - BGP
Sub-codes:
      (n) - normal, (s) - static, (d) - default, (r) - redistribute,
      (i) - interface

     Network            Next Hop         Metric From            Tag Time
R(n) 192.168.0.0/24     192.168.5.3           2 192.168.5.3       0 02:51
R(n) 192.168.1.0/24     192.168.5.3           2 192.168.5.3       0 02:51
R(n) 192.168.2.0/24     192.168.3.1           2 192.168.3.1       0 02:51
C(i) 192.168.3.0/24     0.0.0.0               1 self              0
R(n) 192.168.4.0/24     192.168.3.1           2 192.168.3.1       0 02:51
C(i) 192.168.5.0/24     0.0.0.0               1 self              0
```
.
	* As it can be seen, it contains teh two nets that we configured before as wella
		s all the learnet networks form r1 and r3 with the correct number of hops
		(metric)
	* Sending a ping to 172.16.0.1 should not work beacuse it has not note to
		this network.

14. Both routers have routes set outside the scope of RIP. When we run
	`redistribute connected`.
	**Before runing the command**
	```
	r222# show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP, O - OSPF,
       I - ISIS, B - BGP, > - selected route, * - FIB route

C>* 127.0.0.0/8 is directly connected, lo
R>* 192.168.0.0/24 [120/2] via 192.168.5.3, eth2, 00:02:03
C>* 192.168.0.128/25 is directly connected, eth3
R>* 192.168.1.0/24 [120/2] via 192.168.5.3, eth2, 00:02:03
R>* 192.168.2.0/24 [120/2] via 192.168.3.1, eth1, 00:22:42
C>* 192.168.3.0/24 is directly connected, eth1
R>* 192.168.4.0/24 [120/2] via 192.168.3.1, eth1, 00:22:42
C>* 192.168.5.0/24 is directly connected, eth2

	```
	```
	r1# show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP, O - OSPF,
       I - ISIS, B - BGP, > - selected route, * - FIB route

C>* 127.0.0.0/8 is directly connected, lo
C>* 172.16.0.0/16 is directly connected, eth1
R>* 192.168.0.0/24 [120/2] via 192.168.2.3, eth2, 00:03:06
R>* 192.168.1.0/24 [120/2] via 192.168.2.3, eth2, 00:03:06
C>* 192.168.2.0/24 is directly connected, eth2
C>* 192.168.3.0/24 is directly connected, eth3
C>* 192.168.4.0/24 is directly connected, eth4
R>* 192.168.5.0/24 [120/2] via 192.168.3.222, eth3, 00:23:44

	```
	**After running the commmand**
	```
	r222# show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP, O - OSPF,
       I - ISIS, B - BGP, > - selected route, * - FIB route

C>* 127.0.0.0/8 is directly connected, lo
R>* 172.16.0.0/16 [120/2] via 192.168.3.1, eth1, 00:03:21
R>* 192.168.0.0/24 [120/2] via 192.168.5.3, eth2, 00:07:15
C>* 192.168.0.128/25 is directly connected, eth3
R>* 192.168.1.0/24 [120/2] via 192.168.5.3, eth2, 00:07:15
R>* 192.168.2.0/24 [120/2] via 192.168.3.1, eth1, 00:27:54
C>* 192.168.3.0/24 is directly connected, eth1
R>* 192.168.4.0/24 [120/2] via 192.168.3.1, eth1, 00:27:54
C>* 192.168.5.0/24 is directly connected, eth2
	```
	```
	<r1# show ip route  
Codes: K - kernel route, C - connected, S - static, R - RIP, O - OSPF,
       I - ISIS, B - BGP, > - selected route, * - FIB route

C>* 127.0.0.0/8 is directly connected, lo
C>* 172.16.0.0/16 is directly connected, eth1
R>* 192.168.0.0/24 [120/2] via 192.168.2.3, eth2, 00:08:29
R>* 192.168.1.0/24 [120/2] via 192.168.2.3, eth2, 00:08:29
C>* 192.168.2.0/24 is directly connected, eth2
C>* 192.168.3.0/24 is directly connected, eth3
C>* 192.168.4.0/24 is directly connected, eth4
R>* 192.168.5.0/24 [120/2] via 192.168.3.222, eth3, 00:29:07
	```
	* We can see taht the 172.16.0.0/16 network gets distributed whereas the
		192.16.0.128/28 does not because it is not a classfull net.
	* There are no diferences in the entries.

15. Now that r222 has the route, the ping works. RIPv1 is a classfull protocol
	hence it will only distribute classful @IP.
16. The ping form h223 to h11 does not work. The request reaches the host and
	it generates a response. This response can not be routed by r1.
	* On SimNet7 we can see some ARP messages from r3 reqeusting the @MACof
		h223.
	The request goes from h223 to h11 through SimNet3. 
	The response goes from h11 to r1 who routes it to r3 due to the entry
	192.168.0.0/24. R3 tries to reach the host on network 0 (because it fits
	the mask) and generates an ICMP network unreachable that get fw to SimNet4.
## Issues
* Is adding an interface the same as adding its network?
* When we run a no network, why it does not get anounced through the net?
	(RIPv1-9)
* 192.16.0.128/28 does not because it is not a classfull net? (RIPv1-14)

