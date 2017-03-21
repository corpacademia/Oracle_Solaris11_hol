Most likely you want not just plain vanilla `small-server` Solaris
installation, but you are interested in some additional packages that
can be installed. Instead of installing them manually or via custom
scripts, why don't we specify them in the manifest? As you might have
guessed already, if we can specify the version of `entire` package in
the `software_data` part of the manifest, we can add more packages the
same way, by using their FMRI names. Let's imagine we want to add a new
management platform Puppet which was added recently to Oracle Solaris
11.2. If Puppet is installed from the beginning in the system, it can
make system management much easier. But that's a topic for another
hands-on lab.

First, we have to find Puppet package we want to install.

``` console
root@solarislab:~# pkg list -av '*puppet*'
FMRI                                                                         IFO
pkg://solaris/system/management/puppet@3.6.2,5.11-0.175.2.5.0.2.0:20141114T212642Z ---
pkg://solaris/system/management/puppet-19@3.6.2,5.11-0.175.2.5.0.2.0:20141114T212630Z ---
```

Copy the first package's FMRI until the `@` sign into the clipboard or
Notepad. Open the manifest file and edit it. After the line
with `solaris-small-server` FMRI insert the line with Puppet's FMRI
(shown in bold).

<pre>
  &lt;software_data action=&quot;install&quot;&gt;
    &lt;name&gt;pkg:/entire@0.5.11-0.175.2.11.0.5.0&lt;/name&gt;
    &lt;name&gt;pkg:/group/system/solaris-small-server&lt;/name&gt;
    <b>&lt;name&gt;pkg://solaris/system/management/puppet&lt;/name&gt;</b>
  &lt;/software_data&gt;
</pre>

Don't forget to update the manifest after changing the XML file!

``` console
root@solarislab:~# installadm update-manifest -n default-sparc -m s11.2.11 -f ./s11.2.11-manifest.xml
Changed Manifest: 's11.2.11'
```

The rest is easy. Get back to OpenBoot prompt on the client (use `init
0` for that) and install it again (with `boot net - install`). After
configuring the system (hostname, IP, time zone, etc.) login via console
and check if you have installed the right version of Solaris and the
Puppet package.

<pre>
    ai-client console login: <b>root</b>
    Password: <b>solaris1</b>
    Sep 22 17:43:17 ai-client login: ROOT LOGIN /dev/console
    Oracle Corporation      SunOS 5.11      11.2    May 2015
    root@ai-client:~# <b>pkg list entire</b>
    NAME (PUBLISHER)                                  VERSION                    IFO
    entire                                            0.5.11-0.175.2.11.0.5.0    i--
    root@ai-client:~# <b>pkg list '*puppet*'</b>
    NAME (PUBLISHER)                                  VERSION                    IFO
    system/management/puppet                          3.6.2-0.175.2.5.0.2.0      i--
    system/management/puppet-19                       3.6.2-0.175.2.5.0.2.0      i--
</pre>

Success! Let's summarize what we have learned in this exercise:

1.  AI manifest controls what software is going to be installed on
    the client. You can add packages that are available from your
    IPS repository.
2.  Don't forget to update the manifest in the install service after you
    have edited the XML file.
3.  You can specify which clients should use each particular manifest by
    creating criteria based on hostname, IP address, MAC address, etc.
    IMPORTANT: in this criteria AI uses the hostname and IP address that
    were set in `network-boot-parameters` by OpenBoot, not the
    hostname/IP you *intend* to use after installation. Usually they are
    the same, but could be different.

