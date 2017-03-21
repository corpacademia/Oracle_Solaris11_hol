In the beginning of this lab we also mentioned that it's possible to
install zones together with the host installation. It's time to learn
how to do that.

First of all, we have to configure the zone we are going to install.
Let's create the simplest possible configuration with `zonecfg(1M)`.

``` console
root@solarislab:~# zonecfg -z ai-zone create
```

This command creates a non-global zone with all default parameters. Now
we have to export this configuration into a file:

``` console
root@solarislab:~# zonecfg -z ai-zone export -f ai-zone.cfg
```

Take a look at the resulting file:

``` console
root@solarislab:~# cat ai-zone.cfg
create -b
set brand=solaris
set zonepath=/system/zones/%{zonename}
set autoboot=false
set autoshutdown=shutdown
set ip-type=exclusive
add anet
set linkname=net0
set lower-link=auto
set configure-allowed-address=true
set link-protection=mac-nospoof
set mac-address=auto
end
```

Now we have to make it visible from the client we are about to install.
The best way to do it is to place it on a web server inside the local
network. Luckily, we have a running web server as part of the AI server
installation. We just have to configure the directory where to store
files on this web server. Let's use `svccfg(1M)` for that:

``` console
root@solarislab:~# svccfg -s svc:/system/install/server:default listprop
(that means the directory is not configured yet)
root@solarislab:~# mkdir /var/ai/ai-files
root@solarislab:~# chown -R webservd:webservd /var/ai/ai-files/
root@solarislab:~# svccfg -s svc:/system/install/server:default setprop all_services/webserver_files_dir=/var/ai/ai-files
root@solarislab:~# svccfg -s svc:/system/install/server:default listprop  all_services/webserver_files_dir
all_services/webserver_files_dir astring     /var/ai/ai-files
root@solarislab:~# svcadm refresh svc:/system/install/server:default
```

Now copy the zone configuration file to this place:

``` console
root@solarislab:~# cp ai-zone.cfg /var/ai/ai-files
```

Now, which file should we use to tell AI server to install the zone?
What would be your guess? Of course, it's the manifest file! Everything
that has to be *installed* is configured in the manifest! We will use
the same XML file where we configured the Solaris version.

``` console
root@solarislab:~# vi s11.2.11-manifest.xml
```

Scroll down to the end. Insert the following line (marked bold) after
the `software` tag:

<pre>
      &lt;software_data action=&quot;install&quot;&gt;
        &lt;name&gt;pkg:/entire@0.5.11-0.175.2.11.0.5.0&lt;/name&gt;
        &lt;name&gt;pkg:/group/system/solaris-small-server&lt;/name&gt;
        &lt;name&gt;pkg://solaris/system/management/puppet&lt;/name&gt;
      &lt;/software_data&gt;
    &lt;/software&gt;
<b>    &lt;configuration type=&quot;zone&quot; name=&quot;ai-zone&quot; source=&quot;http://10.80.11.34:5555/files/ai-zone.cfg&quot;/&gt;</b>
  &lt;/ai_instance&gt;
&lt;/auto_install&gt;
</pre>

And, of course, after editing the XML file, don't forget to update the
manifest:

``` console
root@solarislab:~# installadm update-manifest -n default-sparc -m s11.2.11 -f ./s11.2.11-manifest.xml
```

Now everything is ready. Once again login into the client domain and
perform the familiar procedure: halt the system (`init 0`) to get to
OpenBoot `ok` prompt, boot it from AI server (`boot net - install`).
Watch the messages on the screen. At some point you'll see the following
lines:

```
21:40:43    Zone name: ai-zone
21:40:43       source: http://10.80.11.34:5555/files/ai-zone.cfg
```

That means everything is good. Wait until the installation finishes,
reboot and login into the client system. First, let's check if the zone
is installed:

``` console
root@ai-client1:~# zoneadm list -cv
  ID NAME             STATUS      PATH                         BRAND      IP
   0 global           running     /                            solaris    shared
```

