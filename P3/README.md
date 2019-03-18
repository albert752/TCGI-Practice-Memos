<!--
To include in cheatsheet
* netcat command information
* Where config files are
* Exercice 4
-->

# Practice 1 memo of TCGI

## Exercice 1
By running `cat /etc/services | grep daytime` we get a similar output to this
one on each machine:

```
daytime         13/tcp
daytime         13/udp
```
We can conclude that the service daytime is running, for both L4 protocols, on
port 13. In a windows machine we supose that it could be runnin on the same
port due to the universal nature of L4 protocols.

Provided taht the @MAC is located on the same line as the interface id on
`ifconfi`, we run `ifconfig | grep HWaddr` and got the following results:

**On Virt0**
```
eth0      Link encap:Ethernet  HWaddr fe:fd:00:00:01:00  
```

**On Virt1**
```
eth0      Link encap:Ethernet  HWaddr fe:fd:00:00:02:00  
```
A similar aproach is taken for the @IP, we run `ifconfig | grep "inet a"`

The `lo` interface is is used for aplications that do not leave the localhost.
It does not have a HWaddr because of its nature. The filosophy behind `loopback` is to avoid 
using the lower levels (L2 and L1) so the frame never leaves the machine.

## Exercice 2
### Section 1
By running the `netstat -tnlp` comand on `virt1`, we get the following output:

```
root@virt1:~# netstat -tnlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
PID/Program name
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN
641/portmap
tcp        0      0 0.0.0.0:113             0.0.0.0:*               LISTEN
1435/inetd
tcp        0      0 0.0.0.0:21              0.0.0.0:*               LISTEN
1435/inetd
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
1247/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN
1195/exim4
tcp        0      0 0.0.0.0:40605           0.0.0.0:*               LISTEN
653/rpc.statd
tcp6       0      0 :::80                   :::*                    LISTEN
828/apache2
tcp6       0      0 :::22                   :::*                    LISTEN
1247/sshd
tcp6       0      0 ::1:25                  :::*                    LISTEN
1195/exim4

```

After taking a look on the `inetd` configuration file, we can see that all of
its lines except from two are commented. Those two are dedicated to ftp on TCP
they are listed on `netsat -tnlp` under ports 21 and 22. 

### Section 2
Now, to start the `daytime` service, we can uncomment the line from the
`/etc/inetd.conf`. After editing the file, we run `service openbsd-inetd stop`
and then `service openbsd-inetd start` to make sure the restart was done
properly.

Now we can see the `daytime` service listed under the `netsat -tnlp` command.
So from `virt2` we run `nc 10.1.1.1 13` and we get the following output:

```
Sun Mar 17 17:52:10 2019
```
Which is indeed what we expected.

### Section 3
To check if the `ssh` daemon is listening to port 22, we run `netcat -tnlp` and we get
the following output :

```
root@virt1:~# netstat -tnlp | grep ssh
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN		1247/sshd       
tcp6       0      0 :::22                   :::*                    LISTEN		1247/sshd 
```

We can now asure that `sshd` is listening on `virt1`. To stop it, we run this
command `service ssh stop`. And now it does not get listed.

### Section 4
In order to change the `sshd` port, we edit the `sshd` configuration file located at `/etc/ssh/sshd_config` file and change
the `sshd` port vale from `22` to `2222`. The changes take efect once we
restart the daemon. After checking that it is working as intended, we restore
to default and restart.

## Exercice 3
### Section 1
The command `nc -l -p 12345` creates a `TCP` socket that listens to the port
`1234` so it is a server indeed. To see what port is in use by `nc`, on `tty2`
we run `nc -l -p 12345` and get the following output:

```
root@virt1:~# netstat -tnpl | grep nc
tcp        0      0 0.0.0.0:12345           0.0.0.0:*               LISTEN
1613/nc         
tcp6       0      0 :::12345                :::*                    LISTEN
1613/nc   
```
Apart form the port, we also get the `PID` which is `1613`. So now we can run
`lsof -a -p 1613 -d0-10` (-a means AND the following flags, -p the `PID` and -d
the entries delimiations). The results are the following:

