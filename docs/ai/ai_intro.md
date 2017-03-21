In this lab we will learn how to use the new installation method
introduced in Oracle Solaris 11: Automated Installer. We will review
several typical situations and discuss how to solve them using Automated
Installer (AI). This lab includes the following scenarios:

-   Simple installation: one CPU architecture, one Solaris version, one
    set of packages;
-   Several Solaris versions to be installed from one AI server;
-   Different package configurations: small-server, large-server,
    additional packages;
-   Different host configurations: hostnames, IP addresses, etc.;
-   Zone installation from AI server;
-   Installing from Unified Archives

But before we start, 60 seconds of theory.

!!! note "60 seconds of theory"
    Automated Installer has three major components: install services,
    manifests and system profiles. Here is what they do:

    -   **Install Services:** they provide bootable media for clients and
        work in conjunction with IPS repositories. You need one install
        service per each CPU architecture and OS version.
    -   **AI Manifests:** they describe installation configuration (disk
        layout etc.) and package sets. You need one manifest per each set of
        packages (small-server, large-server, additional packages) and
        disk layout. You can modify them dynamically at the time
        of installation.
    -   **System Configuration (SC) Profiles:** they describe host OS
        configuration including hostname, network parameters, time zone,
        passwords etc. They are optional: if you don't have one, you'll have
        to specify all these parameters at first boot.

Automated Installer works in conjunction with IPS repository, so we will
need one for our exercises. Luckily, we have one installed in our lab.
You instructor will give you all necessary information.


