## CMPE 283: Virtualization
### Assignment 2: Instrumentation via hypercall
### Assignment 3: Instrumentation via hypercall (cont’d)

### Team Members: Guru Karthik (only one person Assignment 1, 2 & 3)
Student ID: 015238509 

CPU leaf nodes implemented in this assignment are :
	For CPUID leaf node %eax=0x4FFFFFFF:
		Return the total number of exits (all types) in %eax
	For CPUID leaf node %eax=0x4FFFFFFE:
		Return the high 32 bits of the total time spent processing all exits in %ebx
		Return the low 32 bits of the total time spent processing all exits in %ecx
		%ebx and %ecx return values are measured in processor cycles, across all VCPUs
	For CPUID leaf node %eax=0x4FFFFFFD:
		 Return the number of exits for the exit number provided (on input) in %ecx
		This value should be returned in %eax 
	For CPUID leaf node %eax=0x4FFFFFFC:
		Return the time spent processing the exit number provided (on input) in %ecx
		Return the high 32 bits of the total time spent for that exit in %ebx
		Return the low 32 bits of the total time spent for that exit in %ecx
		
### Steps followed for the assignment

    Download and install VMWare WorkStation
	Download Ubuntu 20.04.3 iso
	Create Ubuntu VM in the VMWare using the iso
	Provide atleast 8GB of RAM, 200GB of Harddisk, 6 CPUs
	
The the Linux kernel source code is forked from git (https://github.com/torvalds/linux). Clone the forked repo from git.
Download the provided MakeFile

### Run the following commands
	sudo apt-get install manpages-dev
    make oldconfig
    make clean
    make prepare
    make -j 8 modules
    sudo make INSTALL_MOD_STRIP=1 modules_install && make install
    lsmod | grep kvm
    rmmod kvm_intel
    rmmod kvm
    modprobe kvm
    modprobe kvm_intel


Modify cpuid.c file and vmx.c file to report back additional information when special CPUID leaf nodes are requested.
	2)
		Return the high 32 bits of the total time spent processing all exits in %ebx
		Return the low 32 bits of the total time spent processing all exits in %ecx
	3) 
		Return the number of exits for the exit number provided (on input) in %ecx
	4)
		Return the time spent processing the exit number provided (on input) in %ecx

Comment on the frequency of exits – does the number of exits increase at a stable rate? Or are there 
more exits performed during certain VM operations? Approximately how many exits does a full VM 
boot entail?

	Total number of exits were not stable. MSR access resulted in more exits. A full VM boot resulted in 1252565 exits.

Of the exit types defined in the SDM, which are the most frequent? Least?
	
	Most frequent exits were: Interrupt Window (exit number 7) and MSR_WRITE (exit number 32)
	Least frequent exits were: MOV DR (exit number 29)

## Assignmment 4:

Team members:
### Guru Karthik (Student ID: 015238509)
### Sai Swarup Rath (Student ID: 014655446)

Answers:

1)
Guru Karthiks's contribution:
Ran assignment 3 code and booted a test VM. Upon booting of guest (nested) VM, recorded the total exit count information for each type of exit handled by the KVM.
Shutdown the guest (nested) VM. Gave answers for What was learnt from the count of exits and the count expectation.

Sai Swarup Rath's contribution:
Removed the ‘kvm-intel’ module from the running kernel. Reloaded the kvm-intel module with the parameter ept=0. Ran the 'insmod' command. Booted the same test VM again, and
captured the output for total exit count. Gave answers for what changed between the two runs ie. with and without ept.


2) Nested Paging i.e. Without ept = 0

![](https://github.com/gk97/linux/blob/master/cmpe283/SS1.PNG)
![](https://github.com/gk97/linux/blob/master/cmpe283/SS2.PNG)

Shadow Paging i.e. With ept = 0

![](https://github.com/gk97/linux/blob/master/cmpe283/SS3.PNG)

3) The Number of exits increased with ept=0 i.e. Shadow paging than with Nested paging. This increase in number of exits is an expected behavior because
in case of Nested paging VM exits occurs during EPT violation. In case of Shadow paging, a VM exit might take place whenever VM tries to execute CR0, CR3, CR4 or page faults.

4) With ept:
Guest VA is translated to PA and then Host PA using the layered page tables. Since guest VM should own the page table, the operations on CR3 are done natively without need for exits.

Without ept: 
The hypervisor needs to simulated the CR3 since guest virtual machine does not own page table in case of shadow paging.