```

t@virt1:~# lsof -a -p 1613 -d0-10
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nc      1613 root    0u   CHR    4,0      0t0  362 /dev/tty0
nc      1613 root    1u   CHR    4,0      0t0  362 /dev/tty0
nc      1613 root    2u   CHR    4,0      0t0  362 /dev/tty0
nc      1613 root    3u  IPv6   3235      0t0  TCP *:12345 (LISTEN)
nc      1613 root    4u  IPv4   3236      0t0  TCP *:12345 (LISTEN)
```

As it can be seen on the last entry, there is a `TCP` file descriptor
listening to the port `12345`.

To determine the open files in the client `netcat` process, we run `netstat -tnp
| grep nc` and saw that the local port was `38729`. By retrieveing the `PID`,
we once again run `lsof -a -p1214 -d0-10` and this was the output:
```
root@virt2:~# lsof -a -p 1240 -d0-10
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nc      1240 root    0u   CHR    4,1      0t0  513 /dev/tty1
nc      1240 root    1u   CHR    4,1      0t0  513 /dev/tty1
nc      1240 root    2u   CHR    4,1      0t0  513 /dev/tty1
nc      1240 root    3u  IPv4   1766      0t0  TCP 10.1.1.2:38729->10.1.1.1:12345 (ESTABLISHED)
nc      1240 root    4u   CHR    4,1      0t0  513 /dev/tty1
nc      1240 root    5u   CHR    4,1      0t0  513 /dev/tty1
```

The fourth one is the `nc` `TPC` file descriptor.

If we now analyze the captured tarfic we can see that:
* The connenction is started by `virt2` with a `TCP` segment of `SYN`.
* Then `virt1` sends its reply with its `SYN` and `ACK`. Now the connection is
	stablished.
* Then, if we send a string from `virt1` to `virt2`, we capture the message and
	its `ACK`. By checking the different parameters (using `follow TCP stream`) we can conclude that:
	* The first message is sent by the cleint to the server with message
		`helloo`
	* The second one goes the other way arrount with payload `.[A.[Ahhhhhhh]]`.
	* We can see taht every message has its `ACK`.

The `follow TCP stream` option allows us to see the decoded messages (ASCII)
and the conversation as a whole or filtered by user.

![Image1](./images/img1.png)

### Section 2
To set up the required scenario, we run `cat /etc/services | nc -l -p 23456 -q0` on `virt1`. 
To retrieve the file on `virt2` we run `nc 10.1.1.1 23456 > file.txt`. Now we
analyze the captured packets using `follow TCP Stream`:

![Image2](./images/img2.png)

As it can be seen, the information is sent in plan text in multiple frames of
maximum lenght of 1514 bytes. The codification is `ANSI`. 

### Section 3
Now we set up the `UDP` adding the `-u` flag to the before mentioned commands.
As it can be seen on the following image is:
* Now by using `UDP` no `ACK` are sent.
* The `UDP` protocol uses the maximum bandwith all the time.
* There is an issue at the end, probably due to the client trying to reache the
	server while closed.

### Section 4
In orfer to set up the descrived scenario on the physical host, we run on one
terminal `date | nc -l -p 12345` and in another one `nc 127.0.0.1 12345`.

In order to capture the traffic, we sniff at the `loopback` interface. The date
is correcte due to the fact the my physical host has been set up correctly.

### Section 5
By running a similar command with `df -h` the service works as expected.

## Exercice 4
The steps are the following:
* First create the script and give it executions permision
* Then edit `/etc/inetd.conf` to add the line `space stream tcp nowait root
	/root/space.sh`
* We assign the `22233` port to the service by editing `/etc/services`
* Now we manualy restart the deamon.

We can now check if it is workig by running `nc 127.0.0.1 22233`. To debug, run ` tail -f /var/log/daemon.log`

## Exercice 5


## Issues
* **E1:** On windows running on the same port.
* ~~**E2S1:** Why ftp is not listed?~~ Use `netstat -tnlp`.
* **E3S1:** More info? Also on S2.
