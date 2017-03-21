Now as we have installed and configured our zones, let's login into both
and see how they look from inside. Remember: in the listings below the
kernel zone's hostname is `kzone1`, non-global zone's one is
`zone1`.

### Processes

We start with some basic process-related commands. For example, how many
processes are running just after the zone is booted?

``` console
root@kzone1:~# ps -ef | wc -l
      56
```

``` console
root@zone1:~# ps -ef | wc -l
      36
```

You see, there some extra processes running in the kernel zone. Let's
sort them by PID and look at the first 20 of them:

``` console
root@kzone1:~# ps -ef | sort -k 2 | head -20
    root     0     0   0 15:16:13 ?           0:00 sched
    root     1     0   0 15:16:13 ?           0:00 /usr/sbin/init
    root     2     0   0 15:16:13 ?           0:00 pageout
    root     3     0   0 15:16:13 ?           0:01 fsflush
    root     5     0   0 15:16:11 ?           0:03 zpool-rpool
    root     6     0   0 15:16:13 ?           0:00 kmem_task
    root     7     0   0 15:16:13 ?           0:00 intrd
    root     8     0   0 15:16:13 ?           0:00 vmtasks
    root     9     0   0 15:16:13 ?           0:00 postwaittq
    root    13     1   0 15:16:16 ?           0:10 /lib/svc/bin/svc.startd
    root    15     1   0 15:16:16 ?           2:03 /lib/svc/bin/svc.configd
  netcfg    50     1   0 15:17:45 ?           0:00 /lib/inet/netcfgd
   dladm    65     1   0 15:17:54 ?           0:00 /usr/sbin/dlmgmtd
  netadm    84     1   0 15:18:06 ?           0:00 /lib/inet/ipmgmtd
    root    97     1   0 15:18:08 ?           0:00 /lib/inet/in.mpathd
    root   122     1   0 15:18:12 ?           0:00 /usr/lib/pfexecd
    root   165     1   0 15:18:25 ?           0:00 /usr/lib/sysevent/syseventd
  daemon   202     1   0 15:18:26 ?           0:00 /usr/lib/utmpd
    root   217     1   0 15:18:27 ?           0:00 /usr/lib/zones/zonestatd
    root   221     1   0 15:18:27 ?           0:00 /usr/lib/rad/rad -sp
```

``` console
root@zone1:~# ps -ef | sort -k 2 | head -20
     UID   PID  PPID   C    STIME TTY         TIME CMD
    root 21253 21253   0 14:28:07 ?           0:00 zsched
    root 21971 21253   0 14:28:16 ?           0:00 /usr/sbin/init
    root 22151 21253   0 14:28:16 ?           0:04 /lib/svc/bin/svc.startd
    root 22153 21253   0 14:28:16 ?           1:06 /lib/svc/bin/svc.configd
  netcfg 22186 21253   0 14:29:12 ?           0:00 /lib/inet/netcfgd
  daemon 22229 21253   0 14:29:15 ?           0:00 /lib/crypto/kcfd
    root 22236 21253   0 14:29:16 ?           0:00 /usr/lib/pfexecd
  netadm 22252 21253   0 14:29:17 ?           0:00 /lib/inet/ipmgmtd
    root 22266 21253   0 14:29:17 ?           0:00 /lib/inet/in.mpathd
  daemon 22272 21253   0 14:29:18 ?           0:00 /usr/lib/utmpd
    root 22287 21253   0 14:29:18 ?           0:00 /usr/lib/rad/rad -sp
    root 22333 21253   0 14:29:18 ?           0:00 /usr/lib/dbus-daemon --system
  netadm 22771 21253   0 14:30:43 ?           0:00 /lib/inet/nwamd
    root 22892 21253   0 14:30:46 ?           0:00 /usr/lib/zones/zoneproxy-client -s localhost:1008
    root 22899 21253   0 14:30:47 ?           0:00 /usr/sbin/nscd
    root 22955 21253   0 14:30:49 ?           0:00 /usr/sbin/cron
  daemon 23010 21253   0 14:30:51 ?           0:00 /usr/sbin/rpcbind
    root 23040 21253   0 14:30:51 ?           0:00 /usr/lib/inet/in.ndpd
    root 23048 21253   0 14:30:52 ?           0:00 /usr/lib/autofs/automountd
```

