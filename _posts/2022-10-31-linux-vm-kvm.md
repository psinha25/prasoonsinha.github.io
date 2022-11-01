---
title: Booting Ubuntu VM on top of KVM-Qemu Hypervisor
# sub-title: Model Serving
layout: post
tags: [linux, virtual machine, operating systems]
mathjax: true
image: /static/img/prasoon-sinha.png
cover-img: "/static/img/linux-vm-cover.png"
share-img: /static/img/prasoon-sinha.png
social-share: false
readtime: true
---

#### Post Goal:

This blog post summarizes how to compile and run a customized Linux kernel on a KVM-qemu virtual machine. It specifies:
1. How to run a VM in KVM using a cloud image
2. How to use a customized Linux kernel in place of the default shipped cloud image kernel
3. How to use GDB with a customized Linxux kernel in KVM.

#### Hardware & Software Specs:

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

#### Booting a VM in KVM w/o Customized Linux Kernel

