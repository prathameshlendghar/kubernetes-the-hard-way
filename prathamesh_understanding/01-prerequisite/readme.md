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

### Summary
1. **Kernel + KVM** (already in Linux) do low-level virtualization.
2. **qemu-kvm** runs the actual guest OS processes using that KVM support.​
3. **libvirt-daemon-system** manages those VMs centrally; **libvirt-clients** and **virtinst** let you control and create them via commands.​
4. **virt-manager** gives you a GUI front‑end to the same libvirt API.​
5. **bridge-utils** lets you create a bridge so all your VMs (Kubernetes nodes) can sit on the same layer‑2 network and talk to each other like physical machines.


> [!NOTE]\
> Finally our KVM and QMEU setup is done. You can also see the "vitual machine manager" within your applications.

### 3. Configuration and VM creation
----
**Difference between `.qcow2` and `.iso` files for booting the OS**

- A `.iso` image is just a bootable installer, similar to a USB drive, and it requires manual installation and configuration of the operating system.
- A `.qcow2` image, on the other hand, is a virtual disk format that can either be **empty** or contain a **pre-installed operating system**.
- In **cloud** and **virtualization** environments, *pre-built qcow2* images are commonly used so that virtual machines can be created and booted almost instantly. 
- The qcow2 format also supports features like snapshots and copy-on-write, allowing the system state to be preserved and restored.

---
1. Download the **Debian12 bookworm** from official debian web page
2. Create `jumpbox` and `node-template` node using the Debian12
```bash
virt-install \
  --name jumpbox \
  --memory 512 \
  --vcpus 1 \
  --disk path=/var/lib/libvirt/images/jumpbox.qcow2,size=10,bus=virtio,format=qcow2 \
  --cdrom /path/to/debian-12.iso \
  --os-variant debian12 \
  --network network=default \
  --graphics vnc
```

```bash
virt-install \
  --name node-template \
  --memory 2048 \
  --vcpus 1 \
  --disk path=/var/lib/libvirt/images/node-template.qcow2,size=20,bus=virtio,format=qcow2 \
  --cdrom /path/to/debian-12.iso \
  --os-variant debian12 \
  --network network=default \
  --graphics vnc
```

> [!NOTE]\
> - Just go with default or prefered configurations.
> - Uncheck any desktop environments (*GNOME, KDE, etc.*) if they are selected.
> - We just need `ssh-server` and `standard system utilities`

\
3. Now create `server` `node-0` `node-1` from `node-template` `.qcow` created above 

**server**
```bash
virt-clone \
  --original node-template \
  --name server \
  --file /var/lib/libvirt/images/server.qcow2
```

**node-0**
```bash
virt-clone \
  --connect qemu:///system \
  --original node-template \
  --name node-0 \
  --file /var/lib/libvirt/images/node-0.qcow2
```

**node-1**
```bash
virt-clone \
  --connect qemu:///system \
  --original node-template \
  --name node-1 \
  --file /var/lib/libvirt/images/node-1.qcow2
```

> [!NOTE]\
We have successfully setup the VMs