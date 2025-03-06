In order to implement Resource pooling and multi-tenancy, cloud companies started using virtualization, allowing them to run multiple OS on the same computer. This technology is also the target of several attacks

## DoS Attacks 
A hacker can use a cloud provider to perform a DDoS attack on the cloud provider itself or one of its clients.

## Co-Residential Attacks 
Using side channels, a hacker can steal private information from virtual machines who share the same server as a malicious VM. But first, the hacker needs to co-locate one of his VMs with the target VM.

## Cache-Based Side Channel Attacks 
Possible for two VMs on the same computer using bare-metal hypervisor sharing highest level-cache

## Hypervisor compromission 
The hypervisor is the software responsible for virtualization. A compromised hypervisor may lead to the compromission of all hosted virtual machines.

## VM Escape 
A hacker tries to exploit a fault in isolation features to gain access to the host machine.

## VM Cloning 
A hacker copies the program files of a VM from the host OS and then makes a clone to access sensitive data.

## VM Memory Dump 
A hacker dumps the memory assigned to the target VM to search for information.
