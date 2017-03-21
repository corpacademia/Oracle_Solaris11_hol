Oracle Solaris 11 Networking Lab
================================

In Solaris 11 several new networking commands were added, some
management practices have changed to make network administration easier
and more robust. In this lab we will learn some new networking commands,
compare them to the old ones and also work with network virtualization
features, which are brand new in Solaris 11.

Exercise N.1: Solaris 11 Networking Basics
------------------------------------------

**Task:** You have to configure network interfaces and network services
(DNS) in Solaris.

**Lab:** We have configured our Solaris virtual machine initially to use
Automatic network configuration. That means that it was configured using
VirtualBox's internal DHCP server. In real life usually it's not the
case. Usually you configure your Solaris servers using manual mode. We
will learn how to do that. We will study the default IP and DNS
configuration and then use it in the manual mode. We will use a new
feature called Vanity Naming which allows you to give network interfaces
any names you want. Note that when we use these new Solaris 11 commands,
all the changes are persistent and will sustain a reboot.

We assume that you have used the 'Automatic' network option mode during
the initial system configuration for your virtual machine. You have
recieved your network configuration from the VirtualBox's internal DHCP
server. Check if you can access the Internet:

``` console
root@solaris:~# ping oracle.com
oracle.com is alive
```

If you are behind a firewall, most likely you will not be able to ping
the outside network. If this is the case, try to ping one of your
internal sites (e.g. your internal DNS server). Or, try `ping 10.0.2.2`.
It's the address of your host machine as seen from inside the VM.

Check your current configuration and record it to use in the future,
when we switch to the manual mode. Enter the following commands and
observe the results.

``` console
root@solaris:~# dladm show-link
root@solaris:~# dladm show-phys
root@solaris:~# dladm show-ether
```

What did you learn from those commands? That you have one physical
Ethernet interface, with the name `net0`, using device `e1000g0`, with
nominal speed 1Gbps. Big change in Solaris 11: all network interfaces by
default now have unified generic names like `net0`, `net1` etc. More than
that: you can even use your own names for network interfaces! More about
this later.

This is our datalink level inventory. Let's move up, on the IP level.
Enter the following commands to figure out your current IP
configuration.

``` console
root@solaris:~# ipadm 
NAME              CLASS/TYPE STATE        UNDER      ADDR
lo0               loopback   ok           --         --
lo0/v4         static     ok           --         127.0.0.1/8
lo0/v6         static     ok           --         ::1/128
net0              ip         ok           --         --
net0/v4        dhcp       ok           --         10.0.2.15/24
net0/v6        addrconf   ok           --         fe80::a00:27ff:fec0:3b0a/10
```

OK, we've got the usual loopback interface and the `net0` interface with
IP address `10.0.2.15/24` which was assigned by the DHCP server. Let's
take a more detailed look at `net0`.

``` console
root@solaris:~# ipadm show-ifprop net0
IFNAME      PROPERTY        PROTO PERM CURRENT    PERSISTENT DEFAULT    POSSIBLE
net0        arp             ipv4  rw   on         --         on         on,off
net0        forwarding      ipv4  rw   off        --         off        on,off
net0        metric          ipv4  rw   0          --         0          --
net0        mtu             ipv4  rw   1500       --         1500       68-1500
net0        exchange_routes ipv4  rw   on         --         on         on,off
net0        usesrc          ipv4  rw   none       --         none       --
net0        forwarding      ipv6  rw   off        --         off        on,off
net0        metric          ipv6  rw   0          --         0          --
net0        mtu             ipv6  rw   1500       --         1500       1280-1500
net0        nud             ipv6  rw   on         --         on         on,off
net0        exchange_routes ipv6  rw   on         --         on         on,off
net0        usesrc          ipv6  rw   none       --         none       --
net0        group           ip    rw   --         --         --         --
net0        standby         ip    rw   off        --         off        on,off
```

A lot of information about IP properties of this `net0` interface. You can
learn about these network parameters later. Consider that your homework
assignment. For now let's move on.

