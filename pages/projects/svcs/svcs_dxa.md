SVCS3 uses the hardware platform "Dextroamphetamine" (shorthand "DXA"). This system was not put into production until March 2024, one of the reasons for this was that I could not have the system boot to a drive connected to the RAID controller. The fix was relatively simple; all I needed to do was change a setting within HP ORCA to have the system boot from a drive on the RAID controller.

On February 11, 2024, the E5-2643 v2 CPUs where switched for E5-2667 v2 CPUs due to the higher core count and slight increase in single-threaded performance. 

On April 5, 2024, a Minecraft server was set up bare metal on this system for use by a college club. 16GB of memory is allocated to the JVM and the server has access to the 16 cores/32 threads of the two E5-2667 v2.

As covered in the [blog post for Week 12 of 2024](../../blog/15/), there has been attempts to run LLMs on this system.

Specifications as of May 2, 2024:

- System: HP ProLiant BL460c G8
- CPUs: 2x Intel Xeon E5-2667 v2
- Memory: 64GB (8x8GB) Nanya Technology NT4GC72B4PB2NL-DI Registered ECC 1Rx4 PC3-12800R
- Storage: 1x Samsung PM851 256GB SATA SSD
