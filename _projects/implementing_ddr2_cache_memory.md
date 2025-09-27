---
layout: page
title: Implementing DDR2 Cache Memory
description: Implementing DDR2 Cache Memory using Xilinx Memory Interface Generator in Pipeline MIPS.
img: assets/img/ddr2_cache.png
importance: 2
category: Grad
---

As my final project for my Computer Architecture class, I took a pipelined MIPS processor with a memory cache implemented on the FPGA fabric, and replaced it with DDR2 memory using the Xilinx Memory Interface Generator (MIG) for the Nexys A7 boards.The project ultimately did not reach its stated goals. After collaboration both with Dr. Beser as well as a subject matter expert from Johns Hopkins, we concluded that a potential glitch in the current version of Vivado (or something additionally unknown) was the issue, and the project was carried over to be completed by another student next semester, the ultimate goal being to integrate this into the class processor code. Despite this setback, the framework for a working cache was completed and I created a presentation and report based on the process and my analysis of the results.

Final paper is available as a PDF.

<div class="pdf-container" style="height: 600px;">
  <embed src="{{ '/assets/pdf/ddr2_mig_report.pdf' | relative_url }}" 
         type="application/pdf" 
         width="100%" 
         height="100%">
</div>