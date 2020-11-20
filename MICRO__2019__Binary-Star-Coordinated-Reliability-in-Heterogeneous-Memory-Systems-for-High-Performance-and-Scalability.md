* # Zhongfa Wang 2020/11/20

  # Paper Information

  * Title: Binary Star: Coordinated Reliability in Heterogeneous Memory Systems for High Performance and Scalability

  * Authors: Xiao Liu; David Roberts; Rachata Ausavarungniru; Onur Mutlu; Jishen Zhao

  * Venue: MICRO 2019
  * Keywords: memory hierarchy design, 3D die-stacked DRAM, nonvolatile memories, comprehensive coordinated memory hierarchy reliability scheme, 

  # Paper content

  ## Summary

  The researchers proposed a Binary Star, a coordinated memory hierarchy reliability scheme. The researchers is trying to: first,reduce the overheads brought by the reliability schemes of 3D die-stacked DRAM and nonvolatile memories (NVRAM); second, the challenges of exploiting persistent memory techniques. ( Firstly, the overheads. The requirements of larger memory and cache demand process scaling. However, the process scaling brings randomly distributed single-cell failure (SCF) to whom the counter measurements (in-DRAM ECC and rank-level ECC) bring significant storage overheads in DRAM chips. Also, the Through Silicon Vias (TSVs) of the 3D DRAM LLC increase the BER. On another hand, most of the NVRAM have much lower endurance (which means they can wear out). Thus the NVRAM needs to adopt hard error protections, which bring high overheads in both performance and storage. All of these technologies, DRAM, 3D die-stacking and NVRAM need reliability schemes respectively. As a result, a system employing 3D DRAM LLC and NVRAM needs to utilize separate reliability schemes for each of them, which brings redundant protection.  Secondly, exploiting persistent memory (NVRAM) has two disadvantages: first, it require code modifications, which require significant software engineering effort; second applications with no persistence requirements suffer from substantial performance and storage overheads in main memory access.) The Binary Star means the coordinated memory hierarchy reliability design appears like a star system that consists of two “stars” – a 3D DRAM LLC and an NVRAM main memory.

  The key insight of this work is the fact that employing 3D DRAM LLC and NVRAM main memory typically utilizes two separate uncoordinated reliability schemes. This decoupled reliability across the LLC and main memory is redundant: the same piece of data can be repeatedly and unnecessarily protected multiple times, in LLC and main memory. Thus, the key idea of this work is that carefully protecting only one of the data copies is sufficient.  

  In Binary Star, the LLC only detects errors by using cyclic redundancy check (CRC) and NVRAM can correct the data, based on the observation that errors in the LLC can be corrected by consistent data copies in NVRAM main memory. For the scenario that data in LLC and memory are inconsistent, the researches propose *periodic forced writeback* and *consistent cache writeback*. The *periodic forced writeback* periodically forces the writeback of dirty cache lines from all cache levels into the NVRAM main memory and maintain an additional copy of data-checkpoint. The *consistent cache writeback* maintains consistency of the checkpointed data between two forced writeback periods.  On another hand, the consistent cache writeback brings heavy performance and NVRAM storage overheads. Thus,  based on the observation that NVRAM wear leveling naturally redirects and maintains the remapping of data updates to alternative memory locations, the Binary Star leverages the remapping techniques that are already employed by existing NVRAM wear leveling mechanisms. At last, when error occurs in DRAM LLC, the researches proposed error correction and recovery schemes too.

  From all these schemes, the Binary Star can eliminate the error correction codes in the LLC and achieves higher reliability than using ECC mechanisms.

  ## Strengths

  * This work focuses on emerging technologies, in other words, 3D DRAM last-level cache and nonvolatile main memory while prior works only handle SRAM cache and DRAM main memory.
  * This work considered reaping the full reliability advantages of them beyond simply replacing the existing memory technologies.
  * The researcher develop a set of new software-hardware cooperative mechanisms to make the Binary Star transparent to the applications.

  ## Weaknesses

  * The Introduction is somehow vague. The writer didn't explain the link between motivations (the reasons why did they do such work) and contributions (what they have done) well.
  * The writer didn't mention the link between DRAM main memory and 3D DRAM LLC. At some part of the paper the writer used DRAM to denote DRAM main memory, which makes the words more vague: is the 3D DRAM LLC the DRAM mentioned before?
  * The paper didn't give the full name of some abbreviations. For example, RECC, PCM and BER.

  ## Thoughts

  * I think it's better to make the introduction part clear. 
  * We should give the full name when an abbreviation occurs at the first time.
  * Energy is am important designing constraint. Since the Binary Star brings more writeback, it will be better to evaluate its energy consumption.

  ## Takeaways and questions

  * I learned knowledge about memory hierarchy: cache, last-level cache and main memory. Also I make clear some of the cache operations, such as writeback.
  * This work mainly talks about reliability. How about performance? How good in performance will the heterogeneous memory system be, comparing to the traditional/ existing memory systems?