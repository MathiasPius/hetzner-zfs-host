# hetzner-zfs-host
These playbooks are used for bringing a Hetzner dedicated server (with 2 or more disks) from the Hetzner Rescue system to semi-useable state.

## Provisioning

It creates 3 zpools:
* **bpool** - contains the boot config (**512m**)
* **rpool** - contains the root filesystem (**8g**)
* **dpool** - data pool, used for anything non-systems-related (**rest of the disk**)

These pools are created in a N-way mirror, where N is the number of disk devices on the server.
All the disks are assumed to be the same size.
The total useable capacity is therefore the size of any one disk, but in return the likelihood of data loss is very low.

Apart from the root disks, the playbook also writes the grub config to each device, so if /dev/sda gets corrupted, it is possible to boot from one of the other devices.

### Partition layout
```
$ lsblk -o NAME,SIZE,TYPE
NAME    SIZE TYPE
sda     1.8T disk
├─sda1    1M part
├─sda2  512M part
├─sda3    8G part
└─sda4  1.8T part
sdb     1.8T disk
├─sdb1    1M part
├─sdb2  512M part
├─sdb3    8G part
└─sdb4  1.8T part
sdc     1.8T disk
├─sdc1    1M part
├─sdc2  512M part
├─sdc3    8G part
└─sdc4  1.8T part
```

### Dataset layout
For an example of the disk layout after successfully provisioning the server, see the following:

```
$ zfs list -o name | tail -n+2 | tree --fromfile
.
├── bpool
│   └── BOOT
│       └── default
├── dpool
│   └── DATA
│       ├── apps
│       │   ├── docker
│       │   ├── sanoid
│       │   └── teamspeak3
│       └── containers
│           ├── 6f6c7dfbfd3c092349432b18d580a33dd872cee8c731ce69b1a43db02b86716b
│           ├── 7ef6a15cc0ef1531842273605515268c04f0aeb6e8a6a82c9b4a4fff21f937f0
│           ├── 91b30c408e6309851e9ff99d6e0101cc7f14d8dd7125353e4cffe08179b54fd7
│           ├── aa3f74965901e5c0fdf0ca2e442cd4d61cda2b32b26eec3967e832d2afdd8300
│           ├── ace706dc9b868973c9eb4ac7a4b0fe874b0c072489716e2ef8ea0ff32d4e7781
│           ├── cf565ab02ee16748ccf22edcf16bbfe7e63fb7ad00a3dfb5b70145ecfc8a98bb
│           └── cf565ab02ee16748ccf22edcf16bbfe7e63fb7ad00a3dfb5b70145ecfc8a98bb-init
└── rpool
    └── ROOT
        └── alpine

8 directories, 12 files
```
Datasets within `apps/` and `containers/` are from running containers, which will obviously not be present after just provisioning the server.

## Configure
This step was ripped from another playbook where containerd was set up in preparation for running k3s, but has been repurposed for this playbook.

This step needs to be expanded, since a lot of the configuration for the server is missing from here, including:

* Installing docker and configuring daemon-remapping, as well as user groups for it
* Configuring the dataset containers for apps/ and containers/
* Setting up sanoid for periodic snapshots of all zpools

# See also
* https://gist.github.com/kongkrit/2d31d58a7656d3b5d3e3ce1a8dda9f56
* https://wiki.alpinelinux.org/wiki/Root_on_ZFS_with_native_encryption
* https://github.com/alpinelinux/alpine-make-vm-image/blob/master/alpine-make-vm-image
* https://ryan.lovelett.me/posts/ubuntu-20-04-root-on-zfs/
* https://docs.oracle.com/cd/E53394_01/html/E54801/gazhv.html#scrolltoc