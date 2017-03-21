**Task:** You want to control the zones' resource usage. You want to
assign certain amount of processing power to each zone.

**Lab:** We now know how to create and clone zones. Now let's try to cap
CPU resources in one zone to demonstrate the basics of resource
management in Solaris.

First, run a simple CPU-consuming script in the `zone1`:

``` console
root@solaris:~# zlogin zone1 "bash -c 'while true ; do date > /dev/null ; done'" 
```

Note that we are simply using `zlogin` to pass the command to the zone.

What's going on in the global zone? Open another window, become root and
check:

``` console
root@solaris:~# vmstat 5 
```

Idle is 0, system time is around 70%. Not good.

``` console
root@solaris:~# zonestat 5 
```

`zone1` consumes 70-80% of total resources, the rest is spent in global
zone (most likely serving `zone1`'s requests). We decided to reduce the
`zone1`'s resource consumption and give it only 50% of our CPU cycles. You
can leave the window with `zonestat` running open and start another
terminal session to change the zone's parameters. This way you'll be
able to see the changes in real time.

``` console
root@solaris:~# zonecfg -z zone1 
zonecfg:zone1> add capped-cpu 
zonecfg:zone1:capped-cpu> set ncpus=0.5 
zonecfg:zone1:capped-cpu> end 
zonecfg:zone1> exit 
root@solaris:~# zoneadm -z zone1 apply 
zone 'zone1': Checking: Adding rctl name=zone.cpu-cap
zone 'zone1': Applying the changes
```

Look into another window where you have `zonestat` running (or run
`zonestat 5` again). You should see the CPU utilization by `zone1` dropped
down to 50%.

You can check the updated `zone1`'s configuration and see that it's CPU
cap is now set to 50%.

``` console
root@solaris:~# zonecfg -z zone1 info
. . . 
capped-cpu:
    [ncpus: 0.50]
rctl:
    name: zone.cpu-cap
    value: (priv=privileged,limit=50,action=deny)
```

That means next time you run zone1 it will receive 50% of one CPU. It's
important to note: it's **not** 50% of ALL CPUs, it's 50% of only one
virtual CPU. In our case in VirtualBox VM we have only one CPU. In most
real life cases you will have tens and hundreds of virtual CPUs. Set
your values accordingly.

Is it also possible to change this CPU cap parameter on the fly. This
change will be temporary, only until the reboot.

``` console
root@solaris:~# prctl -n zone.cpu-cap -r -v 25 -i zone zone1 
```

Check if it works looking in the window with `zonestat` or running it
again:

``` console
root@solaris:~# zonestat 5 
```

Don't forget to stop the infinite loop in your zone! Or simply halt the
zone.

``` console
root@solaris:~# zoneadm -z zone1 halt 
```

Other resources can be capped this way as well: memory, swap, number of
threads etc. Again, think about how it can be used in real life
situations?


