### Zone creation

Non-global zone:

``` console
root@solarislab:~# zonecfg -z zone1 create
```

Kernel zone:

``` console
root@solarislab:~# zonecfg -z kzone1 create -t SYSsolaris-kz
```

That's it! We just use a special `kernel zone` template to create a
zone. Actually, in the first command we use the default `SYSsolaris`
profile as soon as we omitted that parameter. If you are curious, take a
look at the `/etc/zones` directory to compare different profiles. You
will also find that there is a special profile for Solaris 10 zones.

### Default configuration

Non-global zone:

``` console
root@solarislab:/etc/zones# zonecfg -z zone1 info
zonename: zone1
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
        bwshare not specified
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

Kernel zone:

``` console
root@solarislab:/etc/zones# zonecfg -z kzone1 info
zonename: kzone1
brand: solaris-kz
autoboot: false
autoshutdown: shutdown
bootargs:
pool:
scheduling-class:
hostid: 0x2e7d2173
tenant:
anet:
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
        bwshare not specified
        rxfanout not specified
        vsi-typeid not specified
        vsi-vers not specified
        vsi-mgrid not specified
        etsbw-lcl not specified
        cos not specified
        evs not specified
        vport not specified
        iov: off
        lro: auto
        id: 0
device:
        match not specified
        storage: dev:/dev/zvol/dsk/rpool/VARSHARE/zones/kzone1/disk0
        id: 0
        bootpri: 0
capped-memory:
        physical: 4G
```

Here we see more difference between the two. First, we notice that the
kernel zone has a different *brand*. It's a signal to the Solaris kernel to
treat this zone differently from the default `solaris` brand. Also we
see that the `zonepath` parameter disappeared in the kernel zone. Where are
we going to store the zone's root? Scroll down a little bit and find the
`device:` section. What do you see? Now you see that kernel zones keep
their root directories not in a ZFS *file system*, but rather in a ZFS
*volume* which looks like a block device. By default, Solaris creates a
16 Gigabyte volume for that in the `rpool` ZFS pool. Of course, you
can change the size of the volume during installation.

Also take a look at the `zoneadm list -cv` output:

``` console
root@solarislab:~# zoneadm list -cv
  ID NAME             STATUS      PATH                         BRAND      IP
   0 global           running     /                            solaris    shared
   - zone1            configured  /system/zones/zone1          solaris    excl
   - kzone1           configured  -                            solaris-kz excl
```

Again, you see that we don't have a file system path specified for the
kernel zone.

### Zone installation

Now it's time to install both zones. We use exactly the same command for
both kernel zones and non-global zones. Start with the kernel zone:

``` console
root@solarislab:~# zoneadm -z kzone1 install
Progress being logged to /var/log/zones/zoneadm.20150615T163032Z.kzone1.install
pkg cache: Using /var/pkg/publisher.
 Install Log: /system/volatile/install.17385/install_log
 AI Manifest: /tmp/zoneadm16918.sKa4xH/devel-ai-manifest.xml
  SC Profile: /usr/share/auto_install/sc_profiles/enable_sci.xml
Installation: Starting ...

        Creating IPS image
        Installing packages from:
            solaris
                origin:  http://ipkg.us.oracle.com/solaris11/support/
        The following licenses have been accepted and not displayed.
        Please review the licenses for the following packages post-install:
          consolidation/osnet/osnet-incorporation
        Package licenses may be viewed using the command:
          pkg info --license 

DOWNLOAD                                PKGS         FILES    XFER (MB)   SPEED
Completed                            451/451   63995/63995  598.3/598.3  2.2M/s

PHASE                                          ITEMS
Installing new actions                   87551/87551
Updating package state database                 Done
Updating package cache                           0/0
Updating image state                            Done
Creating fast lookup database                   Done
Installation: Succeeded
        Done: Installation completed in 716.730 seconds.
```

What if we want to install a different version of Solaris into the
kernel zone? In the previous example we used the so called "direct
installation" method. In other words, we used the same package
repository that is configured in the global zone. To install a different
version of Solaris, we have to use a separate installation media. For
example, we can use Oracle Solaris 11.3 beta DVD ISO for that:

``` console
root@solarislab:~# zoneadm -z kzone1 install -b /share1/ISOs/sol-11_3-25-text-sparc.iso
```

What do you think is going to happen in this case? You guessed it right:
the kernel zone will boot from this DVD and the usual installation
process will begin. You will go through the familiar questions:
hostname, IP address, time zone, root password, etc. The process is no
different from bare metal or logical domain Solaris installation.

And now install the non-global zone:

``` console
root@solarislab:~# zoneadm -z zone1 install
The following ZFS file system(s) have been created:
    rpool/VARSHARE/zones/zone1
Progress being logged to /var/log/zones/zoneadm.20150615T165123Z.zone1.install
       Image: Preparing at /system/zones/zone1/root.

 Install Log: /system/volatile/install.18535/install_log
 AI Manifest: /tmp/manifest.xml.Z.aylK
  SC Profile: /usr/share/auto_install/sc_profiles/enable_sci.xml
    Zonename: zone1
Installation: Starting ...

        Creating IPS image
Startup linked: 1/1 done
        Installing packages from:
            solaris
                origin:  http://ipkg.us.oracle.com/solaris11/support/
DOWNLOAD                                PKGS         FILES    XFER (MB)   SPEED
Completed                            280/280   53151/53151  374.3/374.3  3.4M/s

PHASE                                          ITEMS
Installing new actions                   71074/71074
Updating package state database                 Done
Updating package cache                           0/0
Updating image state                            Done
Creating fast lookup database                   Done
Updating package cache                           1/1
Installation: Succeeded

        Note: Man pages can be obtained by installing pkg:/system/manual

 done.

        Done: Installation completed in 485.377 seconds.


  Next Steps: Boot the zone, then log into the zone console (zlogin -C)

              to complete the configuration process.

Log saved in non-global zone as /system/zones/zone1/root/var/log/zones/zoneadm.20150615T165123Z.zone1.install
```

Let's look at what's different between these two listings. First, in the
kernel zone we install more packages (451 vs. 280). Also, if you look at
the progress line during the installation you may notice that packages
like hardware drivers are being installed--that's a big difference from
non-global zones.

Now we can boot both zones and complete the installation by configuring
host names, root passwords, time zones, etc. (in case of installing the
kernel zone from media, we have done that already). There is almost no
difference in these processes, they are all very familiar. To do that
login into each zone's console and fill the screens that follow.

``` console
root@solarislab:~# zlogin -C zone1
```

    and then:

``` console
root@solarislab:~# zlogin -C kzone1
```

All the necessary information will be provided by your instructor. Make
sure you choose **Manual** network configuration, not **Automatic**, which
is the default. You will need: IP address, netmask, default router
address. Set the time zone and root password to the same values as in
the global zone.

