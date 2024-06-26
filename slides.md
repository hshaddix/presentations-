---
# try also 'default' to start simple
theme: dracula
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS (experimental)
css: unocss
---

# Strip Clustering with HLS for FPGA 

Slides by: Hayden Shaddix 

06/11/2024


<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>
---

# Talk Contents 

- **Introduction** - Little bit about me 
- **Clustering and FPGAs** - How is clustering useful / tying into FPGAs 
- **HLS** - Why HLS? What makes it a good fit for clustering?
- **I/O of Algorithm** - What format will we recieve and output?
- **General Algorithm Overview** - Logic flow and important cases 
- **Testing** - Troubleshooting, Test Bench, optimization 
- **Kernelization** - Progress/Process 
- **Conclusion**

<br>
<br>

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---

# Introduction

<style>
  .two-column {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .left-column {
    flex: 1;
    padding: 20px;
    font-size: 0.8em; /* Smaller font size */
  }
  .right-column {
    flex: 1;
    padding: 20px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .right-column img {
    max-width: 80%; /* Adjust image size */
    height: auto;
    object-fit: contain;
  }
</style>


<div class="two-column">
  <div class="left-column">
    <ul>
      <li>Family person</li>
      <li>Likes to spend time outdoors</li>
      <li>Graduated from FSU in 2023</li>
      <li>Current NIU grad student</li>
      <li>Currently working on QT as part of CP^2 traineeship
        <ul>
          <li>Writing firmware using HLS for use in FPGA pipeline</li>
          <li>Goal is to take in hit data and output clusters of adjacent hits quickly</li>
        </ul>
      </li>
    </ul>
  </div>
  <div class="right-column">
    <img src="Hayden.png" alt="Me (bottom right) and family">
  </div>
</div>
--- 

# Clustering and FPGAs
<style>
  .two-column {
    display: flex;
    align-items: center;
  }
  .column {
    flex: 1;
    padding: 10px;
  }
  .column img {
    width: 100%;
    height: auto;
    object-fit: contain;
  }
  .caption {
    text-align: center;
    margin-top: 10px;
    font-size: 0.8em;
  }
</style>

<div class="two-column">
  <div class="column">
    <p>Clustering is an important part of any detector pipeline; some of the most important features of clustering especially in an FPGA context is it:</p>
    <ul>
      <li>Reduces volume of data</li>
      <li>Filters out noise / sorts through input stream</li>
      <li>Processes and outputs compact clusters in real-time</li>
      <li>Is much more efficient</li>
      <li>Fits well in a scaleable pipeline</li>
    </ul>
  </div>
  <div class="column">
    <img src="FPGA.png" alt="Diagram of an FPGA">
  </div>
</div>

<div class="caption">
  <a href="https://tac-hep.org/assets/pdf/uw-gpu-fpga/2023-03-22-FPGA-HLS-Lecture-2.pdf" target="_blank">Link to slides: FPGA-HLS Lecture</a>
</div>
---

# HLS 
Clustering algorithm is written in HLS for several reasons:

- Abstracts Complexity: written more similarly to a high-level language (C/C++)
- Parallelism: HLS allows us to handle multiple data stream simultaneously
- Testability: Quick iterations and testing 
- Customizable Hardware: Easy to change if project demands change  

Have continued to learn HLS pretty actively using online manuals and resources: 

<style>
  .split {
    display: flex;
    justify-content: space-between;
  }
  .left {
    flex: 1;
    padding-right: 20px;
  }
  .right {
    flex: 1;
    padding-left: 20px;
  }
  .right img {
    width: 100%;
    height: auto;
  }
</style>

<div class="split">
  <div class="left">
    <!-- Left side content with link -->
    <p> <a href="https://docs.amd.com/r/en-US/ug1399-vitis-hls/Tutorials-and-Examples">Vitis AMD Tutorials</a></p>
  </div>
  <div class="right">
    <!-- Right side content with screenshot -->
    <img src="AMD.png">
  </div>
</div>

---

# I/O 
Since this is a piece in a larger pipeline of other kernels in the FPGA, the inputs and outputs are important

<style>
  .slide-small-text {
    font-size: 0.8em; /* Adjust the size as needed */
  }
</style>

<div class="slide-small-text">

**Inputs**
- Clusters with a position and bitmask 
  - <u>Position</u>
    - ABCStar chip 
    - Strip number 
  - <u>Bitmask</u> : Map of following 3 hit positions 

**Outputs**
- Clusters with position and size 
  - <u>Position</u>
    - ABCStar chip 
    - Strip number 
  - <u>Size</u>
- Output is based off of adjacency of hits in completely local coordinates 
</div>

---

# Clustering Algorithm 

<style>
  .two-column {
    display: flex;
  }
  .column {
    flex: 1;
    padding: 10px;
  }
  .column img {
    width: 100%;
    height: auto;
  }
</style>

<div class="two-column">
  <div class="column">
    <!-- Left column content -->
    <h2>Code Logic Overview</h2>
    <p>The HLS algorithm is relatively straightforward in its general structure:</p>
    <ol>
      <li>Takes in cluster information</li>
      <li>Checks adjacency for bitmask cases</li>
      <li>Merges adjacent hits</li>
      <li>Creates new cluster when a hole exists</li>
      <li>Outputs clusters of adjacent hits</li>
    </ol>
  </div>
  <div class="column">
    <!-- Right column content with image -->
    <img src="Flowchat.png">
  </div>
</div>

---
---
# Implementation/Application 

<style>
  .two-column {
    display: flex;
  }
  .column {
    flex: 1;
    padding: 10px;
  }
  .column img {
    width: 100%;
    height: auto;
  }
</style>

<style>
  .two-column {
    display: flex;
  }
  .column {
    flex: 1;
    padding: 10px;
  }
  .column img {
    width: 100%;
    height: auto;
    object-fit: contain;
  }
  .image-with-caption {
    text-align: center;
    margin-bottom: 20px;
  }
  .image-with-caption img {
    max-width: 100%;
  }
  .image-with-caption p {
    margin-top: 5px;
    font-size: 0.9em;
    color: #555;
  }
</style>

## Nuanced Cases in Strip Clustering

<div class="two-column">
  <div class="column">
    <!-- Left column content -->
    <p>There are a few interesting cases that need to be treated explicitly:</p>
    <ol>
      <li>Gap Cases (holes in the bitmask)</li>
      <li>Clusters across chip boundaries</li>
    </ol>
    <p>Simpler than pixels, but some nuance depending on the final necessary specifications:</p>
      <ul>
        <li>Overlapping hits</li>
        <li>Merge chip boundary or truncate into separate small clusters</li>
        <li>Smallest possible adjacent clusters (no gaps)</li>
      </ul>
  </div>
  <div class="column">
    <!-- Right column content with images and captions -->
    <div class="image-with-caption">
      <img src="AcrossBoundary.png" alt="Merging across ABCStar boundary">
      <p>**Merging across ABCStar boundary**</p>
    </div>
    <div class="image-with-caption">
      <img src="Overlap.png" alt="Case with two separate input clusters with overlapping hits">
      <p>**Case with two separate input clusters with overlapping hits**</p>
    </div>
  </div>
</div>
---
---

# Testing and Optimization

There are a few important things testing this code as it is developed 

1) C Simulation (Test Bench) 
    - Includes several simple cases defined by the user 
    - Easily manipulated
    - Simple testing of input and outputs 
2) C Synthesis (Vitis)
    - Checks viability of pipelining 
    - Gives practical readings (Latency/clock-cycle, etc)
    - Allows for and suggests optimizations 
