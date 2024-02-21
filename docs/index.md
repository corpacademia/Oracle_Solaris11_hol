<img alt="Oracle Solaris 11" src="img/O_Solaris_11_clr.gif" style="float:
right;" >

In this lab we introduce the most interesting features of Oracle Solaris 11,
based on real life use cases. We will:

-   create a disk pool and a file system with ZFS; expand it; snapshot and
    clone it; use ZFS compression and deduplication; use ZFS migration for
    backup: [ZFS Lab](zfs/zfs.md)
-   create a new boot environment as a backup, make our current system
    unbootable because of some fatal mistakes, reboot the system using
    backup BE: [Boot Environments Lab](be/be.md)
-   use new packaging system called IPS to search, install, verify and
    fix packages; update the system using IPS and Boot Environments
    together: [IPS Lab](ips/ips.md)
-   use new networking commands, configure virtual network interfaces: [Networking Lab](net/net.md)
-   create a couple of Solaris virtual environments (zones); install some
    applications into them; clone a zone; use Resource Management with the zones: [Virtualization Lab](virt/virt_pre.md)
-   learn how to install systems and zones with Automated Installer: [AI Lab](ai/ai_intro.md)
-   compare two virtualization options available in Oracle Solaris: non-global
    zones and kernel zones: [Kernel Zones Lab](kz/kz.md) 

Prerequisites
-------------

This lab requires access corpacademia Virtial Lab environment. Please get the Access details from your Instructor

    
The Environment
---------------

In this lab we are going to use Oracle Solaris 11 virtual appliance in
Oracle VirtualBox environment. If you are using lab machines, the
appliance is already installed. 

By default, VirtualBox assigns the IP address `10.0.2.15` to the Solaris
global zone. We will be using also IP addresses `10.0.2.21` and `10.0.2.22`
for local zones. As we are using VirtualBox in NAT (network address
translation) mode, this shouldn't interfere with your outside network
environment.

You should login into Solaris desktop with the following credentials:

Username: `lab` Password: `oracle1`

After logging in, open a terminal window and assume the `root` role:

``` console
lab@solaris:~$ su - 
```

Password for root is: `solaris1`.

Note: we don't recommend to log in as `root`. In Oracle Solaris 11 it is
prohibited by default; `root` is only a role, not a login name.

Putting It All Together
-----------------------

The whole idea of this lab is to show you some Oracle Solaris 11 features that
can be used to create a cloud infrastructure based on Solaris. You have
just created storage pools and filesystems&mdash;think cloud storage. It
was fast, it was simple, it was flexible. You have created and cloned
Solaris zones with applications within them&mdash;think _cloud machine
instances_. You have monitored and managed zones resources&mdash;think _cloud
elasticity, metering and chargeback_. Add to that 
Solaris _network virtualization_, Solaris _security_, Solaris _software lifecycle
management_ and many other features which make Oracle Solaris 11 truly
cloud-oriented operating system. Try them and learn more about Solaris 11!

Final Notes
-----------

The virtual appliance we used in this lab is configured to be able to
perform zone installation without network access. Namely, we've
configured an internal repository with just a small subset of packages
necessary for zone installations. If you are going to continue using
this appliance with open network access, you will need to change the
repository address to Oracle's standard Solaris repository.

``` console
root@solaris:~# pkg set-publisher -G '*' -M '*' -g http://pkg.oracle.com/solaris/release -P solaris 
```


