There is a new virtualization technology called "Kernel Zones" which
became available in Oracle Solaris 11.2. In this lab we will explore its
new features and compare it to other Oracle virtualization technologies.

!!! note "60 seconds of theory"
    Solaris Zones were first announced in 2005 in
    Solaris 10 and then became a popular way of managing application
    workloads. They are very flexible, extremely lightweight and provide
    very good isolation of resources. But Solaris Zones require that all
    applications in every zone run under the same version of OS Solaris,
    which runs in the global zone. In some situations it might become an
    unnecessary restriction. To answer this limitation, Solaris engineers
    created a new virtualization technology which allows you to run zones
    with different versions of Solaris under one global zone. This
    technology is now called "kernel zones" because they run their own
    kernels. We continue calling the earlier Solaris Zones technology
    "non-global" zones.

In this lab we will compare various features of kernel zones and
non-global zones side-by-side and analyze how and where we can use them.
Please pay special attention to the command prompts: they indicate where
we perform the operation. Our global zone is called `solarislab`;
kernel zones will be called `kzone1`, `kzone2`, etc.; traditional
non-global zones are called `zone1`, `zone2`, etc.