What about routing table and DNS settings? We will need them when
configuring our interfaces in manual mode.

``` console
root@solaris:~# netstat -nr

Routing Table: IPv4
Destination           Gateway           Flags  Ref     Use     Interface 
-------------------- -------------------- ----- ----- ---------- --------- 
default              10.0.2.2             UG        4       1778 net0      
10.0.2.0             10.0.2.15            U         3          0 net0      
127.0.0.1            127.0.0.1            UH        2        796 lo0       

Routing Table: IPv6
Destination/Mask            Gateway                   Flags Ref   Use    If   
--------------------------- --------------------------- ----- --- ------- ----- 
::1                         ::1                         UH      2       8 lo0   
fe80::/10                   fe80::a00:27ff:fec0:3b0a    U       2       0 net0  

root@solaris:~# cat /etc/resolv.conf

#
# _AUTOGENERATED_FROM_SMF_V1_
#
# WARNING: THIS FILE GENERATED FROM SMF DATA.
#   DO NOT EDIT THIS FILE.  EDITS WILL BE LOST.
# See resolv.conf(4) for details.

nameserver  192.168.1.1
```

Note the warning in the `resolv.conf` file. There are some changes in
DNS configuration in Solaris 11, we'll talk about them later. Now, just
write down your default router IP address (`10.0.2.2` in case of
VirtualBox installation) and your DNS server address(es) (most likely,
yours are different from `192.168.1.1`).

Now, we are ready to change network management to the manual mode:

``` console
root@solaris:~# netadm enable -p ncp DefaultFixed
```

Check again if you can access the Internet (again, replace `oracle.com`
with one of your internal hosts if you are behind a firewall):

``` console
root@solaris:~# ping oracle.com
ping: unknown host oracle.com
```

Most likely, the reason for this error message is that we can't access
any DNS servers or they are not configured at all. Check the DNS
server's IP address (replace `192.168.1.1` with what you have recorded
while in Automatic mode):

``` console
root@solaris:~# ping 192.168.1.1
ping: sendto No route to host
```

Routing is not configured. OK, the default gateway was `10.0.2.2`
(internal VirtualBox address). Let's try it:

``` console
root@solaris:~# ping 10.0.2.2
ping: sendto No route to host
```

Nothing works! Let's start from the beginning. Check if the same
physical links are available:

``` console
root@solaris:~# dladm show-phys
LINK              MEDIA                STATE      SPEED  DUPLEX    DEVICE
net0              Ethernet             unknown    1000   full      e1000g0
```

OK, physical link is in place. What about IP links?

``` console
root@solaris:~# ipadm 
NAME              CLASS/TYPE STATE        UNDER      ADDR
lo0               loopback   ok           --         --
lo0/v4         static     ok           --         127.0.0.1/8
lo0/v6         static     ok           --         ::1/128
```

Only loopback is available. Time to create an IP link from scratch:

``` console
root@solaris:~# ipadm create-ip net0
root@solaris:~# ipadm
NAME              CLASS/TYPE STATE        UNDER      ADDR
lo0               loopback   ok           --         --
lo0/v4         static     ok           --         127.0.0.1/8
lo0/v6         static     ok           --         ::1/128
net0              ip         down         --         --
```

IP link is there, but there is no IP address assigned to it. Let's fix
that.

``` console
root@solaris:~# ipadm create-addr -a 10.0.2.15/24 net0
net0/v4
root@solaris:~# ipadm 
NAME              CLASS/TYPE STATE        UNDER      ADDR
lo0               loopback   ok           --         --
lo0/v4         static     ok           --         127.0.0.1/8
lo0/v6         static     ok           --         ::1/128
net0              ip         ok           --         --
net0/addr      static     ok           --         10.0.2.15/24
```

Much better. Try pinging some addresses:

``` console
root@solaris:~# ping oracle.com
ping: unknown host oracle.com
root@solaris:~# ping 10.0.2.2
10.0.2.2 is alive
```

