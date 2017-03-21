Of course, there are situations when you want to install a different
version of Oracle Solaris. For instance, in our lab repository we have
versions 11 Express, 11.0, 11.1, 11.2 with their support SRUs. How can
we specify a different version for installation?

In this case we have to create a separate install service. So, the more
different versions and CPU architectures you want to install, the more
install services you have. Let's figure out what's available. We list 10
most recent updates here:

``` console
root@solarislab:~# pkg list -avf  install-image/solaris-auto-install | head    
FMRI                                                                                        IFO
pkg://solaris/install-image/solaris-auto-install@5.11,5.11-0.175.3.0.0.29.1:20150817T163032Z ---
pkg://solaris/install-image/solaris-auto-install@5.11,5.11-0.175.3.0.0.28.0:20150803T161232Z ---
pkg://solaris/install-image/solaris-auto-install@5.11,5.11-0.175.2.14.0.5.0:20150910T165659Z ---
pkg://solaris/install-image/solaris-auto-install@5.11,5.11-0.175.2.13.0.6.0:20150810T175313Z ---
pkg://solaris/install-image/solaris-auto-install@5.11,5.11-0.175.2.12.0.6.0:20150715T155314Z ---
pkg://solaris/install-image/solaris-auto-install@5.11,5.11-0.175.2.12.0.5.0:20150701T004857Z ---
pkg://solaris/install-image/solaris-auto-install@5.11,5.11-0.175.2.11.0.5.0:20150610T201614Z ---
pkg://solaris/install-image/solaris-auto-install@5.11,5.11-0.175.2.10.0.5.0:20150506T023353Z ---
pkg://solaris/install-image/solaris-auto-install@5.11,5.11-0.175.2.9.0.5.0:20150404T222307Z ---
```

Here we use `-v` option here to print full FMRIs (Fault Managed Resource
Indicator) for packages: we will need it later.

Well, we already have created an install service for Solaris 11.3, now
let's create another one for Solaris 11.2, SRU 11, for example.

``` console
root@solarislab:~# installadm create-service -s pkg://solaris/install-image/solaris-auto-install@5.11,5.11-0.175.2.11.0.5.0
OK to use subdir of /export/auto_install to store image? [y|N]: y
  0% : Service svc:/network/dns/multicast:default is not online.  Installation services will not be advertised via multicast DNS.
  0% : Creating service from: pkg://solaris/install-image/solaris-auto-install@5.11,5.11-0.175.2.11.0.5.0
  0% : Using publisher(s):
  0% :     solaris: http://10.80.11.34:81/
  5% : Refreshing Publisher(s)
 15% : Planning Phase
 24% : Download Phase
 62% : Actions Phase
 91% : Finalize Phase
 91% : Creating sparc service: solaris11_2_11_5_0-sparc
 91% : Image path: /export/auto_install/solaris11_2_11_5_0-sparc
 91% : Setting "solaris" publisher URL in default manifest to:
 91% :  http://10.80.11.34:81/
100% : Created Service: 'solaris11_2_11_5_0-sparc'
100% : Refreshing SMF service svc:/system/install/server:default
100% : Warning: mDNS registry of service 'solaris11_2_11_5_0-sparc' could not be verified.
```

OK, done. Now let's make it the default SPARC service.

``` console
root@solarislab:~# installadm delete-service -n default-sparc
WARNING: The service you are deleting, or a dependent alias, is the
alias for the default sparc service. Without the 'default-sparc'
service, sparc clients will fail to boot unless explicitly assigned to
a service using the create-client subcommand.
Are you sure you want to delete this alias? [y|N]: y
Warning: mDNS registry of service default-sparc could not be verified.
Deleted Service: 'default-sparc'
root@solarislab:~# installadm create-service -t solaris11_2_11_5_0-sparc -n default-sparc
  0% : Service svc:/network/dns/multicast:default is not online.  Installation services will not be advertised via multicast DNS.
  0% : Creating sparc alias: default-sparc
  0% : Setting "solaris" publisher URL in default manifest to:
  0% :  http://10.80.11.34:81/
100% : Created Service: 'default-sparc'
100% : Refreshing SMF service svc:/system/install/server:default
100% : Enabling SMF service svc:/network/dhcp/server:ipv4
100% : Warning: mDNS registry of service 'default-sparc' could not be verified.
root@solarislab:~# svcadm disable dhcp/server:ipv4
```

And now you can try to install the same client again to check if this
service installs the version we have created it with, namely, Solaris
11.2.11.

**Spoiler Alert:** We can already tell you what is going to happen
during this test install. If you want to learn it yourself, go ahead and
install the `ai-client` LDom again using the instructions above and then
check what Solaris version was installed with `pkg list entire`. For
those of you who want to save time and skip this test installation step,
here is what's going to happen:

The client system will boot from the install image, which is `11.2.11`,
but it will install the latest available Solaris SRU, which is `11.2.14`
in our default repository. Why so? Because it uses the default manifest
and by default it installs the latest SRU available. If we want to
install that particular SRU, we have to create a new manifest. It's time
to learn about AI manifests.

