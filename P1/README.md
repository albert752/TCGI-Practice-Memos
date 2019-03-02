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
	
### Section 2
To reproduce the descrived scenario, we have to run two servers on Alice and
then two clients, one on Bob and the other on Carla. Other configurations fail
to work because is the client who starts the comunication. If we run
multiple cients in one machine, once we send a message using one of them, we
lose the hability to interface with the others. 

We could also specify diferent SAPs for each server but it wouldi yield
problems that I don't know how to solve yet

### Section 3
First of all we run `brctl show` on L1 and we get this output:

```
bridge name     bridge id               STP enabled     interfaces
br1             8000.fefd00000700       no              eth0
                                                        eth1
                                                        eth2
```
As we can see the spanning tree protocol is not enabled (we don't have any loop
topolgy so it is not needed). The interfaces `eth0`, `eth1` and `eth2` are
assigned to the br1 as shown on the instructions diagram. The same happens with
the other switches, L2 and L3.


## Author

* **Albert Azemar i Rovira** - *Initial work* -
	[albert752](https://github.com/albert752)

## License

This project is licensed under the MIT License - see the
[LICENSE.md](../LICENSE.md) file for details

