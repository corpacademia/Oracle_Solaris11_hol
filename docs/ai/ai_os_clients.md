If you remember, first we have created an install service with the
latest Oracle Solaris version available in our repository, which was
version `11.3`. Then we changed the default to `11.2.11`, but what if we
wanted to install a client with version `11.3`? We now have two install
services configured, one for `11.2.11` (which is the default) and another
for `11.3`. How can we specify that we want to install `11.3` instead of
`11.2`? You may think we must use the same mechanism we used to install a
specific Solaris SRU, namely manifest. Well, you may try to create a
manifest XML file and specify `11.3` instead of `11.2`, but we can tell you
(SPOILER ALERT!!!) that this won't work. The rule is that changing the
manifest is not enough if you want to install a different Solaris
*update* (i.e. second digit as in `11.1`, `11.2`, `11.3`...). In this case you
should use a different *install service*. Luckily, we have one installed
already, its name is `solaris11_3-sparc`.

If we want to specify the client that we want to install `11.3` on, we
have to use `create-client` command. With this command we can
associate specific clients with certain install services. Clients should
be specified using their MAC addresses. Login into the client system and
discover its MAC address:

``` console
root@ai-client1:~# dladm show-phys -m
LINK                SLOT     ADDRESS            INUSE CLIENT
net0                primary  0:14:4f:f8:5:f6    yes   net0
                    1        0:14:4f:f8:3:5c    no    --
                    2        0:14:4f:f8:ee:26   no    --
                    3        0:14:4f:fa:c4:6f   no    --
```

The primary MAC address is `0:14:4f:f8:5:f6`. Now switch to the AI server
window and use this address to associate it with the install service
`solaris11_3-sparc`.

``` console
root@solarislab:~# installadm create-client -e 0:14:4f:f8:5:f6 -n solaris11_3-sparc
Created Client: '00:14:4F:F8:05:F6'
root@solarislab:~# installadm list -c
Service Name      Client Address    Arch  Secure Custom Args Custom Grub
------------      --------------    ----  ------ ----------- -----------
solaris11_3-sparc 00:14:4F:F8:05:F6 sparc no     no          no
```

And now, again the familiar procedure of shutting down the client and
installing it from network.

When this newly installed system reboots, you will notice that you get
the system configuration screen again and you have to specify hostname,
IP address, and other parameters again. Why? We have configured the
system profile already to automate this process, why are we seeing this
again? Take a look at the profiles list:

``` console
root@solarislab:~# installadm list -p
Service Name  Profile Name Environment Criteria
------------  ------------ ----------- --------
default-sparc sc-template  system      none
```

We have created the profile `sc-template` and associated it with the
`default-sparc` install service. And `default-sparc` install service
is an alias of the `s11.2.11-sparc` service. So, profiles are
associated with install services. As soon as we are using a different
install service, we are using it's default profile, not the one we
created earlier. You can use the XML file `sc_template.xml` and create a
new profile with 11.3 service&mdash;you know how to do this already.

One more question: what if we don't have Solaris installed on the
client? Like if we just have created a guest logical domain and it's
completely empty? How can we figure out its MAC address?

Well, it takes a couple of steps. Halt the client system and get to
OpenBoot prompt. Enter the following command:

```
{0} ok show-devs
/cpu@7
/cpu@6
/cpu@5
/cpu@4
/cpu@3
/cpu@2
/cpu@1
/cpu@0
/virtual-devices@100
/iscsi-hba
/virtual-memory
/memory@m0,80000000
/aliases
/options
/openprom
/chosen
/packages
/virtual-devices@100/channel-devices@200
/virtual-devices@100/console@1
/virtual-devices@100/random-number-generator@e
/virtual-devices@100/flashprom@0
/virtual-devices@100/channel-devices@200/virtual-domain-service@0
/virtual-devices@100/channel-devices@200/pciv-communication@0
/virtual-devices@100/channel-devices@200/disk@0
/virtual-devices@100/channel-devices@200/network@0
/iscsi-hba/disk
/openprom/client-services
/packages/obp-tftp
/packages/kbd-translator
/packages/SUNW,asr
/packages/dropins
/packages/terminal-emulator
/packages/disk-label
/packages/deblocker
/packages/SUNW,builtin-drivers
{0} ok
```

Find the line that represents the network interface. In our case it's
`/virtual-devices@100/channel-devices@200/network@0`. Then 'change
directory' (`cd`) into it and type `.properties`:

```
{0} ok cd /virtual-devices@100/channel-devices@200/network@0
{0} ok .properties
local-mac-address        00 14 4f f8 05 f6
max-frame-size           00004000
address-bits             00000030
reg                      00000000
compatible               SUNW,sun4v-network
device_type              network
name                     network
{0} ok
```

Got it! Here is our client's MAC address: `00:14:4f:f8:05:f6`. You can use
it with `create-client` command now and install the system from scratch.

