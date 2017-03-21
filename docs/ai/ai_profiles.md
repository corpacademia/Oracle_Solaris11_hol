Are you tired already of typing in hostname, IP address, root and user
passwords every time you do a system installation? I am! We can solve
this problem by using System Configuration Profiles. We can specify all
those parameters in another XML file (or several files) and use them
during AI installation. This way the system will be usable right after
installation and we won't have that interactive stage at the first boot.

We start with a simple profile and use the same configuration parameters
that we used during interactive configuration.

``` console
root@solarislab:~# sysconfig create-profile -o ai-client
```

You will see the familiar configuration screens. Use the same parameters
(hostname, IP address, router address, user and root passwords, etc.)
that you used before. After you are finished with the tool, you'll see
the following message:

```
    SC profile successfully generated as:
    ./ai-client/sc_profile.xml

    Exiting System Configuration Tool. Log is available at:
    /system/volatile/sysconfig/sysconfig.log.26544
```

You see that the tool has created a directory named `ai-client` and
placed the `sc_profile.xml` file there. We will need this location in
the next command. You can take a look at the resulting XML file to see
its structure and content. Most of things in it are self-explanatory.
Now we have to add this profile to the install service and specify again
the criteria when it should be used. Here is the command for that:

``` console
root@solarislab:~# installadm create-profile -n default-sparc -p ai-client -c hostname=ai-client -f ./ai-client/sc_profile.xml
Created Profile: 'ai-client'
root@solarislab:~# installadm list -p
Service Name  Profile Name Environment Criteria
------------  ------------ ----------- --------
default-sparc ai-client    system      hostname = ai-client
```

Now try to install again our client (`init 0`, `boot net - install`). You
will see that the installation will run the same way, but after reboot
you won't get the configuration screen. You system will get its hostname
and network parameters automatically and you'll see the console prompt:
`ai-client console login: `. That means your system was installed
completely without intervention from your side.

Now you may say: "Well, should I create a separate profile for each and
every system I want to install? They all have different IPs and
hostnames!". You are right&mdash;all system have different identities. Oracle
Solaris engineers know about that and they feel your pain. So they
created a system of templates that you can use to avoid creating
thousands of separate profiles. Let's learn about them.

Take a look at the following file:

``` console
root@solarislab:~# cat /usr/share/auto_install/sc_profiles/template_var
#
# Copyright (c) 2013, Oracle and/or its affiliates. All rights reserved.
#
#
# This file provides a list of supported profile template variables and
# sample values for current version of Solaris.
#
AI_ARCH=i86pc
AI_CPU=i386
AI_HOSTNAME=solaris
AI_IPV4=192.168.56.2
AI_IPV4_PREFIXLEN=24
AI_MAC=01:01:01:01:01:01
AI_MEM=2048
AI_NETWORK=10.0.0.0
AI_NETLINK_DEVICE=e1000g0
AI_NETLINK_VANITY=net0
AI_ROUTER=10.0.0.1
AI_SERVICE=default-i386
AI_ZONENAME=testzone
```

These are the variables that you can use in system configuration
profiles: IP address, hostname and others. It's very important to notice
that those are system parameters *at the time it boots with the AI
image*. In case of SPARC systems those parameters as set in OpenBoot
with `setenv network-boot-parameters` command. Usually (but not
necessarily) those are parameters you want to see in your system after
installation. That's why we can use them in SC profiles and get what we
expected.

Let's copy the SC profile XML file that we just created with the
interactive tool and edit it.

``` console
root@solarislab:~# cp ai-client/sc_profile.xml ./sc_template.xml
root@solarislab:~# vi ./sc_template.xml
```

Instead of giving you step by step instructions on what to change, it
seems to be easier to provide you with the output of `diff(1)` command.
It's pretty easy to figure out the changes that were made.

