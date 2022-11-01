---
title: Booting Ubuntu VM on top of KVM-Qemu Hypervisor
# sub-title: Model Serving
layout: post
tags: [linux, virtual machine, operating systems]
mathjax: true
showavatar: false
cover-img: /static/img/linux-vm-cover.png
social-share: false
readtime: true
---

**Post Goal:**

This blog post summarizes how to compile and run a customized Linux kernel on a KVM-qemu virtual machine. It specifies:
1. How to run a VM in KVM using a cloud image
2. How to use a customized Linux kernel in place of the default shipped cloud image kernel
3. How to use GDB with a customized Linxux kernel in KVM.

**Hardware & Software Specs:**
The specs of my test environment are: 
- CPU: Intel(R) Xeon(R) CPU E5-2620 v4 @ 2.10GHz
- Architecture: x86_64
- Total RAM: 125 GB
- Swap: 8.0 GB
- Host OS: Ubuntu 22.04.1 LTS
- Guest (VM) OS: Ubuntu 20.04 LTS Focal
- Host Kernel Version: Linux 5.15.0-47-generic
- Guest Kernel Version: Linux 5.19.7
- QEMU emulator version: 6.2.0

**Booting a VM in KVM w/o Customized Linux Kernel:**

There are a variety of resources online that suggest that you use `virt-install` and `virsh` to set KVM-based VM. At first, I tried using these 
utilities as Ill, and with an Ubuntu ISO image I did reach a point where I launched a VM that I could `ssh` into. However, using `virsh` to successfully
shutdown and/or startup the VM was unsuccessful. Therefore, I pivoted to using the `qemu-system-x86_64` command instead, which does not abstract 
away as much of the VM management details like `virt-install` and `virsh`. 

