Let's start with the simplest possible case: we have systems with only
one CPU architecture, we want to install the same Solaris version and
the same set of packages on every system. We have to install the AI
package and create the first install service, which is going to be the
default one.

Start with checking if `installadm(1M)` is installed on the system:

``` console
root@solarislab:~# pkg list -a installadm
NAME (PUBLISHER)                                  VERSION                    IFO
install/installadm                                0.5.11-0.175.2.8.0.1.2     i--
```

Letter `'i'` in the last column mean that it's installed. If it's not,
just install it with `pkg install install/installadm`. Now let's check
if we have any install services already (who knows, somebody might have
configured them before us).

``` console
root@solarislab:~# installadm list
There are no services configured on this server.
```

So far, so good. We can create our first install service right now.

``` console
root@solarislab:~# installadm create-service
OK to use subdir of /export/auto_install to store image? [y|N]: y
  0% : Service svc:/network/dns/multicast:default is not online.  Installation services will not be advertised via multicast DNS.
  0% : Creating service from: pkg:/install-image/solaris-auto-install
  0% : Using publisher(s):
  0% :     solaris: http://10.80.11.34:81/
  5% : Refreshing Publisher(s)
 15% : Planning Phase
 24% : Download Phase
 62% : Actions Phase
 91% : Finalize Phase
 91% : Creating sparc service: solaris11_3-sparc
 91% : Image path: /export/auto_install/solaris11_3-sparc
 91% : Setting "solaris" publisher URL in default manifest to:
 91% :  http://10.80.11.34:81/
 91% : Creating default-sparc alias
 91% : Setting "solaris" publisher URL in default manifest to:
 91% :  http://10.80.11.34:81/
100% : Created Service: 'solaris11_3-sparc'
100% : Refreshing SMF service svc:/system/install/server:default
100% : Enabling SMF service svc:/network/dhcp/server:ipv4
100% : Warning: mDNS registry of service 'solaris11_3-sparc' could not be verified.
100% : Warning: mDNS registry of service 'default-sparc' could not be verified.
```

Let's look more closely at the output. First of all, we have to confirm
that we are OK with storing boot image in the default
`/export/auto_install` directory (of course, you can change that if you
want). Then, as soon as it's just our local lab exercise, we are not
going to use multicast DNS to advertise it widely. Three lines after
that tell us that we are going to use a special package that contains
Solaris boot image and it's located on our local IPS repository.

If you look further, you see that `installadm` creates a SPARC service
and from its name we can guess that it's based on Oracle Solaris 11.3
version. Why? Because the last version we have in the repository is
11.3. You can check it:

``` console
root@solarislab:~# pkg list -af entire | head
NAME (PUBLISHER)                                  VERSION                    IFO
entire                                            0.5.11-0.175.3.0.0.29.0    ---
entire                                            0.5.11-0.175.3.0.0.28.0    ---
entire                                            0.5.11-0.175.2.14.0.5.0    ---
entire                                            0.5.11-0.175.2.13.0.6.0    ---
entire                                            0.5.11-0.175.2.12.0.6.0    i--
entire                                            0.5.11-0.175.2.12.0.5.0    ---
entire                                            0.5.11-0.175.2.11.0.5.0    ---
entire                                            0.5.11-0.175.2.10.0.5.0    ---
entire                                            0.5.11-0.175.2.9.0.5.0     ---
```

You see: our current installed version is `11.2.12`, but the latest
available is `11.3.0.0.29`. So `installadm` uses it by default.

What else can we learn from the output? That `installadm` has also
created a default manifest for this service and set the publisher in
that manifest to our current publisher with local address `10.80.11.34`.
It also has enabled a DHCP service, but we don't need it for SPARC
installation so we'd better disable it right now:

``` console
root@solarislab:~# svcadm disable network/dhcp/server:ipv4
```

Now the most interesting part: let's try to install a system from this
AI server. To do that we have to get to the console of a client system.
In our lab environment we have created several guest domains, but we
didn't install Solaris in them. You instructor will give you all
necessary information how to access your domain's console. Usually it's
`telnet localhost 5000` (or other port number). When you get to OpenBoot
`ok` prompt enter the following `setenv` command. Make sure that
everything that goes after `network-boot-arguments` doesn't have any
spaces. Your instructor will provide you with the information about
hostnames and IP addresses&mdash;they are marked in italic in the following command.

<pre>
{0} ok setenv network-boot-arguments hostname=<i>ai-client</i>,host-ip=<i>10.80.10.91</i>,router-ip=<i>10.80.11.254</i>,subnet-mask=255.255.254.0,file=http://10.80.11.34:5555/cgi-bin/wanboot-cgi
</pre>

After that enter the boot command and watch the installation process. We
have skipped most of the output, but you are encouraged to read and try
to interpret as much as possible. You instructor will help you in that.

<pre>
{0} <b>ok boot net - install</b>
network-boot-arguments =  hostname=ai-client,host-ip=10.80.10.92,router-ip=10.80.11.254,subnet-mask=255.255.254.0,file=http://10.80.11.34:5555/cgi-bin/wanboot-cgi
NOTICE: Entering OpenBoot.
NOTICE: Fetching Guest MD from HV.
NOTICE: Starting additional cpus.
NOTICE: Initializing LDC services.
NOTICE: Probing PCI devices.
NOTICE: Finished PCI probing.


