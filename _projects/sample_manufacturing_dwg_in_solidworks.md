---
layout: page
title: Sample Manufacturing Drawing in Solidworks
description: Modeled and generated drawings for manufacturing clarity
img: assets/img/mfg_part.png
importance: 2
category: Undergrad
---

This simple part was created for portoflio entry purposes and contains no proprietary data. The drawing is an example of how I would use Solidworks to model parts and generate drawings that may have a customer drawing, but also rely on specifications and other documents such as P.O. line items to alter the final dimensional or processing requirements of a part. These drawings allow for better understanding on the shop floor, and better configuration management as each change not included in a revision change on the customer’s drawing, will be including in a revision change on the manufacturing blueprint. It also provides an easy drawing to inspect to, or to send to outside vendors if machining cannot be done in-house.

Typically, this will include adding specification features (e.g. MIL-STD) rather than expecting a machinist to look up values. It is also generally helpful to include tolerances for each feature rather than having the reader refer to the tolerance block. Finally, for complex parts (not shown in this example) the part may be modeled in stages, starting from raw block form and showing each step in the machining process (e.g. first turn, milling of features, final turn).

<div class="pdf-container" style="height: 600px;">
  <embed src="{{ '/assets/pdf/mfg_dwg.pdf' | relative_url }}" 
         type="application/pdf" 
         width="100%" 
         height="100%">
</div>