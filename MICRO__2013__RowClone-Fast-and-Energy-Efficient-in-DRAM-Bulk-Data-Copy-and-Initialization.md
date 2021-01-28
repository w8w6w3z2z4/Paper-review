# Zhongfa Wang (2021-01-24)

# Paper information
- Title: RowClone: Fast and Energy-Efficient In-DRAM Bulk Data Copy and Initialization
- Authors: Vivek Seshadri, Yoongu Kim, Chris Fallin, Donghyuk Lee, Rachata Ausavarungnirun, Gennady Pekhimenko, Yixin Luo, Onur Mutlu, Phillip B. Gibbons, Michael A. Kozuch, Todd C. Mowry
- Venue: MICRO 2013
- Keywords: DRAM, Bulk data copy, Bulk data initialization, In-Memory processing, latency, bandwidth, energy proficiency

# Paper content
## Summary

The main memory subsystem is an essential bottleneck of system performance and energy efficiency. Bulk data copy and initialization, also named as bulk data operations, incur high latency, consume high bandwidth and energy, which degrades system performance and energy efficiency. Because using memory channels to accomplish bulk data operations brings these problems, the authors try to accomplish these operations not across memory channels but completely within a DRAM chip. To this end, they proposed RowClone. RowClone is a mechanism that introduces a new way to implement bulk data operations completely within DRAM. 

One key idea of this work is to transfer data between two rows in the same bank and same subarray, via the shared bitline and sense amplifiers. This idea is based on the observation: *if a DRAM cell is connected to a bitline that is at a stable state (either V<sub>DD </sub> or 0) instead of the equilibrium state (0.5V<sub>DD </sub>), then the data in the cell is overwritten with the data (voltage level) on the bitline.* So it is possible to copy data to the destination row (dst) if the bitline and row buffer are at stable state, same as the source row (src). Another key idea of this work is to transfer data between two rows in different banks via the single internal bus. This idea is based on the observation that *a single internal bus is shared across all banks within a chip to perform both read and write operations.* So it is possible to copy data from src to dst via the internal bus.

The key mechanism of RowClone is across internal channels (bitlines or internal bus) instead of memory channels to accomplish bulk data operations. The RowClone contains two modes:

1. Fast Parallel Mode (FPM)

   The objective of FPM is using bitlines within a subarray to perform bulk data operations. FPM lowers the wordline of the src and then raises the wordline of the dst. Meanwhile, FPM keeps the bitlines and row buffers at stable state. Based on the first key idea and observation, if row buffers and bitlines are at stable stage, they overwrite the dst with the same voltage state as the src. So after the src is activated, the bitlines and row buffer are at stable stage, same as the src. After the dst is activated, it will be overwritten by row buffer and be in the same state with the src. FPM needs subarray IO logic to support back-to-back ACTIVATE commands. 

2. Pipelined Serial Mode (PSM) 

   The objective of PSM is using internal bus between different banks to perform bulk data operations. PSM activates two rows from different banks and then puts the source bank in the read mode and the destination bank in the write mode to copy data in a pipelined way. Based on the second key idea and observation, the dst can be overwritten with data of the src via the internal bus. PSM needs a new command TRANSFER. When executing TRANSFER, the chip IO disconnects the internal bus to the memory channel. So different from WRITE/ READ commands, TRANSFER doesn't transfer data out of/into chips. It only copies data from one subarray to another.

Bulk data copy contains three possible cases: src and dst are within the same subarray;  in different banks; in different subarrays within the same bank. The first case and second case can be accomplished by FPM and PSM respectively. For the third case, RowClone uses PSM to first copy the data from src to a temporary row (tmp) in a different bank and then copy the data back to dst. For bulk data initialization, RowClone first initializes a single DRAM row and then uses the copy mechanism mentioned before to  copy the data to other rows.

To implement RowClone, a computer system needs to modify: the ISA, processor microarchitecture support and system software.

* ISA support

  Two new instructions are introduced: memcopy and meminit. The memcopy has operands *src*, *dst* and *size*. It's semantics is copying *size* bytes from *src* to *dst*. The meminit has operands *dst*, *size* and *val*. It's semantics is set *size* bytes to *val* at *dst*.

* Processor microarchitecture support
  * Source/destination alignment and size

    For an operation using FPM mode, the source and destination regions should be in same subarray and be row-aligned. Also the operation should span an entire row. For an operation using PSM, the source and destination regions should be cache line-aligned and the operation must span a full cache line. Thus when there is a mecopy/ meminit instruction, the processor divides the region to be copied/initialized into three portions: 1) row-aligned row-sized portions for FPM, 2) cache line-aligned cache line-sized portions for PSM and 3) the remaining portions.

  * Managing on-chip cache coherence

    The proposed mechanism so far modifies data in memory without going through cache, which makes data between memory and corresponding cache incoherent.  So it is necessary to make data in those two positions coherent. When the memory controller finds out that there are dirty cache lines corresponding to the source rows while performing a copy, it copies the data of that cache lines to the cache lines corresponding to the destination rows in memory.