3) Test vectors (not yet used)
    - More realistic input 
    - Greater length/variability input stream 

--- 

# Kernelization 
Short long-term goal is for a confined clustering kernel to test with larger FPGA pipeline. From my understanding, there are 3 major components necessary to build the kernel and my current working status on each varies: 

<style>
  .two-column {
    display: flex;
    align-items: flex-start;
  }
  .column {
    flex: 1;
    padding: 10px;
  }
  .column img {
    width: 100%;
    height: auto;
    object-fit: contain;
  }
  .image-with-caption {
    text-align: center;
    margin-bottom: 20px;
  }
  .image-with-caption img {
    max-width: 100%;
  }
  .image-with-caption p {
    margin-top: 5px;
    font-size: 0.9em;
    color: #555;
  }
</style>

## Project Components and Status

<div class="two-column">
  <div class="column">
    <!-- Left column content with table -->
    <table>
      <thead>
        <tr>
          <th>Components</th>
          <th>Current Status</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Working HLS</td>
          <td>Still testing, optimization has been on the backburner</td>
        </tr>
        <tr>
          <td>Test Bench</td>
          <td>Created already, should be easily adaptable for different cases</td>
        </tr>
        <tr>
          <td>Project file</td>
          <td>Skeleton version in the works, adapting tutorial version to suit the clustering code</td>
        </tr>
      </tbody>
    </table>
  </div>
  <div class="column">
  </div>
</div>

---

# Conclusion 

<style>
  .small-text {
    font-size: 0.8em; /* Adjust the size as needed */
  }
</style>

<div class="small-text">
  <p>This clustering code and the following kernel will be an important part of the FPGA EF Tracking Pipeline. The major facets of the work I have and will be continuing with are:</p>
  <ol>
    <li>Developing HLS code for strip clustering
      <ul>
        <li>Addressing specific cases</li>
        <li>Working cooperatively to determine what is required at a given stage of development</li>
      </ul>
    </li>
    <li>Testing HLS code
      <ul>
        <li>Test bench and trivial cases</li>
        <li>Ensuring parallel pipelining</li>
        <li>Using realistic input to test algorithm</li>
      </ul>
    </li>
    <li>Kernalization/Optimization
      <ul>
        <li>Creating a kernel</li>
        <li>Optimizing the workflow and structure</li>
        <li>Paying attention to specific needs of the overall pipeline and connecting my project</li>
      </ul>
    </li>
  </ol>
  <p>I am looking forward to continuing to work with you all to get things in order!!!</p>
</div>

