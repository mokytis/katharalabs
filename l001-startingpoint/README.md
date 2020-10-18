# Lab 1: Starting Point

Everything has to start somewhere. This lab is designed to introduce the basic
structure of a kathara lab.

## File Structure

The file structure of a lab is very simple.

```
.
├── a.startup
├── b.startup
├── lab.conf
└── README.md
```

### a.startup

`a.startup` does a few things. Firstly its existance tells kathara to create a
node with the name `a`. The contents of `a.startup` is then run when `a` is
started.

Lets have a look at the contents:

```
ip addr add 10.0.0.1/24 dev eth0
```

Here we are using the `ip` command (part of the `iproute2` package) to add the
ipv4 address `10.0.0.1` to the `eth0` device on the node. The `/24` at the end
tells the linux kernel that any ipv4 address starting `10.0.0.` is on the same
subnet.

### b.startup

`b.startup` does the same as `a.startup` but for a node called `b`, but instead
has the ipv4 address `10.0.0.2` assigned to its `eth0` device.

### lab.conf

`lab.conf` defines the wireing of our network.

```
a[0]=ab
b[0]=ab
```

The syntax here is quite simple. Lets break down the first line `a[0]=ab`.

* `a[0]` means the `eth0` device on the node `a`. `r[45]` would refer to the
  `eth45` device on some node called `r`.
* `=ab` 'connects' that device to a collision domain called `ab`. The name `ab`
  is completly arbitary and we could put `pizza` here if we wanted (although
  that would be bad practice. The name `ab` is good as this collsion domain
  connects nodes `a` and `b`). The line `b[0]=ab` connects `eth0` on `b` to the
  same collision domain that `eth0` on `a` is connected to allowing packets to
  be sent between them. For now you can think of this as a network hub or just a
  wire.

### README.md

Every lab in this repo will have a `README.md` file. This explains what is going
on in the lab. This file is not required by kathara.

## Running the lab

To run the lab make sure that you have
[Kathara](https://github.com/KatharaFramework/Kathara) installed and setup. Then
run `kathara lstart`.

On `b` we can run `tcpdump -i eth0`. This will allow us to see any network
traffic on the `eth0` device.

On `a` runinng `ping 10.0.0.2` should send `ICMP ECHO_REQUEST` packets to `b`.
You should see this in`b`'s `tcpdump` output.

## Stopping the lab

To end the lab run `kathara lclean` in the lab directory.

## Links

Some links and references that may be useful

* [https://en.wikipedia.org/wiki/Subnetwork](https://en.wikipedia.org/wiki/Subnetwork)
* [https://www.ipaddressguide.com/netmask](https://www.ipaddressguide.com/netmask)
* [https://linux.die.net/man/8/ip](https://linux.die.net/man/8/ip)
* [https://en.wikipedia.org/wiki/Collision_domain](https://en.wikipedia.org/wiki/Collision_domain)
* [https://en.wikipedia.org/wiki/README](https://en.wikipedia.org/wiki/README)
* [https://www.tcpdump.org/manpages/tcpdump.1.html](https://www.tcpdump.org/manpages/tcpdump.1.html)
* [https://tools.ietf.org/html/rfc792](https://tools.ietf.org/html/rfc792)
