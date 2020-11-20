# Zhongfa Wang 2020/11/9

# Paper Information

* Title: ComputeDRAM: In-Memory Compute Using Off-the-Shelf DRAMs
* Authors: Gao, Fei; Tziantzioulis, Georgios; Wentzlaff, David
* Venue: MICRO_2019
* Keywords: DRAM, in-memory computing, bit-serial, main memory

# Paper content

## Summary

This is an article about a processing-in-memory technique, conputeDRAM. Usually DRAM memory controllers have nominal operations of specific timing sequences. The technique exploits the side-effects of non-nominal operations of DRAM modules. The proposed computeDRAM can perform logic AND/OR and row copy in memory by violating the DRAM timing constraints. It can perform these operations on off-the-shelf, unmodified commerical DRAM. Since only logic AND/OR and row copy fall short of performing arbitrary computations, the researchers proposed a software framework that could perform arbitrary computations by using the three operations. Evaluation and discussion show that this technique suits massive computations that exhibits a large amount of main memory access and has very high energy efficiency.

## Strengths

* The proposed ComputeDRAM can work on off-the-shelf, unmodified, commercial DRAM, which means it cost minimal hardware design changes.
* The proposed ComputeDRAM suits the modules from all major DRAM vendors in a wide range of voltage and temperature.
* The throughput and performance of the ComputeDRAM is rised when the parallelism and the scale of computation are high. 
* The ComputeDRAM has high energy efficiency in specific operations (logic AND, logic OR and row copy).

## Weaknesses

+ Only when the parallelism and the scale of computation are high is the throughput and performance of the ComputeDRAM high.  Its application scenario is limited.
+ It needs to address the duplicated computation and storage at compiler level, which means the implement of ComputeDRAM needs an update of not only DRAM controller but also compliers.

## Thoughts

* It's a good idea to avoid data movements by processing-in-memory.

## Takeaways and questions

* Since CPU's is complex ( it can perform branch prediction and others, it has complex ISA) and memory and storage have always been a bottleneck of CPU performance, is it a good idea to leave the high throughput tasks to other parts like memory?
* At what rate are the high throughput tasks among all tasks of CPU? Does it worth to research on it?