* Software support

  * Subarray-aware page mapping

    The OS should have knowledge of the memory page, i.e., which pages map to the same subarray in DRAM.

  * Granularity of copy/initialization

    The minimum granularity of FPM copy is the size of row. When copying a contiguous region with interleaving strategies, the minimum granularity of FPM is the product of the row size and the number of channels.

One contribution of this work is that the authors analyzed the internal architecture of DRAM and show that the architecture can be exploited to perform bulk data operations with reduced latency, memory bandwidth and energy consumption. They also  proposed RowClone, the simple and low-cost mechanism to perform bulk data operations. The authors give out the specific back-to-back system design.

The key conclusion is that the proposed RowClone significantly reduces memory bandwidth and energy consumption in tasks containing bulk data operations. Thus this work is effective in improving both system performance and energy efficiency for future, bandwidth-constrained, multi-core systems.

## Strengths

* Successfully exploited DRAM's internal architecture and found out that shared bitlines and internal bus can be used to transfer data between destination rows and source rows.
* The proposed RowClone significantly reduces the memory bandwidth cost and energy cost when performing bulk data operations. And then this work can contribute to bandwidth-constrained, multi-core systems.


## Weaknesses
* The work didn't give a solution to copy data from a memory chip to another. Since a memory module has more than one memory chip, it is possible that there exists a need to copy data from one chip to another.

## Paper presentation
* In the abstract section, there is a statement: "in this work, we propose RowClone, a new and simple mechanism to perform bulk copy and initialization *completely within DRAM*". The expression, completely within DRAM, means there are no operations outside the main memory. But as words in section 4.2.2, RowClone creates an in-cache copy, which means RowClone brings operations in cache. So the expression *completely within DRAM* is incorrect and misleading.
* A variant of RowClone, RowClone-Zero-Insert (RowClone-ZI) is introduced in the evaluation section (section 7.4). RowClone-ZI is proposed to solve the problem that RowClone has low memory-level parallelism so as to degrade performance of zeroing pages. It's better to illustrate this in the detailed design (section 3) and the End-to-end system design section (section 4).

## Thoughts

* It will be better to evaluate the reliability. Since bulk data operations operate massive data, a little error rate may cause lots of error. So, a reliability evaluation seems necessary. 
* The PSM is not as good as the FPM in improving system performance. Because all the rows within a subarray are connected to the same bitlines, so the FPM can perform copy in a parallel way. But for the rows within different banks, with the existence of bank IO, the data transfer should be performed in a serial way. So for the bulk data operations, exploiting parallelism brings benefits.

## Takeaways and questions
* This work gives me a deeper insight of DRAM. It also triggers me that we can exploit the internal architecture of components to optimize system's performance.
* How is the prospect of near-memory computing technology in graphic memory? On NVIDIA's new Ampere graphic cards (RTX 3090, RTX 3080 and others), the energy consumption of graphic memory is a huge problem. The Graphics Double Data Rate 6x (GDDR6x) memory consumes so much energy as the GPU doesn't have enough power. Since graphic computation on GPU is also a bandwidth-constrained work. The application section (section 5.2) mentioned the communication between CPU and GPU. But here the question is about the operations within memory on a graphic card.

## Appendix: The DRAM Background

A DRAM chip is divided into several banks which are connected by shared internal bus. At the terminal of the bus is the chip I/O.  A bank can be further divided into several subarrays connected by global bitlines. A subarray consists of a 2-D array of DRAM cells and a sense amplifier (row buffer). The cell is the basic unit of DRAM. It consists of a capacitor and an access transistor. When the capacitor is fully charged, the cell represents logic 1 and when the capacitor's charge is fully drained, it represents logic 0.  The access transistor determines whether the cell is being accessed. The wire connecting the DRAM cell is bitline and the wire controlling the access transistor is wordline. At the initial *precharged state*, The wordlines are lowered. The bitlines and sense amplifiers are pre-charged to half the V<sub>DD </sub> (maximum voltage). When the memory controller issues an ACTIVATE command, the wordline raises and the transistor turns on. Thus the cells connected to the transistor can access the corresponding bitlines, which then lose/gain charge, thereby raising/lowering the voltage of the bitlines. The sense amplifier can sense the voltage derivation and amplify it, and drive the cell back to its original voltage. When the amplification is finished, the voltage of the bitline and the output voltage of the sense amplifier are the same as the initial voltage of the corresponding cell. This state is called a stable stage.  After the access finishes, the controller issues a PRECHARGE command to lower the wordline and derives the bitlines and sense amplifiers to the  *precharged state*, i.e., 0.5V<sub>DD </sub>.