# Practice 2 memo of TCGI

## Exercice 1
### Section 1

```
Every 1,0s: brctl showmacs br3                          Mon M80004 12:15:44
2019
port no mac addr                is local?       ageing timer
  1     fe:fd:00:00:02:00       no                 6.00
  4     fe:fd:00:00:06:00       no                 6.00
  1     fe:fd:00:00:09:00       yes                0.00
  2     fe:fd:00:00:09:01       yes                0.00
  3     fe:fd:00:00:09:02       yes                0.00
  4     fe:fd:00:00:09:03       yes                0.00

```

```
Every 1,0s: brctl showmacs br2                          Mon M40004 12:16:34
2019
port no mac addr                is local?       ageing timer
  2     fe:fd:00:00:02:00       no                31.20
  1     fe:fd:00:00:06:00       no                31.20
  1     fe:fd:00:00:08:00       yes                0.00
  2     fe:fd:00:00:08:01       yes                0.00
  3     fe:fd:00:00:08:02       yes                0.00

```

#EX 1.2
Alice to bob
```
Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@alice:~# ifconfig eth0 192.168.100.2
root@alice:~# route add -net 192.168.101.0/24 gw 192.168.100.1
root@alice:~# route add -net 192.168.102.0/24 gw 192.168.100.1
root@alice:~# ping 192.168.102.2 -c1
PING 192.168.102.2 (192.168.102.2) 56(84) bytes of data.
64 bytes from 192.168.102.2: icmp_req=1 ttl=63 time=1.42 ms

--- 192.168.102.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.428/1.428/1.428/0.000 ms
root@alice:~# 

```
```
root@L1:~# echo 1 > /proc/sys/net/ipv4/ip_forward
root@L1:~# ifconfig eth0 192.168.100.1/24
root@L1:~# ifconfig eth1 192.168.101.1/24
root@L1:~# ifconfig eth2 192.168.102.1/24
root@L1:~# 


```
```
Password:
Last login: Mon Mar  4 12:13:55 CET 2019 on tty0
Linux vnx 3.3.8 #1 Sun Nov 6 04:59:42 MST 2016 i686

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@bob:~# ifconfig 192.168.102.2
192.168.102.2: error fetching interface information: Device not found
root@bob:~# ifconfig eth0 192.168.102.2
root@bob:~# route add -net 192.168.100.0/24 gw 192.168.102.1
root@bob:~# route add -net 192.168.101.0/24 gw 192.168.102.1
root@bob:~# 

```

And now from alice to frank