<pre>
root@solarislab:~# <b>diff ai-client/sc_profile.xml sc_template.xml</b>
8c8
&lt;         &lt;propval type=&quot;astring&quot; name=&quot;nodename&quot; value=&quot;ai-client&quot;/&gt;
---
&gt;         &lt;propval type=&quot;astring&quot; name=&quot;nodename&quot; value=&quot;{{AI_HOSTNAME}}&quot;/&gt;
21c21
&lt;         &lt;propval type=&quot;net_address_v4&quot; name=&quot;static_address&quot; value=&quot;10.80.10.92/23&quot;/&gt;
---
&gt;         &lt;propval type=&quot;net_address_v4&quot; name=&quot;static_address&quot; value=&quot;{{AI_IPV4}}/{{AI_IPV4_PREFIXLEN}}&quot;/&gt;
24c24
&lt;         &lt;propval type=&quot;net_address_v4&quot; name=&quot;default_route&quot; value=&quot;10.80.11.254&quot;/&gt;
---
&gt;         &lt;propval type=&quot;net_address_v4&quot; name=&quot;default_route&quot; value=&quot;{{AI_ROUTER}}&quot;/&gt;
</pre>

After you are done with editing, install this profile in the install
service and remove the old one. Note that this time we don't specify any
criteria, which means that this profile will be applied to all clients.

``` console
root@solarislab:~# installadm create-profile -n default-sparc -p sc-template  -f ./sc_template.xml
Created Profile: 'sc-template'
root@solarislab:~# installadm list -p
Service Name  Profile Name Environment Criteria
------------  ------------ ----------- --------
default-sparc ai-client    system      hostname = ai-client
              sc-template  system      none
root@solarislab:~# installadm delete-profile -n default-sparc -p ai-client
Deleted Profile: 'ai-client'
root@solarislab:~# installadm list -p
Service Name  Profile Name Environment Criteria
------------  ------------ ----------- --------
default-sparc sc-template  system      none
```

And now it's time to test this profile. First, try to install your
client system with the current settings. If it works OK, then change the
settings with `setenv network-boot-parameters` to a different hostname
and IP and test again. Here is the example. Your instructor will give
you the necessary information about available IP addresses (please,
don't "invent" them yourself, we have very strict rules in the
laboratory!).

<pre>
{0} ok <b>printenv network-boot-arguments</b>
network-boot-arguments =  hostname=ai-client,host-ip=10.80.10.92,router-ip=10.80.11.254,subnet-mask=255.255.254.0,file=http://10.80.11.34:5555/cgi-bin/wanboot-cgi
{0} ok <b>setenv network-boot-arguments hostname=ai-client1,host-ip=10.80.10.91,router-ip=10.80.11.254,subnet-mask=255.255.254.0,file=http://10.80.11.34:5555/cgi-bin/wanboot-cgi</b>
network-boot-arguments =  hostname=ai-client1,host-ip=10.80.10.91,router-ip=10.80.11.254,subnet-mask=255.255.254.0,file=http://10.80.11.34:5555/cgi-bin/wanboot-cgi
{0} ok
</pre>

After the installation reboot the system and check if it set the
parameters you expected.

``` console
ai-client1 console login: lab
Password:
Oracle Corporation      SunOS 5.11      11.2    August 2015
lab@ai-client1:~$ ipadm
NAME              CLASS/TYPE STATE        UNDER      ADDR
lo0               loopback   ok           --         --
   lo0/v4         static     ok           --         127.0.0.1/8
   lo0/v6         static     ok           --         ::1/128
net0              ip         ok           --         --
   net0/v4        static     ok           --         10.80.10.91/23
   net0/v6        addrconf   ok           --         fe80::214:4fff:fef8:5f6/10
   net0/v6        addrconf   ok           --         2606:b400:410:831:214:4fff:fef8:5f6/64
```

Success! Again, let's summarize what we have learned in this exercise.

1.  System parameters which we usually set at the first boot, can be
    configured and set via System Configuration Profiles (SC profiles).
2.  SC profiles can be created using interactive SC tool
    (`sysconfig(1M)`) or by copying and editing XML files.
3.  You can use variables in profiles to install systems with different
    hostnames, IP addresses, etc.

