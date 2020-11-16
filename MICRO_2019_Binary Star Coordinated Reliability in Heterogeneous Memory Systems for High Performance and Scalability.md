# Zhongfa Wang 2020/11/9

# Paper Information

* Title: Binary Star: Coordinated Reliability in Heterogeneous Memory Systems for High Performance and Scalability

* Authors: Liu, Xiao; Roberts, David; Ausavarungnirun, Rachata; Mutlu, Onur; Zhao, Jishen

* Venue: MICRO_2019
* Keywords: reliability, nonvolatile memory, hybrid memory, memory systems

# Paper content

## Summary

This work studies the emerging memory technologies: 3D DRAM last-level cache (LLS) and nonvolatile main memory. The 3D DRAM LLS suffers from degraded reliability comparing to DRAM. On another hand, the performance of persistent memory systems enabled by NVRAM is degraded by the high performance and storage overheads. In a word, the 3D DRAM LLS and VRAM still remains reliability and performance issues. Thus, this work focuses on coupled and coordinated reliability across the different memory hierarchy levels (3D DRAM last-level cache and nonvolatile main memory). The researches propose Binary Star, which coordinates not only  the reliability mechanisms between the last-level cache and main memory but also the wear leveling scheme in main memory. The Binary Star significantly reduces the performance and storage overhead of consistent cache writeback. 

## Strengths

* This work focuses on emerging technologies, in other words, 3D DRAM last-level cache and nonvolatile main memory and considered reaping the full reliability advantages of them beyond simply replacing the existing memory technologies.

## Weaknesses

* The Introduction is somehow vague. The writer didn't explain the link between motivations (the reasons why did they do such work) and contributions (what they have done) well.

## Thoughts

* It's necessary to coordinate the hierarchies, especially when implementing new technologies.

## Takeaways and questions

* This work mainly talks about reliability. How about performance? How good in performance will the heterogeneous memory system be, comparing to the traditional/ existing memory systems?