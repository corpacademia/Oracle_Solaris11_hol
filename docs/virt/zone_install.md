**Task:** Your development team wants a separate environment to develop
and test their new application.

**Lab:** We are going to use Solaris virtualization technology called
Solaris Zones.


!!! note "60 seconds of theory" 
    Solaris Zones are isolated operating environments,
    which run inside a single Solaris instance. Each Solaris Zone has its
    own users, set of processes and applications, hostname and IP address.
    There are two types of zones: kernel zones and non-kernel zones. In this
    lab we will be working with non-kernel zones. They use the same kernel
    that is used by the host Solaris instance, also called "global zone".
    Each zone has it's own dataset ('zone root') where all system files are
    installed. Zones can use additional datasets for applications and users'
    data. By default in Solaris 11 zones use virtual network interface cards
    (VNICs) and "exclusive IP" which means that networking can be managed
    both from inside and outside the zone.


When creating a zone we have to define two most important parameters:
the zone root's location and the network configuration. In the simplest
possible case both of them can be left to their default values. Zone
root by default will be located in `/system/zones/zonename`. Of course,
you can change that, but in this lab we'll leave it at default.
Networking by default is configured as an "automatic VNIC" which is
created automatically at the zone's boot time and destroyed after zone
halts. By default the zone's IP address is defined inside the zone and
quite possibly by a different system administrator. In most datacenter
situations though, you would like to control yourself the IP addresses
assigned to your zones. This is what we are going to do in this lab.

Start with this simple command:

``` console
root@solaris:~# zonecfg -z zone1 
zone1: No such zone configured 
Use 'create' to begin configuring a new zone. 
OK, if you say so...
zonecfg:zone1> create 
create: Using system default template 'SYSdefault'
```

Let's look at what can be configured and what is already configured:

``` 
zonecfg:zone1> info
zonename: zone1
zonepath.template: /system/zones/%{zonename}
zonepath: /system/zones/zone1
brand: solaris
autoboot: false
autoshutdown: shutdown
bootargs: 
file-mac-profile: 
pool: 
limitpriv: 
scheduling-class: 
ip-type: exclusive
hostid: 
tenant: 
fs-allowed: 
anet:
      linkname: net0
      lower-link: auto
      allowed-address not specified
      configure-allowed-address: true
      defrouter not specified
      allowed-dhcp-cids not specified
      link-protection: mac-nospoof
      mac-address: auto
      mac-prefix not specified
      mac-slot not specified
      vlan-id not specified
      priority not specified
      rxrings not specified
      txrings not specified
      mtu not specified
      maxbw not specified
      rxfanout not specified
      vsi-typeid not specified
      vsi-vers not specified
      vsi-mgrid not specified
      etsbw-lcl not specified
      cos not specified
      pkey not specified
      linkmode not specified
      evs not specified
      vport not specified
```

As you can see, `zonepath` is configured by default and one networking
interface, `anet`, is already there. To make sure the zone's IP address
is configured properly, we'll define it here, in `zonecfg`, along with the
default router. In that case we can be sure that it can't be changed
from inside the zone (maliciously or by mistake).

``` 
zonecfg:zone1> select anet linkname=net0
(In spite of having only one anet, we still have to specify which one we select for configuration) 
zonecfg:zone1:anet> set allowed-address=10.0.2.21/24 
(Use the IP address assigned by your instructor)
zonecfg:zone1:anet> set defrouter=10.0.2.2 
(Your instructor will give you the default gateway address)
zonecfg:zone1:anet> end
zonecfg:zone1> exit
```

To check the status of our newly created zone:

``` console
root@solaris:~# zoneadm list -cv 
ID NAME             STATUS     PATH                           BRAND    IP    
0 global           running    /                              solaris  shared
- zone1            configured /zones/zone1                   solaris  excl  
``` 


