# Practice 1 memo of TCGI

## Exercice 1
First of all, to inicialize the simulation, run:
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
Notice that
