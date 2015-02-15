# NAME

vboxmake - Create VirtualBox VMs with unattended installation

# SYNOPSIS

    vboxmake --linux-iso CentOS-7.0-1406-x86_64-Minimal.iso --vm-name basebox-centos-7

    Options:

     -l, --linux-iso       ISO image from which to install Linux
     -i, --initrd          the path to initrd.img in `linux-iso`
     -k, --vmlinuz         the path to vmlinuz in `linux-iso`

     -s, --syslinux-targz  .tar.gz file from which to obtain pxelinux.0
     -p, --pxelinux0       the path to pxelinux.0 in `syslinux-targz`

     -n, --vm-name         the name to give to the VM
     -c, --vm-cpus         the number of CPUs to give to the VM
     -r, --vm-memory       the size of the VM RAM in MB
     -o, --vm-ostype       the VirtualBox OS type for this VM
         --vm-vram         the size of the VM VRAM in MB
     -d, --vm-hd-size      the size of the VM hard disk in MB

     -f, --force           overwrite existing VM of same name

     -u, --unattended      the path to the unattended installation script
     -b, --boot-menu       the path to the Syslinux boot configuration menu
     
     -t, --tftp-dir        the path to VirtualBox's TFTP directory
     -v, --vboxmanage      the path to the VBoxManage executable
     -x, --xorriso         the path to the xorriso executable
         --xorrisofs       the path to the xorrisofs executable

     -h, --help            brief help message
     -m, --man             full documentation

# OPTIONS

- -l, --linux-iso

    `vboxmake` will attempt to extract `initrd.img` and `vmlinuz` from this ISO
    image.  It will also attach this ISO image to the IDE storage controller for
    use as the installation media.

- -i, --initrd

    Default: `/images/pxeboot/initrd.img`

    This is the location of the `initrd.img` file in the Linux ISO.

- -k, --vmlinuz

    Default: `/images/pxeboot/vmlinuz`

    This is the location of the `vmlinuz` file in the Linux ISO.

- -s, --syslinux-targz

    Default: `${HOME}/vboxmake/syslinux-4.07.tar.gz`

    `vboxmake` will attempt to extract `pxelinux.0` from this .tar.gz file.
    It is highly recommended that Syslinux version 4.0.7 is used, as I have
    been unable to get later versions working correctly with VirtualBox.

- -p, --pxelinux0

    Default: `syslinux-4.07/core/pxelinux.0`

    This is the location of `pxelinux.0` in the Syslinux .tar.gz file.

- -n, --vm-name

    Passed to `VBoxManage createvm --name`.

- -c, --vm-cpus

    Default: `2`

    Passed to `VBoxManage modifyvm --cpus`

- -r, --vm-memory

    Default: `1024`

    Passed to `VBoxManage modifyvm --memory`

- -o, --vm-ostype

    Default: `RedHat_64`

    Passed to `VBoxManage modifyvm --ostype`

- --vm-vram

    Default: `10`

    Passed to `VBoxManage modifyvm --vram`

- -d, --vm-hd-size

    Default: `28610`

    Passed to `VBoxManage createhd --size`

- -f, --force

    If `vboxmake` detects a VirtualBox VM with the same name as `--vm-name`,
    it will abort.  Specifying `--force` will cause the VM and any associated
    configuration files to be overwritten.

- -u, --unattended

    Default: `<vboxmake dir>/tftpboot/ks.cfg`

    This is the unattended installation file that will be used to script the
    installation of your VM.  The default script is supplied for testing
    purposes; you'll probably want to create your own.

- -b, --boot-menu

    Default: `<vboxmake dir>/tftpboot/boot_menu`

    This is the Syslinux configuration that is used to bootstrap the installation
    process.  The default file skips the menu altogether, and simply starts the
    installation.  You probably don't want to change this.

- -t, --tftp-dir

    Default: `${HOME}/Library/VirtualBox/TFTP`

    When the first NIC is of type NAT, VirtualBox will look in the TFTP directory
    for a file with the name `--vm-name` and the extension `.pxe`.  For example,
    if the VM name is `basebox`, the TFTP boot process would look for a file
    called `basebox.pxe` in the TFTP directory.  `vboxmake` creates the
    directory and the file automatically.  You should only need to change this
    setting if your installation of VirtualBox is using a different location
    for the TFTP directory.

- -v, --vboxmanage

    Default: `/usr/bin/VBoxManage`

    This is the path to the `VBoxManage` executable.

- -x, --xorriso

    Default: `/usr/local/bin/xorriso`

    This is the path to the `xorriso` executable.

- --xorrisofs

    Default: `/usr/local/bin/xorrisofs`

    This is the path to the `xorrisofs` executable.

# DESCRIPTION

`vboxmake` is a Perl script designed to help automate the production of
VirtualBox base boxes.  It's a work in progress, and has been developed
specifically for use on OS X.  It is currently only suitable for installing
CentOS 7.0, but it probably wouldn't require much effort to make it install
other Linux distros.  Patches and pull requests are welcome...

## The build process

- The built-in VirtualBox PXE-enabled DHCP server supplies a new VM with an
IP address, and specifies the path to the bootloader file.  This path is in
the format \`<vm name>.pxe\`, so there must be one file per VM.
- Because VirtualBox uses [iPXE](http://ipxe.org/) firmware, we replace the
standard \`pxelinux.0\` file with an [iPXE script](http://ipxe.org/scripting).
The script sets the TFTP prefix to the name of the VM, thus allowing separate
PXE configurations to be created for different VMs.  The script then passes
the location of the real \`pxelinux.0\` bootloader file to iPXE.  \`pxelinux.0\`
is obtained from the [Syslinux PXELINUX](http://www.syslinux.org/wiki/index.php/PXELINUX)
project.
- The PXE boot process loads \`pxelinux.0\`, which in turn loads the syslinux
configuration menu file.  The config file provides the necessary kernel
parameters, including the location of \`initrd.img\`, \`vmlinuz\` and the
Kickstart configuration file.
- As part of the build process, an ISO file is generated that contains the
Kickstart configuration file.  This ISO is attached to the VM as a secondary
DVD-ROM drive so that the boot process can load it.  This negates the need
to either build a custom \`initrd.img\` (which is rather difficult on OS X),
or to have the file available over HTTP.
- Finally, the Kickstart configuration takes over, and installs the system.

# PREREQUISITES

## xorriso

xorriso is used to extract \`initrd.img\` and \`vmlinuz\` from the Linux
distribution, and to create the ISO CD-ROM image that contains the Kickstart
configuration file.  Installation is easy:

    cd /tmp
    curl -O http://www.gnu.org/software/xorriso/xorriso-1.3.9.tar.gz

    tar xvzf xorriso-1.3.9.tar.gz
    cd xorriso-1.3.9

    ./configure
    make
    sudo make install

## Linux installation ISO

`vboxmake` has so far only been tested with CentOS 7.0 minimal.

Download a copy of the installation ISO, and place it somewhere handy.
There's no need to manually extract anything from the ISO.

## Syslinux

`vboxmake` needs a copy of `pxelinux.0` from the Syslinux PXELINUX project.
It is highly recommended that Syslinux version 4.0.7 is used, as I have
been unable to get later versions working correctly with VirtualBox.
`vboxmake` will attempt to extract `pxelinux.0` from this .tar.gz file.

# NOTES

To update the README from this POD, install [Pod::Markdown](https://metacpan.org/pod/Pod::Markdown), and then:

    perldoc -Tu ~/git_clones/virtualbox_base_box_builder/virtualbox/vboxmake | \
      pod2markdown > README.md
