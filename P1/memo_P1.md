# Practice 1 memo of TCGI

## Exercice 1
### Section 1
First of all, to initialize the simulation, run:
```
simctl switching-vlan start
```

Now we need to retrieve Alices and Bobs machines. To do so, run:
```
simctl switching-vlan get alice
simctl switching-vlan get bob
```
***Note:*** _The default login is root and xxxx as password._

Once we retrieve both terminals, we need to create a chat between both hosts.
We do so by running the server on Alices machine and the client on Bobs. Here
are the used commands:

On Alice:
```
server-chat-LLC1.py
```
Notice that the default parameters are `eth 0` and the SAP `0x88`.

And on Bob:
```
clinet-chat-LLC1.py fe:fd:00:00:01:00 
```

Then, initialize Wireshak by typing `wireshark` on the physical machine.

After exchanging some messages between Bob an Alice, we proceed to analize what
Wireshark has captured:

![Image 1](./images/scrot_1.png)

From inside to outside of the frame:

* On the data field of the LLC frame, we can find what could be the ASCII
	representation of the message _How are you?_ .
* Then, if we take a look on the LLC header: 
	* We can determine that is using the LLC1 protocol by taking a look on 
		the Control field. As it can be seen, it has the value `0x03` which determines LLC
		1.
	* If we now focus on the first two parameters: DSAP and SSAP, we can
		confirm taht the default settings have been applied.
* Now we switch to the Ethernet header as the LLC1 frame is just the data
	parameter of the frame:
	* The first two fields are for the @MAC. By using the command `ifconfig` on
		Alice and Bob, we can confirm they are right.
	* Due to the fact that we are using the LLC protocol, the L/P field will be
		populated by the length of the LLC frame. If we add the length of the
		message (12 bytes) plus the Control field and both SAPs, it results to
		a value of 15, the same as the one on the parameter length.

## Author

* **Albert Azemar i Rovira** - *Initial work* -
	[albert752](https://github.com/albert752)

## License

This project is licensed under the MIT License - see the
[LICENSE.md](../LICENSE.md) file for details

