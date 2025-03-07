Virtualization is a technology that allows multiple virtual machines (VMs) to run on the same hardware resource. This is a key component for Cloud computing as providers can host several customers on the same computer, improving resource utilization while isolating each VM from one another. Customers also benefit from this technology as they can easily change server provisioning to meet their current needs. However, the rise of Cloud-computing led to an increased attention for virtualization and highlighted many vulnerabilities.

## Co-Residential Attacks 
The main danger of virtualization in cloud computing is sharing a physical resource with a malicious user. When planning an attack on a target VM, an attacker will launch several malicious VMs until one is located on the same computer as the target VM. This objective can be achieved through brute-force but the attacker can also exploit sequential and parallel locality in VM placement. For example, in Amazon EC2, two VMs started almost at the same time are more likly to be co-located. Once the VMs are created, the attacker can verify co-residency with different methods. For example, with VMs using the Xen hypervisor, a user can perform TCP traceroute operation to get the Dom0 IP address. Dom0 is a privileged VM for administration and is shared for all VMs on the same computer. Sharing the same Dom0 IP address as another VM implies Co-residency. Co-residency can also be confirmed using Cache-based Side Channel Attacks (CSCA) if the target VM provides public services. First, the malicious VM fills the CPU cache to the maximum then places a load on the target VM. The attacker can then measure the cache access time : a high cache access time is attributed to a high activity by other co-resident VMs, and since the target VM received a big workload, it is probably the one who evicted data from the cache.

## DoS Attacks 
Usually, DoS attacks implies saturating the network interfaces of the target with a massive load of data, sometimes from different computers (DDoS). But if an attacker acheives co-residency with a VM hosting part of or all of the target, they will share hardware resources such as CPU time, CPU caches or memory. In a cloud environment, a DoS attack can be performed by using all resources from the host, denying access to the other users and thus preventing the execution of the target partially or completely.

## Side Channel Attacks 
Sharing hardware resources between VMs exposes them to Side Channel Attacks/

### Time driven CSCA
Attackers write and read inside CPU caches and measure execution time to deduce information about the activity of the target VM. 

-Evict & Time : The attacker measures the execution time of a service, then write inside the CPU cache (evict). Finally, the attacker measures the execution time again. If it is longer than the first time, it means the service accessed part of the cache who was evicted.

-Prime & Probe : The attacker first fill one or more CPU caches with his data (prime). He then waits for the execution of the targeted service. After that, he tries to access the primed caches : an increase in latency for a memory access means that the target accessed this cache.

### Trace driven CSCA
Instead of measuring the execution time, the attacker analyze the traces left in the cache by the target service after execution. Prime & Probe is the main method for this attack.

## VM Escape 
A VM escape attack aims to compromise the isolation feature between the VMs and the host, effectively gaining access to the host. Usually, the attacker will target the hypervisor, the software handling virtualization.The objective is to modify certain data to increase the attacker capabilities. The privilege level is the most important target as it can give control over other VMs or the hypervisor itselmf. But resource utilization data can also be modified to gain more physical resources than a regular user, a perfect asset for DoS attacks. In addition, an attacker can modify the security policy to let sensitive VMs run on a machine with malicious VMs as a preliminary step for a Side Channel Attack. Once the hypervisor is compromised, the attacker can perform a VM rollback attack. The hypervisor will launch a VM from an older snapshot, unbeknownst to the owner. This attack can disable some security patches on the VM or reset the password alert when attempting a brute force attack. However, a VM escape attack does not necessarily need to go through the hypervisor. In a blog post, Ezequiel Pereira and Wouter ter Maat were able to escape to the host of their Google Cloud SQL instance by exploiting a connection from the Google Guest Agent to the metadata instance and spoofing the SSH keys response, giving them total access to the host.

## VM Cloning and Memory Dump
These two attacks both aim to copy information from the target VM to access sensitive data. In a VM Cloning attack, the malicious user clone the target VM with a copy of the VM program file from the host operating system. In a VM Memory Dump attack, the attacker dumps the memory assigned to the target in an attempt to gain information.
