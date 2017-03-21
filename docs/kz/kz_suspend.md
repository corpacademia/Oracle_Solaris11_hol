One of the coolest Kernel Zones features is their ability to suspend and
resume operations. You can use this feature either to _warm migrate_ a
kernel zone from one system to another, or to preserve your
application's state while rebooting the global zone.

To be able to use this capability we have to configure the storage space
where we keep the suspended zone. First check if it's configured already
and then add the path to file's location.

``` console
root@solarislab:~# zonecfg -z kzone1 info suspend
root@solarislab:~# zonecfg -z kzone1
zonecfg:kzone1> add suspend
zonecfg:kzone1:suspend> set path=/system/zones/kzone1/suspend
zonecfg:kzone1:suspend> end
zonecfg:kzone1> exit
root@solarislab:~# zonecfg -z kzone1 info suspend
suspend:
        path: /system/zones/kzone1/suspend
        storage not specified
```

Good. Now we can try to suspend the kernel zone.

``` console
root@solarislab:~# zoneadm -z kzone1 suspend
root@solarislab:~# zoneadm list -cv
  ID NAME             STATUS      PATH                         BRAND      IP
   0 global           running     /                            solaris    shared
   5 zone1            running     /system/zones/zone1          solaris    excl
   - kzone1           installed   -                            solaris-kz excl
```

So what? It doesn't look very spectacular, does it? Note that `kzone1`
is not in any special `suspended` status, it's just `installed`, like if
we just shut it down. How can we resume it? OK, according to the zones'
lifecycle, from the `installed` state we can `boot` the zone. There is
no special `resume` command.

``` console
root@solarislab:~# zoneadm -z kzone1 boot
root@solarislab:~# zoneadm list -cv
  ID NAME             STATUS      PATH                         BRAND      IP
   0 global           running     /                            solaris    shared
   5 zone1            running     /system/zones/zone1          solaris    excl
   7 kzone1           running     -                            solaris-kz excl
```

Again, nothing spectacular. We need something to see how the zone is
getting suspended and resumed. Open another terminal window, login to
the host (global zone) and then to `kzone1`. Inside `kzone1` start a
simple script that prints out the current timestamp every second.

``` console
root@solarislab:~# zlogin -C kzone1
[Connected to zone 'kzone1' console]

kzone11-3 console login: lab
Password: oracle1 (not shown)
Last login: Tue Aug  4 15:09:04 2015 on console
Oracle Corporation      SunOS 5.11      11.3    June 2015
lab@kzone11-3:~$ while true; do date ; sleep 1 ; done
Wednesday, August  5, 2015 02:35:46 PM EDT
Wednesday, August  5, 2015 02:35:47 PM EDT
Wednesday, August  5, 2015 02:35:48 PM EDT
Wednesday, August  5, 2015 02:35:49 PM EDT
Wednesday, August  5, 2015 02:35:50 PM EDT
Wednesday, August  5, 2015 02:35:51 PM EDT
. . . (it goes on and on and on)
```

Now switch to the previous terminal session in the global zone and
suspend and resume `kzone1` again, while watching what's going on in the
second window (in the `kzone1`).

``` console
root@solarislab:~# zoneadm -z kzone1 suspend
root@solarislab:~# zoneadm -z kzone1 boot
```

In the second window at the same time:

``` console
lab@kzone1:~$ while true; do date ; sleep 1 ; done
Wednesday, August  5, 2015 02:42:04 PM EDT
Wednesday, August  5, 2015 02:42:05 PM EDT
Wednesday, August  5, 2015 02:42:06 PM EDT
Wednesday, August  5, 2015 02:42:07 PM EDT
Wednesday, August  5, 2015 02:42:08 PM EDT
Wednesday, August  5, 2015 02:42:09 PM EDT
Wednesday, August  5, 2015 02:42:10 PM EDT

[NOTICE: Zone suspending]
OSQ: suspending devices...
        suspending ORCL,kz-zvterm@0 (aka zvterm)
        suspending ORCL,kz-zvblk@0 (aka zvblk)
        suspending zvcntrl@0
        suspending zvnet@0
        suspending zvsdir@0
        suspending ORCL,kz-extenders@ff (aka zvnex)

[NOTICE: Zone halted]

[NOTICE: Zone resuming]
        resuming ORCL,kz-extenders@ff (aka zvnex)
        resuming zvsdir@0
        resuming zvnet@0
        resuming zvcntrl@0
        resuming ORCL,kz-zvblk@0 (aka zvblk)
        resuming ORCL,kz-zvterm@0 (aka zvterm)
OSQ: resume devices completed
Wednesday, August  5, 2015 02:42:11 PM EDT
Wednesday, August  5, 2015 02:42:12 PM EDT
Wednesday, August  5, 2015 02:42:13 PM EDT
Wednesday, August  5, 2015 02:42:14 PM EDT
Wednesday, August  5, 2015 02:42:15 PM EDT
Wednesday, August  5, 2015 02:42:16 PM EDT
^C
```

This is really interesting! Now we see that the zone has been suspended
and then resumed and continued the script from the point where it was
left. That was definitely not possible with traditional non-global zones
and now we can do this with kernel zones. Think about it: how can you
use it with your applications? What might be a good test to see if they
are really suspended and resumed?

