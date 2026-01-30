---
title: "Visualizing Forensics: The TLN Vim Profile"
date: 2026-01-30
draft: false
---

## The Problem: Timeline Fatigue

In major incident response engagements, "Excel fatigue" is real. staring at a CSV with 5 million rows from `log2timeline` is a quick way to miss lateral movement.

## The Solution: A Vim-Based Workflow

I developed a workflow to process disk images while keeping the analysis environment lightweight and keyboard-centric.

### 1. Extraction
Processing the raw disk image using **Plaso** to generate a bodyfile.

### 2. Conversion
Converting the bodyfile to the **TLN (Time Line)** format. This format is superior for quick scanning because of its epoch-time sorting and pipe-delimited structure.

### 3. Visualization (The Secret Sauce)
I wrote a custom **Vim profile** that applies syntax highlighting based on the content of the TLN file.

* **Red:** Execution artifacts (Prefetch, Shimcache).
* **Yellow:** File system modifications.
* **Blue:** Network connections.

This allowed me to "scroll" through a compromise at the speed of Vim (`j`, `k`, `Ctrl+D`), spotting anomalies purely by color patterns in the terminal.