Now you see the big difference between kernel zones and non-global
zones. In non-global zones all the processes are part of the global
zone's process space, so their PIDs start with some big number. In
kernel zones they start with `0` and the list is not much different from
the process list in the global zone.

Now let's look at them from the global zone's perspective. Use the `-Z`
switch for the `ps` command to see zone's names next to the processes.

``` console
root@solarislab:~# ps -efZ
    ZONE      UID   PID  PPID   C    STIME TTY         TIME CMD
  global     root     0     0   0   Jul 29 ?           0:10 sched
  global     root     5     0   0   Jul 29 ?           7:26 zpool-rpool
  global     root     6     0   0   Jul 29 ?           0:31 kmem_task
  global     root     1     0   0   Jul 29 ?           0:01 /usr/sbin/init
  global     root     2     0   0   Jul 29 ?           0:00 pageout
  global     root     3     0   0   Jul 29 ?          11:32 fsflush
  global     root     7     0   0   Jul 29 ?           0:12 intrd
  global     root     8     0   0   Jul 29 ?           1:02 vmtasks
. . . (skipped) . . .
   zone1     root 21253     1   0 12:28:07 ?           0:00 zsched
  global     root  1214     1   0   Jul 29 ?           0:03 /usr/lib/ep/eptelemon
  global     root  1116     1   0   Jul 29 ?           0:21 /usr/lib/ep/epdetector dcdc cpu0
  kzone1     root 19817     1   0 11:55:11 ?           0:00 zsched
. . . (skipped) . . .
  kzone1     root 19995 19817   0 11:55:12 ?           6:23 /usr/lib/kzhost
  global  pkg5srv 20737     1   0 12:19:48 ?           0:16 /usr/apache2/2.2/bin/htcacheclean -d20160 -i -l 2048M -n -p /var/cache/pkg/sysr
  global  pkg5srv 20742 20739   0 12:19:49 ?           0:21 /usr/apache2/2.2/bin/64/httpd.worker -f /system/volatile/pkg/sysrepo/sysrepo_ht
  global  pkg5srv 20743 20739   0 12:19:49 ?           0:22 /usr/apache2/2.2/bin/64/httpd.worker -f /system/volatile/pkg/sysrepo/sysrepo_ht
   zone1     root 22151     1   0 12:28:16 ?           0:04 /lib/svc/bin/svc.startd
   zone1   netcfg 22186     1   0 12:29:12 ?           0:00 /lib/inet/netcfgd
  global  pkg5srv 20746     1   0 12:19:50 ?           0:00 /usr/lib/zones/zoneproxyd
   zone1     root 22153     1   0 12:28:16 ?           1:06 /lib/svc/bin/svc.configd
  global     root 21061     1   0 12:28:04 ?           0:01 zoneadmd -z zone1
   zone1     root 21971 21253   0 12:28:16 ?           0:00 /usr/sbin/init
   zone1     root 22266     1   0 12:29:17 ?           0:00 /lib/inet/in.mpathd
. . . (skipped) . . .
  global     root 23168 14852   0 12:32:01 pts/2       0:00 ps -efZ
```

Now this gets interesting! We see a lot of processes in `zone1`, and (no
surprise here) their PIDs are all the same that we just saw from inside
the zone. As for `kzone1`, we see only 2 processes. Just to check, let's
use `-z zonename` parameter:

``` console
root@solarislab:~# ps -f -Z -z kzone1
    ZONE      UID   PID  PPID   C    STIME TTY         TIME CMD
  kzone1     root 19817     1   0 11:55:11 ?           0:00 zsched
  kzone1     root 19995 19817   0 11:55:12 ?           6:28 /usr/lib/kzhost
```

That's right! Just two `kzone1`'s processes are visible from the global
zone. Actually, kzhost is just a big process that keeps all the kernel
zone's processes inside. How big is is? Yes, you guessed it right
again&mdash;it takes the amount of memory that is configured for `kzone1`.
You can check it with the following `ps` command:

``` console
root@solarislab:~# ps -o pid,vsz,rss,comm -z kzone1
  PID  VSZ  RSS COMMAND
19817    0    0 zsched
19995 2120528 2119512 /usr/lib/kzhost
```

The bottom line for these experiments:

-   non-global zone's processes are visible from the global zone and
    belong to its *process space*
-   kernel zone's processes are visible only inside the zone and they
    have *their own* process space

When choosing between non-global zones and kernel zones, please keep in
mind this major difference.