SPARC T4-2, No Keyboard
Copyright (c) 1998, 2014, Oracle and/or its affiliates. All rights reserved.
OpenBoot 4.36.1, 4.0000 GB memory available, Serial #83607872.
Ethernet address 0:14:4f:fb:c1:40, Host ID: 84fbc140.



Boot device: /virtual-devices@100/channel-devices@200/network@0  File and args: - install

. . . . . . . .
Wed Sep 16 21:19:40 wanboot info: miniroot: Download complete
SunOS Release 5.11 Version 11.3 64-bit
Copyright (c) 1983, 2015, Oracle and/or its affiliates. All rights reserved.
Remounting root read/write
Probing for device nodes ...
Preparing network image for use
Downloading solaris.zlib
. . . . . . . . . . .
Done mounting image
Configuring devices.
Hostname: ai-client
Service discovery phase initiated
Service name to look up: default-sparc
Service discovery over multicast DNS failed
Service default-sparc located at 10.80.11.34:5555 will be used
Service discovery finished successfully
Process of obtaining install manifest initiated
Using the install manifest obtained via service discovery

ai-client console login:
Automated Installation started
The progress of the Automated Installation will be output to the console
Detailed logging is in the logfile at /system/volatile/install_log
Press RETURN to get a login prompt at any time.

21:22:41    Install Log: /system/volatile/install_log
21:22:41    Using Derived Script: /system/volatile/ai.xml
21:22:41    Using profile specification: /system/volatile/profile
21:22:41    Using service list file: /var/run/service_list
21:22:41    Starting installation.
. . . . .
21:33:19    91% update-filesystem-owner-group completed.
21:33:19    92% transfer-ai-files completed.
21:33:19    100% create-snapshot completed.
21:33:19    100% None
21:33:19    Automated Installation succeeded.
21:33:20    You may wish to reboot the system at this time.
Automated Installation finished successfully
The system can be rebooted now
Please refer to the /system/volatile/install_log file for details
After reboot it will be located at /var/log/install/install_log
</pre>

We skipped a lot of messages about the installation progress, but you
may want to take a look at them&mdash;it'll give you a better understanding
about the processes going on during installation.

Well, now we can reboot the system to see what was installed. For the AI
boot image login is `root`, password is `solaris`.

<pre> 
ai-client console login:
ai-client console login: <b>root</b>
Password: solaris
Sep 16 21:40:07 ai-client login: ROOT LOGIN /dev/console
Oracle Corporation      SunOS 5.11      11.3    August 2015
root@ai-client:~# <b>reboot</b>
Sep 16 21:40:11 ai-client reboot: initiated by root on /dev/console
. . . . . .
</pre>

After several familiar messages you will come to the System
Configuration screen. From here you define the hostname, network
parameters such as IP address, router IP, etc., time zone, root and user
passwords and others. Pretty standard procedure, so go ahead and enter
the following:

<pre>
Computer name: <b>ai-client</b>
Network configuration: <b>Manually</b>
IP address: <i>(provided by the instructor)</i>
Netmask: <b>255.255.254.0</b>
Router: <b>10.80.11.254</b>
Do not configure DNS
Alternate Name Service: <b>None</b>
Time Zone: Americas -> United States -> <i>your time zone</i>
Language: <i>your preferred language</i>
Territory: <i>your territory</i>
Date and Time: accept the default by pressing <kbd>F2</kbd>
Keyboard: <i>your preferred keyboard</i>
Root password: <b>solaris1</b>
Your real name: <b>Lab User</b>
Username: <b>lab</b>
User password: <b>oracle1</b>
</pre>

On the final screens just press <kbd>F2</kbd> to continue.

After a while you will see the login prompt. Use `lab/oracle1` to login:

<pre>
ai-client console login:
ai-client console login: <b>lab</b>
Password: <b>oracle1</b>
Oracle Corporation      SunOS 5.11      11.3    August 2015
lab@ai-client:~$
</pre>

So, what have we just installed?

``` console
lab@ai-client:~$ uname -a
SunOS ai-client 5.11 11.3 sun4v sparc sun4v
lab@ai-client:~$ pkg list entire
NAME (PUBLISHER)                                  VERSION                    IFO
entire                                            0.5.11-0.175.3.0.0.29.0    i--
lab@ai-client:~$ pkg list | wc
     581    1744   47084
lab@ai-client:~$ pkg list | grep group
group/system/management/rad/rad-server-interfaces 0.5.11-0.175.3.0.0.29.1    i--
group/system/solaris-core-platform                0.5.11-0.175.3.0.0.29.1    i--
group/system/solaris-large-server                 0.5.11-0.175.3.0.0.29.0    i--
```

Congratulations, we have just created an install server and installed
our first client from it. We used the simplest possible configuration:
by default we installed the latest available release of Solaris in you
repository, default manifest specifies that we install
`solaris-large-server` group and also we used interactive System
Configuration tool to provide hostname, IP address, etc. Of course, all
that can be tuned to your particular situation. And this is what we are
going to do in the rest of this lab.

