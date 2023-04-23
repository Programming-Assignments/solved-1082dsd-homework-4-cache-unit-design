Download Link: https://assignmentchef.com/product/solved-1082dsd-homework-4-cache-unit-design
<br>
In a typical digital system design, operations are not executed directly on the data stored in main memory. Instead, processors load data into their own registers first, do operations on the copy of data in the register, and finally store the result back into main memory. Hence, processors need to communicate with main memory frequently. Since the clock rate of main memory is slower than that of the processor, such communications are usually the bottleneck of a digital system. For the purpose of facilitating the performance, caches are introduced between processors and main memory to take advantage of locality, and thus significantly decrease the miss rate. In this homework, you are asked to design cache units on your own.




Figure 1 shows a pipelined MIPS with both data memory and instruction memory included. Please implement two kinds of cache both containing <strong>8 blocks with 4 words in each block</strong>, one of them in<strong> direct-mapped </strong>and the other in<strong> two-way associative</strong> architecture. Both types of the write policy, direct write-through and write-back, are available so that you may choose one by yourself. The I/O specification and interface of the desired 32-word cache unit are illustrated in Figure 2.




<strong>Figure 1. </strong> Pipelined MIPS processor and the memory interface.

<strong>Figure 2. </strong> I/O specification and interface of the cache unit.




<ol start="2">

 <li><strong>I/O Specification of the Processor Interface</strong></li>

</ol>

<table width="555">

 <tbody>

  <tr>

   <td width="128"><strong>Name </strong></td>

   <td width="38"><strong>I/O </strong></td>

   <td width="58"><strong>Width </strong></td>

   <td width="331"><strong>Description </strong></td>

  </tr>

  <tr>

   <td width="128">clk</td>

   <td width="38">I</td>

   <td width="58">1</td>

   <td width="331">positive edge trigger clock</td>

  </tr>

  <tr>

   <td width="128">proc_reset</td>

   <td width="38">I</td>

   <td width="58">1</td>

   <td width="331">synchronous active-high reset signal</td>

  </tr>

  <tr>

   <td width="128">proc_read</td>

   <td width="38">I</td>

   <td width="58">1</td>

   <td width="331">synchronous active-high read enable signal</td>

  </tr>

  <tr>

   <td width="128">proc_write</td>

   <td width="38">I</td>

   <td width="58">1</td>

   <td width="331">synchronous active-high write enable signal</td>

  </tr>

  <tr>

   <td width="128">proc_addr</td>

   <td width="38">I</td>

   <td width="58">30</td>

   <td width="331">address bus (<strong><em>word address</em></strong>)</td>

  </tr>

  <tr>

   <td width="128">proc_wdata</td>

   <td width="38">I</td>

   <td width="58">32</td>

   <td width="331">data bus for writing to cache</td>

  </tr>

  <tr>

   <td width="128">proc_rdata</td>

   <td width="38">O</td>

   <td width="58">32</td>

   <td width="331">data bus for reading from cache</td>

  </tr>

  <tr>

   <td width="128">proc_stall</td>

   <td width="38">O</td>

   <td width="58">1</td>

   <td width="331">active-high control signal that asks processor to wait (cache is busy)</td>

  </tr>

 </tbody>

</table>




<ol start="3">

 <li><strong>I/O Specification of the Memory Interface</strong></li>

</ol>

<table width="555">

 <tbody>

  <tr>

   <td width="130"><strong>Name </strong></td>

   <td width="40"><strong>I/O </strong></td>

   <td width="59"><strong>Width </strong></td>

   <td width="326"><strong>Description </strong></td>

  </tr>

  <tr>

   <td width="130">mem_read</td>

   <td width="40">O</td>

   <td width="59">1</td>

   <td width="326">synchronous active-high read enable signal</td>

  </tr>

  <tr>

   <td width="130">mem_write</td>

   <td width="40">O</td>

   <td width="59">1</td>

   <td width="326">synchronous active-high write enable signal</td>

  </tr>

  <tr>

   <td width="130">mem_addr</td>

   <td width="40">O</td>

   <td width="59">28</td>

   <td width="326">address bus (<strong><em>4-word address</em></strong>)</td>

  </tr>

  <tr>

   <td width="130">mem_wdata</td>

   <td width="40">O</td>

   <td width="59">128</td>

   <td width="326">data bus for writing to slow memory</td>

  </tr>

  <tr>

   <td width="130">mem_rdata</td>

   <td width="40">I</td>

   <td width="59">128</td>

   <td width="326">data bus for reading from slow memory</td>

  </tr>

  <tr>

   <td width="130">mem_ready</td>

   <td width="40">I</td>

   <td width="59">1</td>

   <td width="326">asynchronous active-high one-cycle signal that indicates data arrives from memory / data is done written to memory</td>

  </tr>

 </tbody>

</table>

