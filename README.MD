CMPE283-ASSIGNMENT 1 
-----------------------------------------------------------------------------------------------------------------------------------------------------------------
Completed by 
Aakanksha Gupta 
San Jose State University
-----------------------------------------------------------------------------------------------------------------------------------------------------------------
**To Do:** 
Create a linux kernel module that will query various MSR's to determine virtualization feautures available in the CPU. 


**Step 1:** Created a linux virtual Instance hosted on GCP 

**Machine Configurations:** 
Machine Type: n2-standard-8
Disk Image: Ubuntu 1604 (150GB)

**Step 2:** Check if nested virtualization if enabled for your machine or not,
Note: (By default for gcp it is disabled, follow https://cloud.google.com/compute/docs/instances/nested-virtualization/enabling )

**Step 3:** Verify nested virtualization or enabled or not using below command 
 
 $ gcloud compute ssh <<VM_NAME>>
 
 $ grep -cw vmx /proc/cpuinfo

(If nested virtualization is enabled for your machine, then you should get value > 0)
![image](https://user-images.githubusercontent.com/89236239/141696134-215a4131-b2b1-4252-90d9-8a1e0c7549b3.png)


**Step 4:** 
Clone repo https://github.com/aakankshagupta1994/linux.git

**Step 5:**
Added Primary/processor/exit/entry based VM execution controls list to .c file. (here it is already done, make changes are required)

**Step 6:**
On trying **make** this new added file, you might see some errors. Errors may be because of version mismatch. 

Follow below steps to mount new kernel version:

$ cp /boot/config-$(uname -r) .config   

$ make oldconfig

**Step 7:** 
Building and Installing Kernel:
 
 
 $ sudo make modules -j 8
 
 $ sudo make -j 8 
 
 $ sudo make INSTALL_MOD_STRIP=1 modules_install -j 8
 
 $ sudo make install -j 8

On successful build, uname -a should show the updated version
![image](https://user-images.githubusercontent.com/89236239/141696787-0fb1102c-c69b-4d0c-b89b-08223125e286.png)


**Step 8: **
On successful make, .ko file will get generated, mount .ko file using below command
Installing module

 $ sudo insmod cmpe283-1.ko


**Step 9: (optional)**
If any other change is required in .c file, 
1. unmount the already mounted module, using 

 $ sudo rmmod cmpe283-1.ko
2. make changes to your file. 
3. mount new updated file again, using 

 $ sudo insmod cmpe283-1.ko

(Check whether module is mounted or not using lsmod | grep cmpe283)


**Step 10:** 
Check results, using 

 $ dmesg
 
 -------------------------------------------------------------------------------------------------------------------------------------------------------------
 ASSIGNMENT-2
 -------
TASK:
------
Your assignment is to modify the CPUID emulation code in KVM to report back additional information 
when special CPUID leaf nodes are requested.
1. For CPUID leaf node %eax=0x4FFFFFFF:
◦Return the total number of exits (all types) in %eax

2. For CPUID leaf node %eax=0x4FFFFFFE:
◦Return the high 32 bits of the total time spent processing all exits in %ebx
◦Return the low 32 bits of the total time spent processing all exits in %ecx
▪%ebx and %ecx return values are measured in processor cycles, across all VCPUs


QUESTIONS: 
------
1. For each member in your team, provide 1 paragraph detailing what parts of the lab that member 
implemented / researched. 

INDIVIDUAL ASSIGNMENT


PLATFORM USED: 
------
GOOGLE CLOUD VM 

PREREQUISITES:
----
1. ASSIGNMENT-1 SETUP
2. BASIC NESTED VM RUNNING ON GCP KNOWLEDGE
(If not follow https://cloud.google.com/compute/docs/instances/nested-virtualization/overview)

DETAIL:
------

GO TO /linux/arch/x86/kvm/cpuid.c file ---> kvm_emulate_cpuid function
this is the function, which is used to emulate cpuid instruction. 
Check directory linux/arch/x86/kvm/cpuid.c for details.


GO TO /linux/arch/x86/kvm/vmx/vmx.c ----> vmx_handle_exit 
This function emulates all the exits. 
Check directory linux/arch/x86/kvm/vmx/vmx.c for details.

TEST STEPS: 
-----

Follow below steps for successful loading of the module.
```
make -j 8 modules
make -j 8
sudo make INSTALL_MOD_STRIP=1 modules_install
sudo make install 
```

Check whether kvm is already mounted or not using below command
```
lsmod | grep kvm
```
 If already mounted, remove kvm and kvm_intel and mount it again using below commands. 
```
rmmod kvm_intel
rmmod kvm
modprobe kvm_intel
modprobe kvm
```



Run nested VM on gcp to call CPUID command 

Here I have installed CentOS on VM using below commands

```
sudo apt-get install qemu-kvm libvirt-bin bridge-utils virt-manager

wget http://kozyatagi.mirror.guzel.net.tr/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-DVD-2009.iso

$ virt-install  --network bridge:virbr0 --name l2vmcmpe283 \
 --os-variant=centos7.0 --ram=1024 --vcpus=1  \
 --disk path=/var/lib/libvirt/images/l2vmcmpe283-os.qcow2,format=qcow2,bus=virtio,size=5 \
  --graphics none  --location=/var/lib/libvirt/images/CentOS-7-x86_64-DVD-2009.iso \
  --extra-args="console=tty0 console=ttyS0,115200"  --check all=off
```

Start server using 

```
virsh start l2vmcmpe283
virsh console l2vmcmpe283
```

Check whether server is running or not 
```
virsh -c qemu:///system list
```


RESULTS: 

![image](https://user-images.githubusercontent.com/89236239/142974636-faa9d681-c525-40d2-8470-90d80f7b71a1.png)


ASSIGNMENT-3
------
Your assignment is to modify the CPUID emulation code in KVM to report back additional information 
when special CPUID leaf nodes are requested.
For CPUID leaf node %eax=0x4FFFFFFD:
◦Return the number of exits for the exit number provided (on input) in %ecx
▪This value should be returned in %eax 
•For CPUID leaf node %eax=0x4FFFFFFC:
◦Return the time spent processing the exit number provided (on input) in %ecx
▪Return the high 32 bits of the total time spent for that exit in %ebx
▪Return the low 32 bits of the total time spent for that exit in %ecx

QUESTIONS: 
------
1. For each member in your team, provide 1 paragraph detailing what parts of the lab that member 
implemented / researched. 

INDIVIDUAL ASSIGNMENT

3. Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are there 
more exits performed during certain VM operations? Approximately how many exits does a full VM 
boot entail?

--> It was observed that the frequency of exit was not stable, sometimes they increased by 5, sometimes no change etc. 
--> It was observed that mostly exits were caused due to I/O instructions. 
--> On full VM reboot it was observed a total of 12300 exits increased

4. Of the exit types defined in the SDM, which are the most frequent? Least?
--> Most common exits were EPT violations.
--> Least common were Triple fault.



PLATFORM USED: 
------
GOOGLE CLOUD VM 

PREREQUISITES:
----
1. ASSIGNMENT-1 SETUP
2. BASIC NESTED VM RUNNING ON GCP KNOWLEDGE
(If not follow https://cloud.google.com/compute/docs/instances/nested-virtualization/overview)

TEST STEPS: 
----
FOLLOW ASSIGNMENT-2 STEPS FOR TESTING CHANGES 

RESULTS: 
----

TEST CASE 1 : %eax=0x4FFFFFFC
If the exit type is not supported by Intel-SDM, returned eax=abx=ecx=0x00000000 and edx = 0xffffffff
If exit type is not supported by KVM, returned eax=abx=ecx=edx0x00000000
![image](https://user-images.githubusercontent.com/89236239/143984844-502d627e-49a2-436d-9b7a-e7b0cbd46b0a.png)

TEST CASE 2: %eax=0x4FFFFFFC
If the exit type is not supported by Intel-SDM, returned eax=abx=ecx=0x00000000 and edx = 0xffffffff
If exit type is not supported by KVM, returned eax=abx=ecx=edx0x00000000
![image](https://user-images.githubusercontent.com/89236239/143985079-2d49e663-d104-4d34-a96c-cdc1de79dc3a.png)











  



 

