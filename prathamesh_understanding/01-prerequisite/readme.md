# My Approach

I will be using the Linux `Linux prathamesh-VivoBook-ASUS-Laptop-X505ZA-X505ZA 6.14.0-33-generic #33~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC Fri Sep 19 17:02:30 UTC 2 x86_64 x86_64 x86_64 GNU/Linux`
 as my base OS and OS-kernel

## Attempting to setup required VM's through Linux KVM

### 1. Checking PC capability for virtualization and installing KVM
`/proc/cpuinfo` file contains the details regarding the underlying CPU
```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
        OR
grep -c vmx /proc/cpuinfo
        OR
grep -c svm /proc/cpuinfo

## Returns number of cores that report virtualization support. 
## If O/P is number greater than 0 then machine is supported for Virtualisation
```
> [!NOTE]\
> On Intel CPUs, virtualization support (VT-x) may be disabled by default.  
> If the `vmx` flag does not appear, enable **Intel Virtualization Technology** in your BIOS/UEFI settings.



### 2. Understanding the magic or Linux kernal virtualization capabilities and setting up the tools required for virtualization

- Architecturally, the **hypervisor(KVM) lives in the kernel**, reusing **Linux’s scheduler, memory management, device drivers, and I/O stack**, instead of reinventing all that in a separate hypervisor binary.
- **KVM kernel modules**: **kvm**, plus **kvm-intel** or **kvm-amd**, providing the core virtualization infrastructure and CPU‑specific handling.

#### What we need to install seperately (and Why)
```bash 
$ sudo apt install -y 
qemu-kvm 
virt-manager 
libvirt-daemon-system 
virtinst 
libvirt-clients 
bridge-utils
```
- **QEMU** is a user-space program that creates and manages virtual machines by **emulating hardware devices**, while delegating CPU virtualization to KVM for near-native performance. Each virtual machine appears as a normal Linux process.
- **virt‑manager / other tools**: Human‑friendly interfaces (*GUI* or higher‑level tools) that talk to libvirt so you can create, configure, and operate VMs without low‑level details.
- **libvirt (libvirtd, virsh, XML definitions)**: Management API and service that organizes VMs, networks, and storage, and provides a stable interface instead of dealing directly with QEMU commands.
- **virtinst** is a set of command‑line utilities (most importantly virt-install) for creating and installing VMs.​\
Instead of clicking through a GUI, you can run one command that specifies CPU/RAM/disk/ISO/network and it will define and start the guest via libvirt.
- **libvirt-clients** This installs CLI tools like virsh, virt-xml, etc., which talk to the libvirt daemon.​\
You use them to list/start/stop VMs, define networks, inspect status, etc. useful for scripting and when working over SSH
- **Bridge-utils**: bridge-utils provides tools like brctl to create and manage Linux bridge interfaces (e.g., br0).​\
A bridge works like a virtual switch; connecting your physical NIC and VM tap interfaces to it lets VMs appear as normal hosts on your LAN (bridged mode) instead of just using default NAT.