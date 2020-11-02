# Simple Switch

This lab walks you through setting up a network switch using the `iproute2`
package. We will have 3 nodes which all will be connected to different network
cards on a 4th node.

A network switch (sometimes called a MAC bridge) operates on layer 2 of the OSI
model. It keeps a record of which MAC addresses can be accessed via each of its
network cards and forwards on packets accordingly.

## Nodes

### Overview

In this lab we have 4 nodes:

* sw0
    - a layer 2 network switch
    - it has 3 network cards (links)
        - `eth0` is connected to collision domain `sw0port0`
        - `eth1` is connected to collision domain `sw0port1`
        - `eth2` is connected to collision domain `sw0port2`
    - the links are connected to a bridge called `br0`
* h1
    - a host
    - `eth0` has ipv4 address `10.0.0.1`
    - `eth0` is connected to collision domain `sw0port0`
* h2
    - a host
    - `eth0` has ipv4 address `10.0.0.2`
    - `eth0` is connected to collision domain `sw0port1`
* h3
    - a host
    - `eth0` has ipv4 address `10.0.0.3`
    - `eth0` is connected to collision domain `sw0port2`

### Connectivity Diagram

```
 node  bridge  link   collision domain   link  node
+-----+------+------+                  +------+----+
| sw0 | br0  | eth0 |-----sw0port0-----| eth0 | h1 |
|     |      |      |                  +------+----+
|     |      +------+                  +------+----+
|     |      | eth1 |-----sw0port1-----| eth0 | h2 |
|     |      |      |                  +------+----+
|     |      +------+                  +------+----+
|     |      | eth2 |-----sw0port2-----| eth0 | h3 |
|     |      |      |                  +------+----+
+-----+------+------+
```

### sw0

We are going to use the `iproute2` package to make sw0 function as a network
switch. This is done by creating a bridge and then adding network cards (links)
to it.

To do this we first make sure that the state of each link we want to be apart of
the switch is up.

```
ip link set eth0 up
ip link set eth1 up
ip link set eth2 up
```

We now have to create the bridge and set it as the master for each link.

```
ip link add name br0 type bridge


ip link set eth0 master br0
ip link set eth1 master br0
ip link set eth2 master br0
```

Finally we set the state of the bridge to up.

```
ip link set br0 up
```

Now when packets are recieved from a link that is apart of the bridge, the
bridge will forward the packet onto the link that it knows a device with the
packet's destination MAC address is connected to. We will be using `tcpdump` and
the `bridge` utitly to capture the packets being sent to observe how it does
this.

### h1, h2, h3

For each host we are simply going to asign a static ipv4 address to the eth0
link.

```
# h1.startup
ip addr add 10.0.0.1/24 dev eth0

# h2.startup
ip addr add 10.0.0.2/24 dev eth0

# h3.startup
ip addr add 10.0.0.3/24 dev eth0

```

## Running the lab

As normal, start the lab with `kathara lstart`. This will give you access to a
shell on each node, however it would be useful to have a second shell on sw0 to
so you can monitor a couple of things.

### Getting a second shell

It is very easy to get a second shell on a node. Inside the lab directory run
`kathara connect <nodename>`. In this case a second shell on sw0 is wanted so run
`kathara connect sw0`.

### Lets monitor what's going on

To be able to see what is going on you can use a few different commands.
`bridge` (which is apart of the `iproute2` package) is a useful utility for
interacting with the network bridge we have setup. We can also use tcpdump to
view packets being sent across a given interface.

On one shell for sw0 run `bridge monitor` and on the other run `tcpdump -i br0
`.

On h2 and h3 run `tcpdump -i eth0`.

Now from h1 run `ping 10.0.0.2` to send ICMP ping requests to h2. You should see
an ARP Request being sent to both h2 an h3. You should also notice some output
to the `bridge monitor` command. It is showing that it now knows that h1's eth0
MAC address is reachable via it's own eth0 link that is part of br0.

h2 sends an ARP Reply informing h1 of it's MAC address. You will then see a
series of ICMP echo requests and ICMP echo reply packets being sent between h1
and h2. h3 shouln't be reciving these as sw0 is using it's br0 bridge to only
forward the packets to the correct recipiant.

To work out what is going on here I recommend researching the Address
Resolution Protocol (ARP) and Internet Control Message Protocol (ICMP) ping.