### Processors and Memory

Let's move on and figure out what resources are available in the zones.
What is the output of `psrinfo(1M)`?

``` console
root@kzone1:~# psrinfo
0       on-line   since 06/15/2015 15:16:12
```

``` console
root@zone1:~# psrinfo
0       on-line   since 06/10/2015 17:30:04
1       on-line   since 06/10/2015 17:30:07
2       on-line   since 06/10/2015 17:30:07
3       on-line   since 06/10/2015 17:30:07
4       on-line   since 06/10/2015 17:30:07
5       on-line   since 06/10/2015 17:30:07
6       on-line   since 06/10/2015 17:30:07
7       on-line   since 06/10/2015 17:30:07
```

You see the difference? By default Solaris 11.2 assigns only one virtual
CPU to a kernel zone (you can change that with `zonecfg` command). In
Solaris 11.3 this default has been changed to 4 virtual CPUs. A
non-global zone by default can use all the CPUs available in the global
zone. Run the same `psrinfo` command in the global zone and compare the
results.

What does it mean for us? Non-global zones have more flexibility in
terms of resource control. By default you give them what's available and
they share all the resources with the global zone. Later you can *limit*
their resource usage or assign specific groups of CPUs. It's completely
opposite in kernel zones. By default they get the minimum, just one
virtual CPU. You can change that if/when you need more, even during the
initial configuration phase.

What about memory? How much is available? As you can see below, the
situation is very similar.

``` console
root@kzone1:~# prtconf | grep Memory
Memory size: 2048 Megabytes
```

``` console
root@zone1:~# prtconf | grep Memory
prtconf: devinfo facility not available
Memory size: 16384 Megabytes
```

``` console
root@solarislab:~# prtconf | grep Memory
Memory size: 16384 Megabytes
```

The same thing here: the non-global zone by default can use the whole
physical memory available in the global zone. For the kernel zone we
have to specify how much memory we want to assign to it. By default it's
2 Gigabytes in `11.2` and 4 Gigabytes in `11.3`.

### Virtualization support

Also it's interesting to take a look at what `virtinfo(1M)` reports in
each case: kernel zone, non-global zone and global zone.

``` console
root@kzone1:~# virtinfo
NAME            CLASS
kernel-zone     current
logical-domain  parent
non-global-zone supported
```

``` console
root@zone1:~# virtinfo
NAME            CLASS
non-global-zone current
logical-domain  parent
logical-domain  supported
```

``` console
root@solarislab:~# virtinfo
NAME            CLASS
logical-domain  current
non-global-zone supported
kernel-zone     supported
logical-domain  supported
```

You can see that you can even create non-global zones inside kernel
zones! You can do that, but remember: "Don't over-virtualize!"

Another interesting test: try to run the `zonename` command in the
global zone, kernel zone and non-global zone. What does it show in each
of them?

``` console
root@solarislab:~# zonename
global
```

``` console
root@kzone1:~# zonename
global
```

``` console
root@zone1:~# zonename
zone1
```

Interesting! That means it's really hard to distinguish a kernel zone
from a "true" global zone. Your applications will never notice that they
are running in a virtualized environment.

### Packages

One of the great things about non-global zones is that you can manage
software packages in them both from inside and outside. You can install
packages being logged in a non-global zone, but more importantly, you
can update, install and uninstall packages from the global zone.
Normally, when you do `pkg update` in the global zone, all installed
non-global zones are being updated too. You can install packages
recursively into the zones by using `pkg install -r` command. Even more,
you can name the specific zones you want to install the packages into,
with `-z <zonename>` parameter, or exclude some zones with `-Z
<zonename>`. What's important, you can do that even when the zones are
not running. That's a big advantage of non-global zones in software
management.

As you might have guessed already, it's different in kernel zones. As
soon as they have a separate block device with a file system in it, 
that is not available from the global zone, we can't update their 
packages without logging into the kernel zone. All package-related 
commands&mdash;install, update, uninstall&mdash;should be performed 
from within the kernel zone.

Comparing software management operations in non-global zones and kernel
zones, we can see that because non-global zones are more tightly
integrated with the global zone, it gives us more flexibility in package
operations. Kernel zones are more isolated by design (separate kernel,
ZFS pool and process space), so they are treated as separate Solaris
instances. Which approach is better&mdash;you decide, depending on your
particular situation and requirements.