The zone is configured, we can install and boot it right now. But before
the installation we'll configure a profile for the Solaris instance
which will be running inside the zone. By doing that we are avoiding
configuring the zone interactively during the first boot. Our zone will
be ready for use immediately after start.

``` console
root@solaris:~# sysconfig create-profile -o /root/zone1-profile
```

This command will bring you to the interactive dialog very similar to
the standard Solaris installaion. Use <kbd>F2</kbd> to confirm your choices and
move from screen to screen. If <kbd>F2</kbd> doesn't work for you, 
use <kbd>Esc</kbd>-<kbd>2</kbd> (press
and release <kbd>Esc</kbd> and then <kbd>2</kbd>). You will have to enter:

-   Computer Name (hostname for the zone): `zone1`
-   Network configuration: choose **Automatically**
-   Time zone: choose your time zone from the list
-   Date: confirm the current date
-   Root password: `solaris1`
-   New user account details: real name, login name and password. This
    will be the first user of the zone. We have entered `Zone User`,
    `zuser`, `oracle1`
-   Other options leave to defaults

Now, when the zone's profile is created, we can install the zone and
initialize it using this profile.

``` console
root@solaris:~# zoneadm -z zone1 install -c /root/zone1-profile 
A ZFS file system has been created for this zone.
Progress being logged to /var/log/zones/zoneadm.20111113T200358Z.zone1.install
   Image: Preparing at /zones/zone1/root.

Install Log: /system/volatile/install.4418/install_log
AI Manifest: /tmp/manifest.xml.NVaaNi
SC Profile: /root/zone1-profile.xml
Zonename: zone1
Installation: Starting ...
```

Here you can take a break. The installation will take about 8-10
minutes, depending on your computer.

``` console
...Long output is skipped... 
Next Steps: Boot the zone, then log into the zone console (zlogin -C) 
to complete the configuration process. 
```

Check the status again:

``` console
root@solaris:~# zoneadm list -cv 
ID NAME             STATUS     PATH                           BRAND    IP    
0 global           running    /                              solaris  shared
- zone1            installed  /zones/zone1                   solaris  excl  
```

It's time to boot our zone:

``` console
root@solaris:~# zoneadm -z zone1 boot 
root@solaris:~# zoneadm list -cv
ID NAME             STATUS     PATH                           BRAND    IP    
0 global           running    /                              solaris  shared
1 zone1            running    /zones/zone1                   solaris  excl  
```

Note the zone's status has changed to `running`.

Now log into our zone's console (note `-C`). You will have to wait a
couple of minutes while the system is initializing services for the
first time. While waiting for the zone to boot completely, you can open
another terminal window, become root (`su -`) and login into the zone
directly with `zlogin zone1` This way you don't have to wait for all the
services to start, but you can watch the booting process in real time.
Run `prstat` and watch various system services starting one after
another.

``` console
root@solaris:~# zlogin -C zone1 
[Connected to zone 'zone1' console] 
```

You will get the standard Solaris login prompt (you might need to press
<kbd>Enter</kbd> one more time). Congratulations! You've just configured
"virtualization within virtualization" using Oracle technologies:
Solaris zones within Oracle VirtualBox (or within OVM for SPARC a.k.a.
Logical Domains).

Try to login using `root`'s credentials (`root/solaris1`). Here is the
result:

``` console
zone1 console login: root
Password: 
Roles can not login directly
Login incorrect
Nov 13 15:23:07 zone1 login: login account failure: Permission denied
```

A-ha! This is a new Solaris 11 security feature called "root as a role".
That means that you can't login into a system as `root`. You have to use
normal user's credentials and only then you will be able to use `sudo`
or `pfexec` according to your roles and privileges.

Try to login again with `zuser/oracle1`.

``` console
Oracle Corporation      SunOS 5.11      11.1    September 2012
zuser@zone1:~$
```

Success!

Note: to escape from the zone's console first type `exit` to close the
session and then at the console prompt use: <kbd>~</kbd> <kbd>.</kbd> (tilde period).
