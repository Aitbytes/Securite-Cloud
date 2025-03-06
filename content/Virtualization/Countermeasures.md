To mitigate the risk of co-resident attacks, researchers proposed various approaches

## Preventing Side Channel Attacks
Most SCA work with high resolution internal clocks. Eliminating references to internal clocks will protect software from several SCA. This solution however is hardware-based and costly

## Periodic Migration 
Regular migration following a VCG mechanism will hinder the capacity of an attacker to perform co-residential attacks. Drawbacks of this method include performance degradation and power consumption.

## Preventing Co-Residency Verification 
To verify co-residency, attackers check if their VMs and the target VM share the same Dom0 IP address. Dom0 is a high-privilege VM used to administrate other VMs and is shared by all VMs on a physical computer. If the Dom0 IP address is confined, a malicious user will have to resort to difficult techniques if he wants to confirm co-residency.

## Detecting Co-location attacks 
Various features can be observed for anomalies in order to detect Cache-Based Side Channel Attacks, including CPU usage, RAM usage and cache miss.

## Allocation Policy 
An allocation policy is the policy employed by a cloud provider to allocate VMs to a physical resource. Some policies can increase the number of attempts an attacker needs to achieve a co-residency. For example, CLR (Co-location Resistance Algorithm) opens a fixed number of servers and randomly allocates a VM, or PSSF (Previously-selected-server-first) will try to allocate a user’s VM to a server already used by the user.
