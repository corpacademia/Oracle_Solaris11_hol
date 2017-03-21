**Task:** Your development team wants a copy of this environment for
testing purposes.

**Lab:** We will configure a new zone ('`zone2`') and then clone it from
the existing `zone1`.

First, configure `zone2` the same way you've configured `zone1`:

``` console
root@solaris:~# zonecfg -z zone2 
zone2: No such zone configured 
Use 'create' to begin configuring a new zone. 
zonecfg:zone2> create 
create: Using system default template 'SYSdefault'
zonecfg:zone2> select anet linkname=net0
(In spite of having only one anet, we still have to specify which one we select for configuration) 
zonecfg:zone2:anet> set allowed-address=10.0.2.22/24 
(Use the IP address assigned by your instructor)
zonecfg:zone2:anet> set defrouter=10.0.2.2 
(Your instructor will give you the default gateway address)
zonecfg:zone2:anet> end
zonecfg:zone2> exit
```

Check:

``` console
root@solaris:~# zoneadm list -cv 
  ID NAME             STATUS     PATH                           BRAND    IP    
   0 global           running    /                              solaris  shared
   1 zone1            running    /zones/zone1                   solaris  excl  
   - zone2            configured /zones/zone2                   solaris  excl  
```

Then we create the new zone's profile. Start the System Configuration
Tool and repeat all the configuration steps you did for `zone1`. Just
change Computer Name to `zone2`.

``` console
root@solaris:~# sysconfig create-profile -o /root/zone2-profile
```

Before cloning we have to shutdown our running `zone1`:

``` console
root@solaris:~# zoneadm -z zone1 shutdown 
```

Now clone `zone1` and configure `zone2` automatically using this
profile:

``` console
root@solaris:~# zoneadm -z zone2 clone -c /root/zone2-profile zone1 
root@solaris:~# zoneadm list -cv 
  ID NAME             STATUS     PATH                           BRAND    IP    
   0 global           running    /                              solaris  shared
   1 zone1            installed  /zones/zone1                   solaris  excl  
   2 zone2            installed  /zones/zone2                   solaris  excl  
```

Now boot both zones:

``` console
root@solaris:~# zoneadm -z zone1 boot 
root@solaris:~# zoneadm -z zone2 boot 
root@solaris:~# zoneadm list -cv 
  ID NAME             STATUS     PATH                           BRAND    IP    
   0 global           running    /                              solaris  shared
   1 zone1            running    /zones/zone1                   solaris  excl  
   2 zone2            running    /zones/zone2                   solaris  excl  
```

Success! And it was faster than the initial installation, wasn't it?

After it's done, login into zone2.

``` console
root@solaris:~# zlogin zone2 
```

First of all, what about our Apache server?

``` console
root@zone2:~# pkg list '*apache*' 
NAME (PUBLISHER)                                  VERSION                    IFO
web/server/apache-22                              2.2.27-0.175.2.0.0.42.1    i--
```

Great! It's installed already! Check if it's running:

``` console
root@zone2:~# svcs *apache* 
online 11:48:47 svc:/network/http:apache22 
```

Try the `zone2's` address (`10.0.2.22` or whatever IP you used in
`zonecfg` step) in Firefox in the global zone.

**This is Zone1 and it works!** - of course, we have cloned not only
the installed applications, but also all the files. Change it to **Zone2**,
just for consistency sake. Now you know how to use the vi editor, don't
you?

``` console
root@zone2:~# vi /var/apache2/2.2/htdocs/index.html 
```