Initially, I attempted to use an [Ubuntu ISO image](https://ubuntu.com/download/desktop) with `qemu-system-x86_64`. However, I struggled to successfully boot the VM. This is because the recent releases of Ubuntu ship with a graphical installer, which made it difficult to use since I was using a remote server as our host machine (it is strange I successfully used the ISO with `virt-intall` without X-forwarding but found tremendous difficulty with it when using `qemu-system-x86_64`). As such, I ultimately (and successfully) used a cloud image to boot our VM on the zerberus remote server. 

To guide the use of a cloud image, I used this (great blog post)[https://powersj.io/posts/ubuntu-qemu-cli/] about cloud image and `cloud-init`. I used the Ubuntu 20.04 (LTS Focal cloud disk image)[https://cloud-images.ubuntu.com/focal/current/]. Unlike the Ubuntu ISO, a cloud config image is also needed to specify who can login to the virtual cloud server. This is because when VM instances are launched in cloud, `cloud-init` searches for a datasource to obtain instance metadata. Since I was launching a local QEMU image, I provided a local data source for the cloud image to read from. There is a plethora of configurations that can be set, but the purposes of this tutorial I provided minimal configurations, as shown below: 

```console
$ cat metadata.yaml 
instance-id: iid-local01
local-hostname: cloudimg

$ cat user-data.yaml 
#cloud-config
ssh_authorized_keys:
  - ssh-rsa <rsa key from id_rsa.pub>
```
The `metadata.yaml` file specifies configuration data for the VM itself, whereas `user-data.yaml` specifies the ssh key that I generated on my machine for my user (psinha) to be able to `ssh` into the VM without a password. To generate the cloud image that combines the data from both files, you can run
```console
$ cloud-localds seed.img user-data.yaml metadata.yaml
```
QEMU defaults to using the SeaBIOS firmware for bootup. I used this open-source BIOS as well. The full QEMU command I used to boot the VM is:
```console
$ sudo qemu-system-x86_64 \
    -machine accel=kvm,type=q35 \
    -cpu host \
    -m 4G \
    -smp 4 \
    -nographic \
    -device virtio-net-pci,netdev=net0 \
    -netdev user,id=net0,hostfwd=tcp::10022-:22 \
    -drive if=virtio,format=qcow2,file=~/focal-server-cloudimg-amd64.img \
    -drive if=virtio,format=raw,file=seed.img
```
Here's a breakdown of the command:
- `-machine accel=kvm`,type=q35 enables KVM acceleration, which improves VM performance compared to using QEMU.
- `-cpu host` selects the CPU model and passes host processor features to the guest 
- `-m 4G` sets the amount of memory for the VM instance.
- `-smp 4` sets the number of cores for a symmetric multiprocessing (SMP) computer.
- `-nographic` disables graphical output such that QEMU is a simple CLI application accessible within the terminal window we are using to launch the VM.
- `-device virtio-net-pcci,netdev=net0` creates a virtio pass-through network device.
- `-netdev user,id=net0,hostfwd=tcp::10022-:22` directs QEMU to listen to port 10022 on the host and relay any connections to port 22 on the VM. This allowed me to `ssh` into my VM without obtaining its IP address.
- `-drive if=virtio,format=qcow2,file=~/focal-server-cloudimg-amd64.img` specifies the Ubuntu cloud image downloaded earlier as the virtual disk image for our VM.
- `-drive if=virtio,format=raw,file=seed.img` specifies that the seed image we created earlier will be a secondary image used as our local datasource 

After running the boot command, I accessed the VM directly via a separate terminal window. Since in my boot command I created a redirect of localhost’s port 10022 to forward traffic to my VM’s port 22, I `ssh` to my VM using port 10022 with `ssh -p 10022 ubuntu@0.0.0.0`. I verified my VM has internet access by simply executing `sudo apt-get install vim` in my VM to ensure the `apt-get` utility can connect to the internet and install any packages.

**Booting a VM in KVM w/ a Customized Linux Kernel:**

The last section focused on booting a VM with an Ubuntu 20.04 cloud image. I did not change the kernel version that ships with the cloud image. In this section, I will show how you can use your own kernel version and install kernel modules. The benefit of doing this is you can edit the source code of the kernel and see the implications of your changes in your VM. 

I used the [Linux Kerne 5.19.7](https://kernel.org/). After `wget`ting the tar file, I extracted the source code into `~/linux-5.19.7` on my host machine, created a separate build directory `~/kbuild` (which was a sibling of `~/linux-5.19.7`) and ran the following in `~/kbuild`:

```console
$ yes "" | make -C <kernel dir> O=$(pwd) config
```
This makes a configuration file (`.config`) in `~/kbuild` with the default options selected. Prior to building the kernel, I edited the generated `.config` file by clearing:
- `CONFIG_MODULE_REVOCATION_KEYS=""`
- `CONFIG_MODULE_TRUSTED_KEYS=""`
- `CONFIG_MODULE_TRUSTED_KEYS=""` (although below I explain why I ultimately should not have cleared it). 

This ensured that during the installation of kernel modules (the following step), I did not need any signatures to install the modules. I also ensured that `CONFIG_SATA_AHCI=y` such that the SATA disk driver was built into the kernel. 

I next built the kernel in `~/kbuild` by running `make -j4`. The `-j4` option sppeds up the building process by using multiple cores. Unfortunately, the build still took around 2 hours! After doing so, I installed the kernel modules locally on my host machine. Since I did not pare down my `.config` file or use `make menuconfig`, the default kernel modules that ship with the Linux Kernel 5.19.7 were installed. I created another directory `~/install_mod_dir` to install the kernel modules. In `~/kbuild`, I executed:
```console
$ make INSTALL_MOD_PATH=<install_mod_dir> modules_install
```
At first, I received the following SSL error message:

![SSL Error Message](/static/img/ssl-error.png)

This is because I had cleared `CONFIG_MODULE_SIG_KEY` in my `.config` file in `~/kbuild`. After setting `CONFIG_MODULE_SIG_KEY="certs/signing_key.pem"`, I remade the kernel, built the kernel, and successfully installed the kernel modules. 

Next, I mounted my disk image to a local directory `~/mount_point` that I created such that I could copy the kernel modules installed on the host machine onto the disk image. I used the `guestmount` command, as shown below: 
```console
$ export LIBGUESTFS_DEBUG=1 LIBGUESTFS_TRACE=1
$ guestmount -a ~/osmedia/focal-server-cloudimg-amd64.img /dev/sda1 ~/mount_point
```
The `-m` flag specifies the partition in the disk image that I want to mount. I determined that the root directory of my VM is on the `/dev/sda1` partition by logging into my VM and running `df` in the root directory. The export enabled me to debug if the `guestmount` command failed. This was especially useful, as I immediately ran into an error:

![Permission Denied](/static/img/permission-denied.jpeg)

Ultimately, running the `guestmount` command in `sudo` (`sudo su`) enabled me to successfully mount the disk image and overcome the Permission Denied error. I then copied the installed kernel modules (using the `cp` command) from `~/install_mod_dir/lib/modules` to `~/mount_point/lib/modules` and unmounted the disk image 
```console
fusermount -u ~/mount_point
```
One issue I encountered as I was attempting to copy the kernel modules to the disk image was running out of disk space. This is because the default virtual disk size was 2.2 GB. To increase the virtual disk size, I ran 
```console
$ qemu-img resize focal-server-cloudimg-amd64.img 10GB
```
Now I was able to successfully copy the modules to the disk image.

After, I successfully booted the VM with my customized Linux kernel by slightly tweaking the intial VM boot command. The full command is below: 

```console
$ sudo qemu-system-x86_64 \
    -machine accel=kvm,type=q35 \
    -cpu host \
    -m 4G \
    -smp 4 \
    -nographic \
    -device virtio-net-pci,netdev=net0 \
    -netdev user,id=net0,hostfwd=tcp::10022-:22 \
    -hda /home/psinha/osmedia/focal-server-cloudimg-amd64.img \
    -hdb seed.img \
    -kernel "/home/psinha/kbuild/arch/x86/boot/bzImage" \
    -append "root=/dev/sda1 console=ttys0,115200n8"
```
- `-kernel <path/to/kernel image>` prompted QEMU to to use the specific kernel we built in the previous steps
- `-append “root=/dev/sda1 console=ttys0,115200n8”` specified where the partition of the root directory on our disk image is and to add a console to the log in of the VM

I verified that my VM kernel version was correct by running `uname -r` in my VM. Moreover, I verified that my VM had internet access by successfully executing `sudo apt-get install bison`.

