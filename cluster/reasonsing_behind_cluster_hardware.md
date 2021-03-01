# Why did you choose this hardware?

This dokument should sheed some light on the thoughts behind the choice for each hardware component. This process wasn't worry free since there was no expertice in designing the hardware in clusters. More importantly there was no real knowladge of the workload for the cluster. As it will be determined by the various projects coming up in the next years. Usually cluster hardware is determined after you have a good idea of the work you want to perfom on the cluster. Depending on this load, you focus more on the network side, or the io-side or the cpus. But because of the boundery conditions the time of ordering wasn't nagotiable a broad approach was choosen. The process of defining the different hardware component was a very iterative process and invold advices from Mr. Kösling and Mr. Nikisch from SysGen.

## The Cpu
AMD EPYC Rome 7352 24C/48T, 2,30GHz 128MB L3
Since AMD offers better price per core performance I went for AMD. The specific CPU model was choosen based on the best price per core value. The clock speed of the cores is not the highest, but since it's beeing used in a highly parallalized workload this is not too important.

## The Chasis
Supermicro SuperServer 2024US-TRT
Since the last cluster is made by supermicro and the good experiences with their hardware we choose it. Because of the recommendations of Mr. Kösling we choose a normal 2U server. Instead of the combined setup in the old cluster. Changing and expanding hardware will be easier in such a 2U server also those units can be split up and repurposed easier. The choosen model is the newest chasis from supermicro for AMD architecture having a 2U hight and allowing for 3.5 hdd. 

## The Ram
16 * 16GB DDR4 3200MHy ECC
The Ram speed is determined by the cpu. Since the cpu has 8 channels for ram and there are two cpus in each server this amounts to a total of 16. Spark hardware specification state that a cluster node can have up to 200 GB of ram, since the spun up JavaVirtualMachine might have trouble with more. Since there is the option to run mutiple nodes on a single server there is a way to circumvent these limitation. Since we want maximal Ram at the highest possible throughput the choise fell on 16RM ram modules. With the sweet spot between 8Gb and 32Gb models.

## OS storage
2 * 960GB Samsung SSD PM983 NVMe
Since the old cluster used ssd for the os in a fakeraid 0 setup. I choose a similar setup for the new cluster. This storage is used only for the os and very important data. I'm expecting not too many reading and writing cycles in normal operation. The choosen Samsung SSD connect via pci 3.0 and are very fast by at the same time being relatives cost effective. 

## DATA storage
10 * 4TB SAS 3.5 HDD
I was switching between SATA and SAS disk quite often. In the end the choice fell on the SAS models since the price difference is quite low. The more disk the better because I expect hadoop to distribute the data across all hdds in a server so we get maximum io performance. With 10 disk we reach almost the maximum capacity of the server. In total the new cluster will then have a total capacity of 14 * 40GB = 560GB. Since data saved on hdfs has some redundancy the actual capacity is not know. A raid 6 setup would in this situation amount to 32GB per server and a total of 448GB for the entire new cluster. This should be enough space for the next years to come. Smaller disk sizes are not really available and not so much cheaper.

## Hdd Controler
This has not been defined by me. We let the participants in the bid choose a compatible controller for the setup.

## Network
A lot of thoughts went into determining the right network setup. Ultimatly we deemed option above 10G-Ethernet as too expensive. The spark cluster specificaitons recommends 10G. But as always this really depends on the future workload. For task which demand a lot of exchange in data between nodes the network could become a bottle neck. Another faktor might be the pretty high latency of 10G RJ45 Ethernet. But all options above it need an addtional network card, and way more expencive network switches and cabled. So for the time being we keep the onboard networking. Maybe in future project there will be money to upgrade the network. Since there are dual networkcard in the servers I plan to aggreagate the ports so we end up having a 20GB connection between a server an the network switch.

## Network switch
A particular model hase not been put into the specification of the cluster. All servers in the network will be connected via 2 ports. The old cluster will also get a new switch and be updated to dual 10G connections. The connection between the switches will be done using at least 8 aggregated port. 
