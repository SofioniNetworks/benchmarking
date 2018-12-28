# Performance Analysis of VPP and vRouter

<img src="https://media.licdn.com/dms/image/C4D0BAQGcetGJaM_8pg/company-logo_200_200/0?e=2159024400&v=beta&t=zOorZC2j6Y6c2GaKLllYxPgQ0Yn_s-9TtJC2m7eK_r8" width="50" height="50" /> **Sofioni Networks**

## Overview
This document is targeted to describe the performance analysis between Tungsten Fabric's data plane, **vRouter** and **VPP**. This report provides results and a setup guide to packet processing performance testing of the considered data planes. This report includes baseline performance data and provides system configuration and test cases.

## Platform Specifications
### Hardware Specifications
| Item | Description |
| ------ | ------ |
|Server Platform | Supermicro X10SRD-F
| Processor | Intel(R) Xeon(R) CPU E5-2630 v4 @ 2.20GHz; 20 cores
| Memory  | 128GiB Total; 32GiB per channel, 4 channels
| Local Storage | 4TB HDD
| NICs | Intel I350 Gigabit Network Connection; 1Gbit/s capacity

### Software Versions
| Item | Description |
| :------ | :------ |
| Host OS | CentOS Linux release 7.6.1810; Kernel Version: 3.10.0-957.1.3.el7.x86_64
| Guest OS | Fedora release 21; Kernel Version: 3.19.1-201.fc21.x86\_64
| QEMU-KVM | qemu-kvm 1.5.3; libvirtd (libvirt) 4.5.0; 
| TRex | Traffic generator; latest version (v2.49); NIC driver: igb\_uio; TRex Mode: stateless

## Setup Details
Three nodes are used in the performance analysis of **vRouter** and **VPP**. One node is controller and the rest of them are compute nodes. Controller is deployed on CentOS (7.6.1810 release) virtual machine while the compute service is installed on physical machines. Three nodes [setup][CVPPD] of Tungsten Fabric environment is fully automated. One physical node uses **VPP** as data plane and the other one uses **vRouter** . Following picture depicts the deployment setup.

                             ------------------
                            |                  |
                            | openstack control|
                            |         +        |
                            | contrail control |
                             ------------------
        ------------------           ||             ------------------
       |     compute      |          ||            |     compute      |
       |        +         |<---------||----------->|        +         |
       |     vRouter      |                        |       VPP        |
       |                  |                        |                  |
        ------------------                          ------------------

## Test Cases
VMs flavor used in the performance testing has:
  - 6 vCPUs
  - 40 GiB RAM
  - 200 HDD
  - Metadata with hugepages enabled (Hw:large)
  - Fedora 21
  
### Test# 1
Four VMs are launched on both compute hosts, two on each. Each pair of VMs on both host are launched in the same network. One VM runs TRex server and the other runs TRex client. TRex server throws traffic towards client which sends it back as a reply against each request made by TRex server. The following figure describes the topology used in this scenario.


                             ------------------
                            |                  |
                            | openstack control|
                            |         +        |
                            | contrail control |
                             ------------------
        ------------------           ||             ------------------
       | vRouter compute  |          ||            |    VPP compute   |
       |                  |          ||            |                  |
       |                  |          ||            |                  |
       |    VM_1_TS (N2)  |<---------||----------->|    VM_1_TS (N1)  |
       |       +          |                        |         +        |
       |    VM_2_TC (N2)  |                        |    VM_2_TC (N1)  |
        ------------------                          ------------------


##### Results
Following table shows the packets transferred by VPP and vRouter at NDR.

| Data Plane | Max PPS at NDR | Packet Size in Bytes | Max TX |
| :------ | :------ | :------ | :------ |
| vRouter | 0.28mpps | 64  | 143.88Mbps
| VPP | 0.36mpps | 64   | 184.86 Mbps
| vRouter | 0.27mpps | 512 | 1.11Gbps
| VPP | 0.34mpps | 512 | 1.40Gbps
| vRouter |  0.26mpps | 1500 | 3.13Gbps
| VPP | 0.32mpps | 1500 | 3.85Gbps



### Test# 2
Four VMs are launched on both compute hosts, two on each. Each VM in a pair on both host is launched in a different network. One VM runs TRex server and the other runs TRex client. TRex server throws traffic towards client which sends it back as a reply against each request made by TRex server. The following figure describes the topology used in this test case.


                             ------------------
                            |                  |
                            | openstack control|
                            |         +        |
                            | contrail control |
                             ------------------
        ------------------           ||             ------------------
       | vRouter compute  |          ||            |    VPP compute   |
       |                  |          ||            |                  |
       |                  |          ||            |                  |
       |    VM_1_TS (N1)  |<---------||----------->|    VM_1_TS (N1)  |
       |       +          |                        |         +        |
       |    VM_2_TC (N2)  |                        |    VM_2_TC (N2)  |
        ------------------                          ------------------

##### Results
Following table shows the packets transferred by VPP and vRouter at NDR.

| Data Plane | Max PPS at NDR | Packet Size in Bytes | Max TX |
| :------ | :------ | :------ | :------ |
| vRouter | 0.28mpps | 64 | 143.77Mbps
| VPP | 0.36mpps | 64 |  184.94Mbps
| vRouter | 0.27mpps | 512 | 1.11Gbps
| VPP | 0.34mpps | 512| 1.40Gbps
| vRouter |  0.26mpps | 1500 | 3.13Gbps 
| VPP | 0.33mpps | 1500 | 3.97Gbps

**Note: Further test cases will be added soon.**

### Acronyms

> **VPP:** Vector Packet Processing <br/>
> **VM:** Virtual Machine <br/>
> **TX:** Transmission <br/>
> **mpps:** Million packets per second <br/>
> **Gbps:** Giga bits per second <br/>
> **Mbps:** Mega bits per second <br/>
> **PPS:** Packets per second <br/>
> **TS:** TRex server <br/>
> **TC:** TRex client <br/>
> **NDR:** No drop rate <br/>
> **N1:** Network 1 <br/>
> **N2:** Network 2 <br/>

  [CVPPD]: <https://github.com/OMajeed/contrail-vpp-deploy>
