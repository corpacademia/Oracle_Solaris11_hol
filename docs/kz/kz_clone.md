Sometimes we want to create a zone, install and configure applications
in it and use it as a template: whether it's for scaling out the
application (e.g. load balancing), or for distributing it to other
locations, or for using it for testing and training purposes. There are
several different ways to do that. You can use different technologies
for that:

-   zone cloning feature (`zoneadm clone` command)
-   Automated Installer mechanisms (manifests and profiles)
-   Unified Archives (available since `11.2`)

In this lab we will explore the third method as it's the most flexible one.

So, we have a kernel zone already and we want to create a clone of it.
First, we create a Unified Archive:

``` console
root@solarislab:~# archiveadm create /share1/uar/kzone2.uar -z kzone2
```

Then we create a new kernel zone with zonecfg, but we use the UAR as a
template:

``` console
root@solarislab:~# zonecfg -z kzone3 create -a /share1/uar/kzone2.uar -z kzone2
```

And then install the zone from that UAR:

``` console
root@solarislab:~# zoneadm -z kzone3 install -a /share1/uar/kzone2.uar -z kzone2
```

That's it! Now you are going to boot the zone, login to its console and
go through the initial configuration process. Remember, when installing
from a UAR, we remove all individual parameters like hostname, IP
address and root password. Sometimes it's more convenient to create this
profile beforehand with `sysconfig create-profile` and then use it
during installation. In that case, your new zone will be ready to use
after the first boot.

