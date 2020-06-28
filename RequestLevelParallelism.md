## Request-level parallelism

### 1 Introduction
  - Goals and requirements shared between WSC architects and server architects:
    - Cost-performance
    - Energy-efficiency
    - Dependability via redundancy
    - Network I/O
    - Both interactive and batch processing workloads
  - Difference between WSCs and HPC:
    - HPCs generally have much faster processors and much faster networks between the nodes than are found in WSCs.
    - HPC clusters tend to have long-running jobs that keep the server fully utilized while the utilization of servers in WSCs range between 10% and 50% per day.
  - Difference between WSCs and conventional data centers:
    - Conventional datacenters tend have a great deal of hardware and software heterogeneity to serve their varied customers.
    - Often the largest cost in a conventional datacenter is the people to run it.
  - characteristics not shared with server architecture:
    - Ample parallelism
    - Operational costs count
    - Scale and the opportunities/problems associated with scale
      - Hardware can be cheaper.
      - Errors can be more frequent.

### 2 Computer architecture of warehouse-scale computers
  - The 19-inch rack is still the standard framework to hold servers. Servers are measured in the number of rack units that they occupy in a rack. One U is 1.75 inches high.
  - A 7-foot rack offers 48 U, so the most popular switch for a rack is a 48-port Ethernet switch.
  - These switches typically offer two to eight uplinks, which leave the rack to go to the next higher switch in the network hierarchy.
  - Oversubscription
  - WSC Memory hierarchy
    - Most applications fit on a single array within a WSC. Those that need more than one array use sharding or partitioning, meaning that the dataset is split into independent pieces and then distributed to different arrays.
    - For block transfers outside a single server, it does not matter whether the data are in memory or on disk since the rack switch and array switch are the bottlenecks.
  |Data(2009) |Local|Rack|Array
  |---|---|---|---|
  |DRAM latency (ms)| 0.1 | 100 | 300
  |Disk latency (ms)| 10,000 | 11,000 | 12,200
  |DRAM bandwidth (MB/sec) | 20,000 | 100 | 10
  |Disk bandwidth (MB/sec) | 200 | 100 | 10
  |DRAM capacity (GB)| 16 | 1040 | 31,200
  |Disk capacity (GB)|2000 | 160,000 | 4800,000

### 3 Physical infrastructure and costs of warehouse-scale computers
  - Typical procedure for power transmission
    - From high-voltage utility 115,000 volts to substation 13,200 volts
    - From substation to uninterruptible power supply (UPS)
    - From UPS to a power distribution unit (PDU) 480 volts
    - From PDU to 208 volts that servers can use
  - Overall efficiency is around 89%

### 4 Measuring efficiency of a WSC
  - Power utilization effectiveness (or PUE)
    - PUE = (Total facility power)/(IT equipment power)

### 5 Cloud computing: the return of utility computing
  - Comparison between a WSC with a datecenter with only 1000 servers [data collected in 2010]:
    - 5.7 times reduction in storage costs
    - 7.1 times reduction in administrative costs
    - 7.3 times reduction in networking costs

### 6  Amazon web services
  - AWS's important decisions when Amazon started offering utility computing via the Amazon Simple Storage Service (Amazon S3) and Amazon Elastic Computer Cloud (EC2) in 2006:
    - Virtual machines.
    - Very low costs.
    - No contract required.
    - Reliance on open source software.

### 7 Crosscutting issues
  - WSC network as a bottleneck
  - Using energy efficiency inside the server
    - While PUE measures the efficiency of a WSC, it has nothing to say about what goes on inside the IT equipment itself. In 2007, many power supplies were 60% to 80% efficient, which means there were greater losses inside the server than there were going through the many steps and voltage changes from the high-voltage lines at the utility tower to supply the low-voltage lines at the server.
    - WSC should be energy proportionality (servers should consume energy in proportion to the amount of work performed).

### 8 Putting it all together: A Google warehouse-scale computer
  - containers
  - cooling and power in the Google WSC: The restricted space of the container prevents the mixing of hot and cold air, which improves cooling efficiency. The "cold" air is kept 81F, which is balmy compared to the temperatures in many conventional datacenters. By controlling airflow to prevent hot spots, the container can run at a much higher temperature.

### 9 Fallacies and pitfalls
  - Pitfall: trying to save power with inactive low power modes versus active low power modes:
    - You cannot access DRAMs or disks in these inactive low power modes, so you must return to fully active mode to read or write, no matter how low the rate. The pitfall is that the time and energy required to return to fully active mode make inactive low power modes less attractive.
  - Pitfall: using too wimpy a processor when trying to improve WSC cost-performance:
    - If the serial work increases latency, then the cost of using a wimpy processor must include the software development costs to optimize the code the return it to the lower latency.
  - Fallacy: given improvements in DRAM dependability and the fault tolerance of WSC systems software, you don't need to spend extra for ECC memory in a WSC.
  - Fallacy: turning off hardware during periods of low activity improves cost-performance of a WSC.
  - Fallacy: replacing all disks with Flash memory will improve cost-performance of a WSC.