First ping failure tells us that most likely DNS is not avalable. Second
ping shows that we can at least access our default gateway. Let's
continue moving further and ping our DNS server.

Ping the network again:

``` console
root@solaris:~# ping 192.168.1.1 (replace 192.168.1.1 with your DNS server IP address)
ping: sendto No route to host
```

Routing is not configured. Check:

``` console
root@solaris:~# netstat -nr

Routing Table: IPv4
Destination           Gateway           Flags  Ref     Use     Interface 
-------------------- -------------------- ----- ----- ---------- --------- 
10.0.2.0             10.0.2.15            U         3          2 net0      
127.0.0.1            127.0.0.1            UH        2       1214 lo0       

Routing Table: IPv6
Destination/Mask            Gateway                   Flags Ref   Use    If   
--------------------------- --------------------------- ----- --- ------- ----- 
::1                         ::1                         UH      2      12 lo0   
```

Yes, indeed. There is no default gateway. Add the default gateway and
check again:

``` console
root@solaris:~# route -p add default 10.0.2.2
add net default: gateway 10.0.2.2
add persistent net default: gateway 10.0.2.2
root@solaris:~# ping 192.168.1.1 (replace 192.168.1.1 with your DNS server IP address)
192.168.1.1 is alive
root@solaris:~# ping oracle.com (replace oracle.com with your internal site)
ping: unknown host oracle.com
```

We can reach our DNS server, but our system is not configured to use it.
If you think that editing your /etc/resolv.conf is enough, remember the
warning in that file:

```
    # WARNING: THIS FILE GENERATED FROM SMF DATA.
    #   DO NOT EDIT THIS FILE.  EDITS WILL BE LOST.
```

That means that in Solaris 11 name service configuration is different
from what you used before. To use DNS we have to configure the
`dns/client` service and also the `name-service/switch` service which
used to be configured via `/etc/nsswitch.conf`. Yes, it's a little bit
more complicated, but it's more robust and manageable. It's a general
direction in Solaris: most of the services are configured via SMF
framework, not via config files. Here are the commands:

``` console
root@solaris:~# svccfg -s dns/client 'setprop config/nameserver = net_address: 192.168.1.1'
root@solaris:~# svccfg -s dns/client 'setprop config/domain = astring: "example.com" ' (replace example.com with your local default domain name or skip this step)
root@solaris:~# svccfg -s name-service/switch 'setprop config/host = astring: "files dns" '
root@solaris:~# svcadm refresh name-service/switch
root@solaris:~# svcadm refresh dns/client
```

Alternatively, you can edit the usual files `/etc/resolv.conf` and
`/etc/nsswitch.conf`, but you have to import them into the naming service
configuration:

``` console
root@solaris:~# nscfg import -f svc:/system/name-service/switch:default
root@solaris:~# nscfg import -f svc:/network/dns/client:default
root@solaris:~# svcadm refresh dns/client
```

Now our ping finally reaches the Internet:

``` console
root@solaris:~# ping oracle.com (replace oracle.com with one of your internal hosts)
oracle.com is alive
```

**New names.** Do you remember the days when you were a junior Solaris
system administrator and wondered why all network interfaces in Solaris
have these funny names? `le`, `bge`, `ce`, `xge`, `e1000g`.... Now, as you can
see, they all are called `net0`, `net1`, `net2`, ... Much simpler, right? Even
more than that: you can give your interfaces your own names. Here is the
example. Show what we've got now:

``` console
root@solaris:~# dladm
root@solaris:~# ipadm
```

Imagine we want to use our network interfaces for different services on
our Solaris box. We have web server, application server etc. We can name
our network interfaces `web1`, `app0`, `db1` etc. Start by deleting the `net0`
IP interface

``` console
root@solaris:~# ipadm delete-ip net0
```

...now rename the NIC

``` console
root@solaris:~# dladm rename-link net0 web1
root@solaris:~# dladm 
```

