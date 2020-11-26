* # Zhongfa Wang 2020/11/25

  # Paper Information

  * Title: Binary Star: Coordinated Reliability in Heterogeneous Memory Systems for High Performance and Scalability

  * Authors: Xiao Liu, David Roberts, Rachata Ausavarungniru, Onur Mutlu, Jishen Zhao

  * Venue: MICRO 2019
  * Keywords: memory hierarchy design, 3D die-stacked DRAM, nonvolatile memories, comprehensive coordinated memory hierarchy reliability scheme, 

  # Paper content

  ## Summary

  The researchers proposed the Binary Star, a coordinated memory hierarchy reliability scheme. The *Binary Star* means the coordinated memory hierarchy reliability design appears like a star system that consists of two “stars” – a 3D DRAM LLC and an NVRAM main memory. The researchers are trying to reduce the overheads brought by the reliability schemes across the 3D die-stacked DRAM last level cache (3D DRAM LLC) and nonvolatile memories (NVRAM).

  The key insight of this work is the fact that employing 3D DRAM LLC and NVRAM main memory typically utilizes two separate uncoordinated reliability schemes. This decoupled reliability across the LLC and main memory is redundant: the same piece of data can be repeatedly and unnecessarily protected multiple times, in LLC and main memory. Thus, the key idea of this work is that carefully protecting only one of the data copies is sufficient.  

  The key mechanism of Binary Star (the coordinated memory hierarchy reliability design)  is substituting the LLC's error-correction with error-detection, and correcting the LLC's error with the data copies in the NVRAM, since errors in the LLC can be corrected by consistent data copies in NVRAM main memory.

  Normally both the 3D DRAM LLC and NVRAM main memory use error checking codes (ECC) to protect data. Different from that, Binary Star lets the LLC adopt only cyclic redundancy check (CRC) codes to ensure errors can be detected. Also, it makes NVRAM maintain consistent data copies that can be used to correct (or avoid) the detected errors in the LLC. To make the data copies in NVRAM consistent with the data in LLC, Binary Star imposes:

  1. Periodic forced writeback

     Binary Star periodically forces the writeback of all dirty cache lines from all cache levels into the NVRAM main memory without invalidation. This movement generates a set of consistent data, i.e., a checkpoint of data, which can be used to recover from LLC errors detected by CRC. The checkpointed data is saved in specific NVRAM blocks. During each periodic forced writeback, Binary Star marks the corresponding NVRAM data blocks as consistent and invalidates the previous one (Binary Star reserves only one checkpoint). In a word, periodic forced writeback backups the data in all dirty cache lines without invalidating them and maintains the data copies (the checkpointed data) in specific NVRAM blocks.

  2. Consistent cache writeback

     This scheme is proposed to guarantee the consistency of the checkpointed data between two forced writeback periods. A normal writeback may corrupt the consistency of the checkpointed data if the data is written back to the same location of  checkpointed data.  This mechanism redirects natural LLC writebacks to NVRAM locations that are different from the locations of the checkpointed data, so that checkpointed data stays consistent.

  The error-correcting procudure of Binary Star:

  If the LLC controller detects an uncorrectable error (an error in a dirty LLC line) Binary Star stalls the application and triggers the rollback procedure. The whole procedure rolls the memory hierarchy back to the consistent checkpointed state and resumes normal application execution. If the LLC controller detects an correctable error (an error in a clean LLC line), the memory controller invalidates the LLC line and issues a cache miss to retrieve the correct copy of the cache line from NVRAM.

  To address the problem that consistent cache writeback imposes substantial overheads (address remapping, free data block management and alternative memory regions management), Binary Star imposes a coordinating wear leveling mechanism. By slightly modifying the existing NVRAM wear leveling mechanism, it only redirects NVRAM updates of each data block once during the interval between two consecutive periodic forced writebacks.

  To implement Binary Star, a computer system needs to modify:

  * 3D DRAM LLC

    Replace the ECC in LLC with CRC.
  
  * Memory controller

    * Modify the Segment Swap design by maintaining a set of metadata, which mark segments for the data from periodic forced writeback and consistent cache writeback.
  * Modify the memory controller's wear leveling control logic to enable the consistent cache writeback mechanism.
    * Add two instructions to support periodic forced writeback and retrieve and roll back to checkpointed consistent data (correct error).

  * System software

    A Binary Star daemon that is in charge of periodic forced writeback and rollback. It can be implemented as a kernel loadable module.

  Binary Star overheads:
  
  * Overheads in 3D DRAM Cache. The CRC-32 code is 50% lower than the storage overhead of traditional ECC.
  * Overheads in NVRAM. With a 512GB NVRAM and a 1MB segment size, the modification to NVRAM introduces 0.0003% storage overhead. The free segment list takes 1.1875MB additional space at most.
* Overheads in Software. Binary Star requires a loadable module to the operating system.
  * Overhead of periodic forced writeback and uncorrectable error-triggered rollback. The periodic forced writeback takes 50µs to 3.6 seconds. The rollback takes dozens of microseconds.

  The evaluation results show that Binary Star provides strong error protection on 3D DRAM LLCs, regardless of technology node and single cell error. Given specific single cell BERs, Binary Star shows the lowest device failure rate, compared to common ECC technologies. On the other hand, Binary Star provides comparable performance to the baseline with 3D DRAM cache and DRAM main memory. Also, the periodic forced writeback interval length is a noticeable variable, because it causes a trade off between performance, reliability, NVRAM endurance and the effective NVRAM capacity.
  
  ## Strengths
  
  * Binary Star is the first memory hierarchy reliability scheme that coordinates the reliability mechanisms across the last-level cache and nonvolatile memory.
* This significantly reduces the performance and storage overhead of consistent cache writeback.
  * The work improves the throughput and reliability of memory LLC and main memory hierarchy at the same time.
* The proposed technique that coordinates the consistent cache writeback technique with NVRAM wear leveling reduces the performance, storage and hardware implementation overheads.
  
  ## Weaknesses
  
* The scheme requires relatively more modifications on different levels: LLC, memory controller, and even operating system, which makes it hard to implement.
  * Imposing more writebacks brings more energy consumption, which is a huge constraint of the computer system.
* Even having wear-leveling schemes, the NVRAM has wear out problems. Bringing more data writes doesn't help maintain its lifetime.
  * Since the Binary Star reduces the error-correcting in LLC, how could it have better reliability than the schemes who have ECC in both LLC and main memory? The paper doesn't give an answer.
  * The Introduction is somehow vague. The writer didn't explain the link between motivations (the reasons why they do such work) and contributions (what they have done) well.
  
  ## Thoughts
  
  * Energy is an important design constraint. Since Binary Star brings more writeback, it will be better to evaluate its energy consumption.
  * This work focuses on the resilience technique to transient errors, hard errors and the Randomly distributed single-cell failure (SCF) of DRAM. Is there any more coordination that we can exploit between LLC and main memory?
  * Is there any more coordination that we can exploit between others' memory hierarchies?
  
  ## Takeaways and questions
  
  * This work talks about the cooperation schemes between memory hierarchies.
  * This work mainly talks about reliability. How about performance? How good will the heterogeneous memory system be, compared to the traditional/ existing memory systems?
  * The reason why Binary Star's reliability is better than the memory hierarchy protected by ECC in both LLC and main memory needs to be discussed.
  * How to guarantee the checkpointed data cover all the dirty lines of LLC during two consecutive periodic forced back? By temporal and spatial locality? 