<h1>4.Address Mapping</h1>

A 32-word cache consists of 8 blocks, each containing 4 words. The processor accesses one word at a time, while the cache unit can only access 4 words at a time from the memory. Therefore, the last (rightmost) 2 bits of <em>proc_addr</em> represent the <em>offset</em> inside one block, indicating which word of the 4 words should be the target. The rest 28 bits are simply the <em>mem_addr</em> while accessing the memory. Figure 3 illustrates the address mapping between the processor, the memory, and the cache unit.




<strong>Figure 4.</strong>  Address mapping style of two architectures




Reduce cache misses by adopting more flexible block-placement schemes:

<ul>

 <li>Direct mapped: A block can go in exactly one place in the cache.</li>

 <li>Fully-associative: A block can be placed in any location.</li>

 <li>Set-associative: A block can be placed in a restricted set of locations.</li>

</ul>




In a direct-mapped cache, the position of a memory block is given by

(block number) modulo (number of cache blocks)

In a set-associative cache, the set containing a memory block is given by

(block number) modulo (number of sets in the cache)

<h1>5.Functional and Timing Specification of the Processor Interface</h1>

In this homework, the processor interface is a pseudo processor that only performs the memory accessing tasks in order to verify the functionality of the cache unit. Therefore, all the signals from the processor interface change at the rising clock edge with a very slight input delay.




From time to time, the cache needs to access data from the memory and then wait for several cycles. Whenever this is the case, the <em>proc_stall</em> signal should be set high to stall the processor. The <em>proc_stall</em> signal should be set high before the next positive clock edge so that the processor can be correctly stalled. On the other hand, when there is no need to access memory, the cache simply finishes the job in one cycle. For example, in the read case, <em>proc_rdata</em> should be prepared and returned immediately to the processor at the same cycle if the required data are right available.




There are two possible cases where a stall is necessary:  (a) <em>Read miss</em> in both write-through and write-back caches; (b) <em>Write hit</em> in <em>only write-through</em> caches.

Note that since <em>write miss</em> can be regarded as “<em>read miss then write hit</em>”, a write miss in write-through caches involve two successive stalls, while only one stall is needed in write-back caches.




The pseudo processor in the given testbench performs only three tasks. It reads the whole initial data from the memory, rewrites the whole memory with new data, and finally reads the rewritten data back from the memory. All possible cases are taken into account, so you should design the finite state machine in the cache unit carefully and neatly.




<h1>6.Functional and Timing Specification of the Memory Interface</h1>

The memory interface is the functional model of an ordinary memory or a system bus. Figure 5 shows the timing diagram of the memory interface. After the memory is set to the read or write mode, it takes several cycles to complete the operation. The <em>mem_ready</em> signal serves as an indicator, saying that the required data is ready in read mode or the data is written correctly in write mode. If the <em>mem_read</em> and <em>mem_write </em>signals are both set high or both set low, the memory would do nothing.







<ul>

 <li>Read operation.</li>

</ul>




<ul>

 <li>Write operation.</li>

</ul>

<strong>Figure 5.</strong>  Timing diagram of the memory interface.




<h1>7.Homework Requirements</h1>

<strong>Verilog Code. </strong>

Verilog code named “<strong>cache_dm.v</strong>” and “<strong>cache_2way.v”</strong>, for the direct mapped and 2way associative cache units respectively, are required. You need to synthesize your design, and make sure there are <em>no latches, timing violations, or negative endpoint slack</em>, which are things not allowed in a synthesizable design. Performance is not part of the grading criteria this time, so you are not required to optimize your design.




<strong>Post Synthesis Related Files.</strong>

These include SDF files, DDC files, and the gate-level design Verilog code. (Six files in total. See Submission Requirement.)




<strong>Report.  </strong>

The report should be named “<strong>report.pdf</strong>” in <strong>PDF</strong> format, and should contain at least the following parts:

<ul>

 <li>The cycle time you used in cache_syn.sdc and in tb_cache.v to pass the postsynthesis simulation respectively.</li>

 <li>General specification of the cache unit.</li>

</ul>

(such as numbers of words, placement policy, and so on);

<ul>

 <li>Read/write policy (write-through or write-back);</li>

 <li>Design architecture or the finite state machine of the cache unit;</li>

 <li>Performance evaluation of your cache design, including the miss rates of read/write operations, the execution cycles, the stalled cycles, and so on;</li>

</ul>

※ Hint: Try to add a few lines in tb_cache.v for calculating the miss rates, the execution cycles, and the stalled cycles on your own. Just make sure you fully understand the different conditions of I/O signals when a cache hits or misses.

<ul>

 <li>Compare the performance of the two architectures, and discuss the reasons for such results.</li>

</ul>

We would appreciate it if you give a discussion on the cache architectures, or describe your design guideline in details and the experiences you gain from this homework.