Add back in the IP interface and its address:

``` console
root@solaris:~# ipadm create-ip web1
root@solaris:~# ipadm create-addr -a 10.0.2.15/24 web1
```

Cleaning up... Undo it all

``` console
root@solaris:~# ipadm delete-ip web1
root@solaris:~# dladm rename-link web1 net0
root@solaris:~# ipadm create-ip net0
root@solaris:~# ipadm create-addr -a 10.0.2.15/24 net0
root@solaris:~# ipadm
```

You may need to restart your DNS client service after this exercise:

``` console
root@solaris:~# svcadm disable dns/client
root@solaris:~# svcadm enable dns/client
```

One word of advice: having this kind of freedom, please try to avoid
long discussions about network interface naming, similar to what you
have already had regarding host naming policies. :-)

Exercise N.2: Network Virtualization
------------------------------------

**Task:** You want to create Virtual Network Interface Cards (VNICs) to
use them with your Zones. You want to build and manage your
application's network infrastructure completely inside the box for
development and testing purposes.

**Lab:** We will create VNICs, assign IP addresses to them and learn how
to limit bandwidth on them.

First we show the links. Links can be physical or virtual. Note that for
physical NICs, we use a new naming scheme `net0`, `net1`, etc. that hides
the actual device name.

``` console
root@solaris:~# dladm show-link
```

Show only the physical ethernet NICs:

``` console
root@solaris:~# dladm show-ether
```

And to see the actual hardware devices used for the netX NICs:

``` console
root@solaris:~# dladm show-phys
```

The next command shows a bit more information like the physical
location:

``` console
root@solaris:~# dladm show-phys -L
```

So now we create a VNIC that we call vnic1, using `net0` as its
underlying datalink. Note that VNICs are first-class NICs in terms of
visibility (e.g. `snoop`)

``` console
root@solaris:~# dladm create-vnic -l net0 vnic1
```

Show the VNICs:

``` console
root@solaris:~# dladm show-vnic
```

We can easily limit bandwith on a VNIC:

``` console
root@solaris:~# dladm set-linkprop -p maxbw=40 vnic1
root@solaris:~# dladm show-vnic
```

Now we create an IP interface. This is analgous to plumbing the
interface:

``` console
root@solaris:~# ipadm create-ip vnic1
```

Now we assign a persistent IP address to the VNIC:

``` console
root@solaris:~# ipadm create-addr -a 10.2.3.4/24 vnic1
```

Ping the VNIC:

``` console
root@solaris:~# ping 10.2.3.4
```

Show all available datalinks, both physical and virtual

``` console
root@solaris:~# dladm show-link
```

Finally list all IP addresses:

``` console
root@solaris:~# ipadm show-addr
```

Now we tear down what we've just created:

``` console
root@solaris:~# ipadm delete-addr vnic1/v4
root@solaris:~# ipadm delete-ip vnic1
root@solaris:~# dladm delete-vnic vnic1
root@solaris:~# dladm show-link
```

Now you see how new networking commands work. Of course, you can still
use the old-style `ifconfig`, but the new commands are easier to use
and, most importantly, they make presistent changes.

Find more food for thought and inspiration here:

-   Strategies for Network Administration in Oracle Solaris 11.2  
    http://docs.oracle.com/cd/E36784_01/html/E37473/index.html  
-   How to get started configuring your network in Oracle Solaris 11  
    http://www.oracle.com/technetwork/articles/servers-storage-dev/s11-network-config-1632927.html  
-   How to Script Oracle Solaris 11 Zones Creation for a Network-In-a-Box Configuration  
    http://www.oracle.com/technetwork/articles/servers-storage-admin/o11-118-s11-script-zones-524499.html  
-   How to Restrict Your Application Traffic Using Oracle Solaris 11  
    Network Virtualization and Resource Management  
    http://www.oracle.com/technetwork/articles/servers-storage-admin/o11-095-s11-app-traffic-525038.html

