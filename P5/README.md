# Practice 5 memo of TCGI
## Exercise 1
After checking with `netstat` that the daemon it is not running, we start it.
We can see on the `dhcp` configuration file that:

```
subnet 10.0.0.0 netmask 255.255.255.0 {
  range 10.0.0.50 10.0.0.60;
```

All the machins will be given an @IP between the 10.0.0.50 and the 10.0.0.60.

## Exercise 2
After setting up the `DHCP` server on Joker's machine, we can now try to run
the client on Alice by typing `dhclient3 eth1`. After capturing almost two
minutes of traffic, this are the results:

![Image1](./images/img1.png)

* It all begins with a `DHCP-DISCOVER` from Alice's machine to broadcast. Then,
	Joker, checks if the `@IP` that wants to assign to Alice is in use by doing
	an ARP request. 
* Only because no one replies the ARP, he now sends a `DHCP-OFFER` to Alice
	with all the configuration including the lease time (1m10s).
* Then Alice replies with a `DHCP-REQUEST` containing the same information as
	the offer to which the server replies with a `DHCP-ACK` containing all the
	information and confirming that the sent configuration is ok.

Now the assigned `@IP` of alice is `10.0.0.50` and it is not accessible through
its domain name because its `@IP` has changed but has not been updated in the
DNS.

If we now explore the file `/var/lib/dhcp3/dhclient.leases` on Alices machine
we can see all the leases she has been granted from joker, even the expired
ones:

```
lease {
  interface "eth1";
  fixed-address 10.0.0.50;
  option subnet-mask 255.255.255.0;
  option dhcp-lease-time 70;
  option dhcp-message-type 5;
  option domain-name-servers 10.0.0.21;
  option dhcp-server-identifier 10.0.0.201;
  option domain-name "example.com";
  renew 2 2019/04/02 15:57:00;
  rebind 2 2019/04/02 15:57:32;
  expire 2 2019/04/02 15:57:41;
}
```

* The renewal timer, is set by default to 50% of the length of the lease.
	When the timer goes off, the client transitions from the BOUND state to the
	RENEWING state.
* The rebin timer is te to 7/8 of the length of the lease. Once it has expired
	and the lease has not been renewed, it will try to bind to any other active
	`DHCP` server with the current `@IP`

## Exercise 3
After having executed the client in Alice for a while, we executed `dhclient -r
eth1` whitch will trigger an `@IP` release. And it sure does:

![Image2](./images/img2.png)

We can see that Alice specifies her old `@IP`. An now, if we run `ip a` on her
machine, she does not have any `@IP`

## Exercise 4
Now with manual allocation, we can see taht Alice is getting the `@IP`
configurd in the `DNS`. There is no `DNS` resolution of the name, I suspect it
has been store on Joker's machine but I don't know where. The is still a lease
time because no other specific parameter has been setup on the `DHCP` only for
Alice.
