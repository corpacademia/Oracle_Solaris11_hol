One of the very important additions to Oracle Solaris in version `11.2`
was Unified Archives. It gives a lot of options on systems backup and
restore, systems cloning including various use cases: with or without
zones, from physical to virtual and back, etc. In the following exercise
we will use a UAR (Unified ARchive) file with Oracle Solaris
distribution which is normally available for download from `oracle.com`.
Alternatively, you can create you own archive from the existing system
using `installadm(1M)`. Unified archive preserves everything that was
installed in the system, including zones, so it's a perfect way of
cloning systems with applications.

The plan is pretty simple:

1.  Copy the UAR file to the AI webserver's files location (it should be
    visible by the client)
2.  Copy and edit the `default_archive.xml` manifest file to show the
    archive's location (in URL format)
3.  Install the manifest into one of the install services
4.  Define the client criteria to install from this archive (or make
    it default)

We have several UAR files in `/share1/uar`:

``` console
root@solarislab:~# ls -lh /share1/uar
total 15405066
-rw-r--r--   1 nobody   nobody      4.7G Jun 29  2014 sol-11_2-openstack-sparc.uar
-rw-r--r--   1 nobody   nobody      1.3G Jul 18  2014 sol-11_2-sparc.uar
-rw-r--r--   1 nobody   nobody      887M Sep 28 09:57 sol-11_3_1_3_0-sparc.uar
-rw-r--r--   1 nobody   nobody      916M Jul  8 16:01 sol-11-3-beta.uar
```

Remember, we have configured the AI web server directory to store files,
it's `/var/ai/ai-files/`. Copy one of the UAR files to that directory:

``` console
root@solarislab:~# cp /share1/uar/sol-11_2-sparc.uar /var/ai/ai-files
root@solarislab:~# chown webservd:webservd /var/ai/ai-files/sol-11_2-sparc.uar
```

Now we have to edit the manifest XML. Remember, the local web server
directory is `/var/ai/ai-files`, but from outside it looks like
`http://10.80.11.34:5555/files`. Copy the default archive manifest file
to the current directory and edit it.

``` console
root@solarislab:~# cp /usr/share/auto_install/manifest/default_archive.xml ./
root@solarislab:~# chmod 644 default_archive.xml
root@solarislab:~# vi default_archive.xml
```

Find the line with UAR location: `<file uri="file:///.cdrom/archive.uar"/>` and replace is with the following:

            <file uri="http://10.80.11.34:5555/files/sol-11_2-sparc.uar"/>

Now add this manifest to the existing install service:

``` console
root@solarislab:~# installadm create-manifest -n default-sparc -m archive -c hostname=ai-client1 -f ./default_archive.xml
Created Manifest: 'archive'
root@solarislab:~# installadm list -m
Service Name             Manifest Name Type    Status   Criteria
------------             ------------- ----    ------   --------
default-sparc            archive       xml     active   hostname = ai-client1
                         s11.2.11      xml     default  none
                         orig_default  derived inactive none
solaris11_2_11_5_0-sparc orig_default  derived default  none
solaris11_3-sparc        orig_default  derived default  none
```

In the command above we also specified the client criteria that defines
which client(s) will be installed with this manifest. We use the same
client system and if you remember, last time we specified its hostname
in `network-boot-parameters` as `ai-client1`. You may also remember
that we specified its MAC address with `create-client` command and
assigned it to use Solaris `11.3` install service. This time we want to
install from `default-sparc` install service (as soon as we have
installed the manifest into it) so we have to remove this client
association.

``` console
root@solarislab:~# installadm list -c
Service Name      Client Address    Arch  Secure Custom Args Custom Grub
------------      --------------    ----  ------ ----------- -----------
solaris11_3-sparc 00:14:4F:F8:05:F6 sparc no     no          no
root@solarislab:~# installadm delete-client -e 00:14:4F:F8:05:F6
Deleted Client: '00:14:4F:F8:05:F6'
root@solarislab:~# installadm list -c
There are no clients configured for local services.
```

Everything is ready. Now we can try to install the client and check
which Solaris version was installed. If everything goes well it should
be `11.2.0` (a.k.a. General Availability release).

``` console
ai-client1 console login: lab
Password: oracle1
lab@ai-client1:~$ pkg list entire
NAME (PUBLISHER)                                  VERSION                    IFO
entire                                            0.5.11-0.175.2.0.0.42.0    i--
```

Success! Did you notice that it was also faster that installing from a
repository? That's because we are sending a long stream of data instead
of installing the system package by package. Again, let's wrap up what
we have learned.

1.  We can install systems not only from the repository, but also from
    Unified Archives, essentially cloning the servers.
2.  Installation from UAR usually goes faster than from
    package repository.
3.  Don't forget to specify criteria that define which clients will
    be installed. You can specify it either during manifest
    creation/update or as a separate command.