First, let's see what we've got already:

``` console
root@solarislab:~# installadm list -m
Service Name             Manifest Name Type    Status  Criteria
------------             ------------- ----    ------  --------
default-sparc            orig_default  derived default none
solaris11_2_11_5_0-sparc orig_default  derived default none
solaris11_3-sparc        orig_default  derived default none
```

Note that we have two services (one per each Solaris update version)
plus one alias. Each service has one default manifest. Our plan is to
modify the default manifest and add it to the default service. First, we
copy the manifest XML file from its default location to the home
directory (actually, it can be any directory).

``` console
root@solarislab:~# cp /usr/share/auto_install/manifest/default.xml s11.2.11-manifest.xml
```

Now we can edit this copy. So we want to specify which SRU we are going
to install. Let's list what's available.

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

Suppose we decided to install version `11.2.11`. Take a note of the
version string: `0.5.11-0.175.2.11.0.5.0` -- you will need it when
editing the manifest. Now start your favorite text editor and open the
manifest file:

``` console
root@solarislab:~# vi s11.2.11-manifest.xml
```

Scroll to almost the end of the file and find the following lines:

<pre>
        For instance, to specify a particular build of S11.2, the
        following should be used:

            &lt;name&gt;pkg:/entire@0.5.11,5.11-0.175.2.0.0.build&lt;/name&gt;
      --&gt;
      &lt;software_data action=&quot;install&quot;&gt;
        &lt;name&gt;pkg:/entire@0.5.11-0.175.2&lt;/name&gt;
        &lt;name&gt;pkg:/group/system/solaris-large-server&lt;/name&gt;
      &lt;/software_data&gt;
</pre>

Change them to tell AI server that you want that specific SRU of
Solaris. Also we can change the default 'solaris-large-server' group to
'solaris-small-server' one. After editing it should look like this
(changes are in **bold**):

<pre>
        For instance, to specify a particular build of S11.2, the
        following should be used:

            &lt;name&gt;pkg:/entire@0.5.11,5.11-0.175.2.0.0.build&lt;/name&gt;
      --&gt;
      &lt;software_data action=&quot;install&quot;&gt;
        &lt;name&gt;pkg:/entire@0.5.11-0.175.2<strong>.11.0.5.0</strong>&lt;/name&gt;
        &lt;name&gt;pkg:/group/system/solaris-<strong>small</strong>-server&lt;/name&gt;
      &lt;/software_data&gt;
</pre>

Now we have to install this manifest in the default SPARC service.

``` console
root@solarislab:~# installadm create-manifest -n default-sparc -m s11.2.11 -f ./s11.2.11-manifest.xml
Created Manifest: 's11.2.11'
root@solarislab:~# installadm list -m
Service Name             Manifest Name Type    Status   Criteria
------------             ------------- ----    ------   --------
default-sparc            orig_default  derived default  none
                         s11.2.11      derived inactive none
solaris11_2_11_5_0-sparc orig_default  derived default  none
solaris11_3-sparc        orig_default  derived default  none
```

OK, we have amended the manifest and installed it onto the server, but
how can we use it? How can we let the install server know that which
manifest to use? Well, we can specify a set of criteria to decide which
manifest to apply. We can include IP address, hostname, MAC address in
this criteria and the install server will use certain manifests for
different clients. If the client doesn't fit any criteria, then the
default manifest is applied. Let's define such criteria for our client.

If you remember, before booting the client from OpenBoot prompt we
assigned several boot parameters to it: IP address, hostname, wanboot
script location, etc. So now we can use, for example, its hostname
`ai-client` as a criteria.

``` console
root@solarislab:~# installadm set-criteria -n default-sparc -m s11.2.11 -c hostname=ai-client
Changed Manifest: 's11.2.11'
root@solarislab:~# installadm list -m
Service Name             Manifest Name Type    Status  Criteria
------------             ------------- ----    ------  --------
default-sparc            s11.2.11      derived active  hostname = ai-client
                         orig_default  derived default none
solaris11_2_11_5_0-sparc orig_default  derived default none
solaris11_3-sparc        orig_default  derived default none
```

And now we can try to install the client. Login to the console with
`telnet localhost 5000` (using the port number provided by your
instructor) and repeat the installation process described above.

After you passed all the installation and configuration steps, login
into the client and check it's version:

``` console
root@ai-client:~# pkg list entire
NAME (PUBLISHER)                                  VERSION                    IFO
entire                                            0.5.11-0.175.2.11.0.5.0    i--
```

Success! Let's explain what we have observed here.

1.  By default install service is always created from the latest
    available version of Solaris
2.  For each Solaris Update (i.e. `11.1`, `11.2`, `11.3`) we need a separate
    install service
3.  By default install service installs the latest available SRU for the
    Solaris Update this service is created for (in our case the service
    created from Solaris `11.2.11` will install `11.2.14` unless we
    specify otherwise)
4.  To specify a particular SRU which you want to install you have to
    edit and install AI manifest into the install service

Now we will learn how to customize installation process even further. We
start with adding other software packages.