Hmm... Nothing. Where is our zone? We were told it's going to be
installed at the first boot! Let's investigate further. Check the status
of the following service (it's responsible for zone installation):

``` console
root@ai-client1:~# svcs zones-install
STATE          STIME    FMRI
offline*       17:52:37 svc:/system/zones-install:default
root@ai-client1:~# svcs -x
svc:/system/zones-install:default (Zones auto-install)
 State: offline* transitioning to online since September 24, 2015 05:52:37 PM EDT
Reason: Start method is running.
   See: http://support.oracle.com/msg/SMF-8000-C4
   See: /var/svc/log/system-zones-install:default.log
Impact: This service is not running.
```

Well, it's not running, but it's "transitioning to online". That's a
good sign. Also we can take a look at its log file:

``` console
root@ai-client1:~# tail  /var/svc/log/system-zones-install:default.log
zoneadm: ai-zone: No such zone configured
zoneadm: ai-zone: No such zone configured
Configuring zone ai-zone
Installing zone ai-zone
zoneadm -z ai-zone install  -m /usr/share/auto_install/manifest/zone_default.xml -c /usr/share/auto_install/sc_profiles/enable_sci.xml
The following ZFS file system(s) have been created:
    rpool/VARSHARE/zones/ai-zone
Progress being logged to /var/log/zones/zoneadm.20150924T215408Z.ai-zone.install
       Image: Preparing at /system/zones/ai-zone/root.
```

A-ha! Something is going on! It looks like our zone is being installed
right now. Check the default `zonepath` location:

          
``` console
root@ai-client1:~# ls /system/zones
ai-zone
```

Indeed! The file system for the zone is created already. Check the log
and the service again:

``` console
root@ai-client1:~# tail  /var/svc/log/system-zones-install:default.log
         Startup: Refreshing catalog 'solaris' ... Done
        Planning: Solver setup ... Done
        Planning: Running solver ... Done
        Planning: Finding local manifests ... Done
        Planning: Fetching manifests:   0/280  0% complete
        Planning: Fetching manifests: 280/280  100% complete
        Planning: Package planning ... Done
        Planning: Merging actions ... Done
        Planning: Checking for conflicting actions ... Done
root@ai-client1:~# tail  /var/svc/log/system-zones-install:default.log
        Download:  4644/53146 items   32.9/374.3MB  8% complete (2.4M/s)
        Download:  6193/53146 items   58.5/374.3MB  15% complete (3.7M/s)
        Download:  7479/53146 items   94.5/374.3MB  25% complete (6.1M/s)
        Download:  9006/53146 items  111.6/374.3MB  29% complete (5.4M/s)
        Download: 10556/53146 items  125.9/374.3MB  33% complete (3.2M/s)
        Download: 12752/53146 items  127.9/374.3MB  34% complete (1.6M/s)
        Download: 14934/53146 items  142.4/374.3MB  38% complete (1.7M/s)
        Download: 16263/53146 items  156.6/374.3MB  41% complete (2.8M/s)
        Download: 17750/53146 items  175.2/374.3MB  46% complete (3.2M/s)
root@ai-client1:~# tail  /var/svc/log/system-zones-install:default.log
         Actions: 14096/71080 actions (Installing new actions)
         Actions: 17986/71080 actions (Installing new actions)
         Actions: 21739/71080 actions (Installing new actions)
         Actions: 25137/71080 actions (Installing new actions)
         Actions: 28848/71080 actions (Installing new actions)
         Actions: 32244/71080 actions (Installing new actions)
         Actions: 35602/71080 actions (Installing new actions)
         Actions: 39909/71080 actions (Installing new actions)
         Actions: 43946/71080 actions (Installing new actions)
root@ai-client1:~# svcs zones-install
STATE          STIME    FMRI
offline*       17:52:37 svc:/system/zones-install:default
root@ai-client1:~# zoneadm list -cv
  ID NAME             STATUS      PATH                         BRAND      IP
   0 global           running     /                            solaris    shared
   - ai-zone          installed   /system/zones/ai-zone        solaris    excl
root@ai-client1:~# svcs zones-install
STATE          STIME    FMRI
online         18:00:31 svc:/system/zones-install:default
```

Great! It is installed. Now you can boot it, login via the console
(`zlogin -C ai-zone`), configure its profile and start using it. As you
might have guessed, you can configure the zone with manifests and system
profiles the same way we did it for AI clients. But we leave this
exercise for your homework.

