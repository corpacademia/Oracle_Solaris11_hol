Application packaging with IPS
------------------------------

The Image Packaging System (IPS) is the network based package management
system included in Oracle Solaris 11. It provides a framework for
complete software lifecycle management such as installation, upgrade and
removal of software packages. IPS takes advantage of ZFS using ZFS boot
environments, such that administrators can update a system while running
production services and then boot into the new boot environment when a
planned maintenance window comes along. IPS uses package repositories to
install software over the network, and provides for automatic package
dependencies.

In this session we will be taking the zoneplot, a command line utility
for measuring and plotting zone statistics, and creating an IPS package
for it that we can install onto the system. This is a very simple
example to give you an overview of the packaging process, but it's
recommended you take at look at the [IPS Developer
Guide](http://docs.oracle.com/cd/E26502_01/html/E21383/index.html).

For this package we will need two scripts. First accepts a Unix pipe
stream and shows its data using gnuplot. Second script converts the
zonestat outpout into a stream readable by the first script. Both
scripts are publicly available. Download them from the Internet:

    root@solaris:~# wget http://www.lysium.de/sw/driveGnuPlotStreams.pl
    root@solaris:~# wget http://zoneplot.googlecode.com/files/zoneplot

IPS uses package manifests to describe how a package is assembled - what
files or directories are included, what dependencies the package has,
and basic information about a package such as name, version and
description. Fortunately we can generate much of this manifest using IPS
and pointing it to a location, or proto area, which contains all the
files and directories that we want to include in this package. We'll
start by creating this proto area and laying things out how we'd expect
the package to look like:

    root@solaris:~# mkdir -p PROTO/usr/bin

In this case we're creating a PROTO directory in \$HOME. Within that
directory, we have a /usr/bin layout. We'll copy our scripts into this
proto area and make them executable:

    root@solaris:~# cp zoneplot driveGnuPlotStreams.pl PROTO/usr/bin
    root@solaris:~# chmod a+x PROTO/usr/bin/*

We can see in the following graphic how our directory is now laid out

      
     zoneplot     driveGnuPlotStreams.pl
          \                   /
                  bin
                   |
                  usr
                   |
                   PROTO

Let's generate an initial manifest:

    root@solaris:~# pkgsend generate PROTO > zoneplot.p5m.gen

Let's look at the contents:

    root@solaris:~# cat zoneplot.p5m.gen
    dir group=bin mode=0755 owner=root path=usr
    dir group=bin mode=0755 owner=root path=usr/bin
    file usr/bin/driveGnuPlotStreams.pl group=bin mode=0755 owner=root path=usr/bin/driveGnuPlotStreams.pl
    file usr/bin/zoneplot group=bin mode=0755 owner=root path=usr/bin/zoneplot

As you'll see here there are two sets of 'actions' - two directory
actions that describe usr and usr/bin directories, and two file actions
that describe `zoneplot` and `driveGnuPlotStreams.pl`. These are the
contents of our package. Usual IPS practice encourages us to not include
directory actions that are already defined on the system. We will remove
them.

    root@solaris:~# vi zoneplot.p5m.gen
    root@solaris:~# cat zoneplot.p5m.gen
    file usr/bin/driveGnuPlotStreams.pl group=bin mode=0755 owner=root path=usr/bin/driveGnuPlotStreams.pl
    file usr/bin/zoneplot group=bin mode=0755 owner=root path=usr/bin/zoneplot

The next thing that we need to do is to add some basic meta-information
about the package - including package name, package version and package
description. Add the following lines to our manifest:

    root@solaris:~# vi zoneplot.p5m.gen
    root@solaris:~# cat zoneplot.p5m.gen
    set name=pkg.fmri value=zoneplot@1.0,5.11-0
    set name=pkg.summary value="zoneplot utility"
    set name=pkg.description value="Utility to plot the output of zone statistics"
    set name=info.classification value="org.opensolaris.category.2008:Applications/System Utilities"
    file usr/bin/driveGnuPlotStreams.pl group=bin mode=0755 owner=root path=usr/bin/driveGnuPlotStreams.pl
    file usr/bin/zoneplot group=bin mode=0755 owner=root path=usr/bin/zoneplot

IPS has automatic package dependency checking for all package management
operations on the system. For example, if a package requires other
packages to work it will automatically install these. We will do the
same for zoneplot. We will first try and detect what package
dependencies this package might have using the pkgdepend generate
command:

    root@solaris:~# pkgdepend generate -md PROTO zoneplot.p5m.gen > zoneplot.p5m.dep
    root@solaris:~# cat zonepot.p5m.dep
    set name=pkg.fmri value=zoneplot@1.0,5.11-0
    set name=pkg.summary value="zoneplot utility"
    set name=pkg.description value="Utility to plot the output of zone statistics"
    set name=info.classification value="org.opensolaris.category.2008:Applications/System Utilities"
    file usr/bin/zoneplot path=usr/bin/zoneplot owner=root group=bin mode=0755
    file usr/bin/driveGnuPlotStreams.pl path=usr/bin/driveGnuPlotStreams.pl owner=root group=bin mode=0755
    depend fmri=__TBD pkg.debug.depend.file=perl pkg.debug.depend.path=usr/bin pkg.debug.depend.reason=usr/bin/driveGnuPlotStreams.pl pkg.debug.depend.type=script type=require
    depend fmri=__TBD pkg.debug.depend.file=bash pkg.debug.depend.path=usr/bin pkg.debug.depend.reason=usr/bin/zoneplot pkg.debug.depend.type=script type=require

You'll see that we have detected that there are two files in this
package (zoneplot and driveGnuPlotStreams.pl) that have external
dependencies. zoneplot is a bash script and thus depends on bash.
driveGnuPlotStreams.pl is a perl script and thus depends on perl. While
both of these are already installed by default on Oracle Solaris 11,
it's good practice to make sure we define these dependencies.

The next step is resolving these dependencies into the packages they are
delivered in. We need to determine what packages bash and perl are a
part of. We use the pkgdepend resolve command to achieve this.

    root@solaris:~# pkgdepend resolve -m zoneplot.p5m.dep
    root@solaris:~# cat zoneplot.p5m.dep.res (NOTE the .res extension!)
    set name=pkg.fmri value=zoneplot@1.1,5.11-0
    set name=pkg.summary value="zoneplot utility"
    set name=pkg.description value="Utility to plot the output of zone statistics"
    set name=info.classification value="org.opensolaris.category.2008:Applications/System Utilities"
    file usr/bin/zoneplot path=usr/bin/zoneplot owner=root group=bin mode=0755
    file usr/bin/driveGnuPlotStreams.pl path=usr/bin/driveGnuPlotStreams.pl owner=root group=bin mode=0755
    depend fmri=pkg:/runtime/perl-512@5.12.4-0.175.1.0.0.24.0 type=require
    depend fmri=pkg:/shell/bash@4.1.9-0.175.1.0.0.24.0 type=require

We can see that these dependencies have been resolved to
pkg:/runtime/perl-512 and pkg:/shell/bash. While IPS does a good job at
trying to detect dependencies by looking at files (ELF headers, \#!
script definitions, etc.), sometimes we will have to manually provide
additional dependencies. In this case, we know that this utility depends
on gnuplot to plot the statistics graphically, so we will add this to
our manifest.

First we have to figure out the gnuplot's FMRI:

    root@solaris:~# pkg search -o pkg.shortfmri gnuplot
    PKG.SHORTFMRI
    pkg:/image/gnuplot@4.6.0-0.175.1.0.0.24.0

Now copy this FMRI into the `zoneplot.p5m.dep.res` file as following:

    root@solaris:~# vi zoneplot.p5m.dep.res
    root@solaris:~# cat zoneplot.p5m.dep.res
    set name=pkg.fmri value=zoneplot@1.1,5.11-0
    set name=pkg.summary value="zoneplot utility"
    set name=pkg.description value="Utility to plot the output of zone statistics"
    set name=info.classification value="org.opensolaris.category.2008:Applications/System Utilities"
    file usr/bin/zoneplot path=usr/bin/zoneplot owner=root group=bin mode=0755
    file usr/bin/driveGnuPlotStreams.pl path=usr/bin/driveGnuPlotStreams.pl owner=root group=bin mode=0755
    depend fmri=pkg:/runtime/perl-512@5.12.4-0.175.1.0.0.24.0 type=require
    depend fmri=pkg:/shell/bash@4.1.9-0.175.1.0.0.24.0 type=require
    depend fmri=pkg:/image/gnuplot@4.6.0,5.11-0.175.1.0.0.24.0 type=require

Now that we have our PROTO area and manifest completed, it is now time
to publish this package to an IPS repository. First we will need to
create a repository to host this package - it's best practice not to
publish to the existing solaris default publishers even if you have a
copy of them locally.

Let's first create a ZFS data set to host our repository:

    root@solaris:~# zfs create -o mountpoint=/repository rpool/repository

And now create an IPS package repository there:

    root@solaris:~# pkgrepo create /repository

We will need to set the publisher prefix of this repository so that we
can add it to our publisher configuration on our system. We will call it
'zoneplot':

    root@solaris:~# pkgrepo -s /repository set publisher/prefix=zoneplot

Now we are ready to publish our package:

    root@solaris:~# pkgsend -s /repository publish -d PROTO zoneplot.p5m.dep.res
    pkg://zoneplot/zoneplot@1.0,5.11-0:20130524T160043Z
    PUBLISHED

We can confirm this by taking a look at the status of the repository:

    root@solaris:~# pkgrepo -s /repository info
    PUBLISHER  PACKAGES  STATUS           UPDATED
    zoneplot         1                      online                2013-05-24T16:00:43.695049Z

We can query the package now within the repository:

    root@solaris:~# pkg info -g /repository zoneplot
              Name: zoneplot
           Summary: zoneplot utility
       Description: Utility to plot the output of zone statistics
          Category: Applications/System Utilities
             State: Not installed
         Publisher: zoneplot
           Version: 1.0
     Build Release: 5.11
            Branch: 0
    Packaging Date: May 24, 2013 04:00:43 PM
              Size: 6.04 kB
              FMRI: pkg://zoneplot/zoneplot@1.2,5.11-0:20130524T160043Z

Let's go ahead and add this publisher to our configuration:

    root@solaris:~# pkg set-publisher -p /repository
    root@solaris:~# pkg publisher
    PUBLISHER            TYPE      STATUS P LOCATION
    solaris                        origin     online      F http://pkg.oracle.com/solaris/release
    zoneplot                       origin     online      F file:///repository

And now finally install zoneplot (you will see that it will pull in a
number of package dependencies):

    root@solaris:~# pkg install zoneplot
