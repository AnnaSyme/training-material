---
layout: tutorial_hands_on

title: "Chloroplast genome assembly"
zenodo_link: "http://doi.org/10.5281/zenodo.3567224"
questions:
  - "How can we assemble a chloroplast genome?"
objectives:
  - "Assemble a chloroplast genome from long reads"
  - "Polish the assembly with short reads"
  - "Map reads to the assembly and view"
  - "View an annotated assembly"
time_estimation: "2h"
key_points:
  - "A chloroplast genome can be assembled with long reads and polished with short reads"
  - "The assembly graph is useful to look at and think about genomic structure"
  - "We can map raw reads back to the assembly and investigate areas of high or low read coverage"
  - "We can view an assembly, its mapped reads, and its annotations in JBrowse"
contributors:
  - annasyme
---

# Introduction
{:.no_toc}

## What is genome assembly?

Genome assembly is the process of joining together DNA sequencing fragments into longer pieces, ideally up to chromosome lengths.The DNA fragments are produced by DNA sequencing machines, and are called "reads". These are in lengths of about 150 nucleotides (base pairs), to up to a million+ nucleotides, depending on the sequencing technology used. Currently, most reads are from Illumina (short), PacBio (long) or Oxford Nanopore (long and extra-long).

It is difficult to assemble plant genomes as they are often large (for example, 3,000,000,000 base pairs), have many repeat regions (such as transposons), and may be polyploid. This tutorial shows genome assembly for a smaller data set - the plant chloroplast genome - a single circular chromosome which is typically about 160,000 base pairs. It is thought that the the chloroplast evolved from a cyanobacteria that was living in plant cells.

In this tutorial, we will use a subset of a real data set from sweet potato, from the paper {% cite Zhou2018 %}. To find out more about each of the tools used here, see the tool panel page for a summary and links to more information.

>### Agenda
> In this tutorial we will deal with:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Upload data

Let's start with uploading the data.

> ### {% icon hands_on %} Hands-on: Import the data
> 1. Create a new history for this tutorial and give it a proper name
>
>    {% include snippets/create_new_history.md %}
>    {% include snippets/rename_history.md %}
>
> 2. Import from [Zenodo](https://zenodo.org/record/3567224) or a data library (ask your instructor):
>   - FASTQ file with illumina reads: `sweet-potato-chloroplast-illumina-reduced.fastq`
>   - FASTQ file with nanopore reads: `sweet-potato-chloroplast-nanopore-reduced.fastq`
>   - Note: make sure to import the files with "reduced" in the names, not the ones with "tiny" in the names.
>    ```
>    https://zenodo.org/record/3567224/files/sweet-potato-chloroplast-illumina-reduced.fastq
>    https://zenodo.org/record/3567224/files/sweet-potato-chloroplast-nanopore-reduced.fastq    
>    ```
>
>    {% include snippets/import_via_link.md %}
>    {% include snippets/import_from_data_library.md %}
{: .hands_on}

# Check read quality

We will look at the quality of the nanopore reads.

> ### {% icon hands_on %} Hands-on: Check read quality
>
> 1. **Nanoplot** {% icon tool %}:
>    - *"Select multifile mode"*: `batch`
>    - *"Type of file to work on"*: `fastq`
>    - *"files"*: select the `nanopore FASTQ file`
{: .hands_on}

There are five output files. Look at the `HTML report` to learn about the read quality.

> ### {% icon question %} Question
> What summary statistics would be useful to look at?
>
> > ### {% icon solution %} Solution
> > This will depend on the aim of your analysis, but usually:
> > * **Sequencing depth** (the number of reads covering each base position; also called "coverage"). Higher depth is usually better, but at very high depths it may be better to subsample the reads, as errors can swamp the assembly graph.
> > * **Sequencing quality** (the quality score indicates probability of base call being correct). You may trim or filter reads on quality. Phred quality scores are logarithmic: phred quality 10 = 90% chance of base call being correct; phred quality 20 = 99% chance of base call being correct. More detail [here](https://en.wikipedia.org/wiki/Phred_quality_score).
> > * **Read lengths** (read lengths histogram, and reads lengths vs. quality plots). Your analysis or assembly may need reads of a certain length.
> {: .solution}
{: .question}

*Optional further steps:*
* Find out the quality of your reads using other tools such as fastp or FastQC.
* To visualize base quality using emoji you can also use FASTQE.
* Run FASTQE for the illumina reads. In the output, look at the mean values (the middle row)
* Repeat FASTQE for the nanopore reads. In the tool settings, increase the maximum read length to 30000.
* To learn more, see the [Quality Control tutorial]({{site.baseurl}}/topics/sequence-analysis/tutorials/quality-control/tutorial.html)

# Assemble reads

We will assemble the long nanopore reads.

> ### {% icon hands_on %} Hands-on: Assemble reads
> 1. **Flye** {% icon tool %}:
>    - *"Input reads"*: `sweet-potato-chloroplast-nanopore-reduced.fastq`
>    - *"Estimated genome size"*: `160000`
>    - *Leave other settings as default*
{: .hands_on}

There are five output files.
* *Note: this tool is heuristic; your results may differ slightly from the results here, and if repeated.*
* View the `log` file and scroll to the end.
* How many contigs (fragments) were assembled?
* What is the length of the assembly?
* View the `assembly_info` file.
* What are the contig names and lengths?
* The assembly sequence is in the `consensus`. Re-name this `flye-assembly.fasta`

> ### {% icon hands_on %} Hands-on: View the assembly
> 1. **Bandage Info** {% icon tool %}:
>    - *"Graphical Fragment Assembly"*: the Flye output file `Graphical Fragment Assembly` (not the "assembly_graph" file)
>    - *"Estimated genome size"*: `160000`
>    - *Leave other settings as default*
> 2. **Bandage Image** {% icon tool %}:
>    - *"Graphical Fragment Assembly"*: the Flye output file `Graphical Fragment Assembly` (not the "assembly_graph" file)
>    - *"Node length labels"*: `Yes`
>    - *Leave other settings as default*
{: .hands_on}

Your assembly graph may look like this:

![assembly graph](../../images/sweet-potato-assembly-graph.png "Assembly graph of the nanopore reads, for the sweet potato chloroplast genome. In your image, some of the labels may be truncated; this is a known bug under investigation.")

> ### {% icon question %} Question
> What is your interpretation of this assembly graph?
> > ### {% icon solution %} Solution
> > One interpretation is that this represents the typical circular chloroplast structure: There is a long single-copy region (the node of around 78,000 bp), connected to the inverted repeat (a node of around 28,000 bp), connected to the short single-copy region (of around 11,000 bp). In the graph, each end loop is a single-copy region (either long or short) and the centre bar is the collapsed inverted repeat which should have about twice the sequencing depth.
> {: .solution}
{: .question}

*Optional further steps:*
* Repeat the Flye assembly with different parameters, and/or a filtered read set.
* Try an alternative assembly tool, such as Canu or Unicycler.

# Polish assembly

Short illumina reads are more accurate the nanopore reads. We will use them to correct errors in the nanopore assembly.

> ### {% icon hands_on %} Hands-on: Map reads
> 1. **Map with BWA-MEM** {% icon tool %}:
>    - *"Will you select a reference genome from your history"*: `Use a genome from history`
>    - *"Use the following dataset as the reference sequence"*: `flye-assembly.fasta`
>    - *"Algorithm for constructing the BWT index"*: `Auto. Let BWA decide`
>    - *"Single or Paired-end reads"*: `Single`
>    - *"Select fastq dataset"*: `sweet-potato-illumina-reduced.fastq`
>    - *"Set read groups information?"*: `Do not set`
>    - *"Select analysis mode"*: `Simple Illumina mode`
{: .hands_on}

* This maps the short reads to the assembly, and creates an alignment file.
* Re-name this file `illumina.bam`

> ### {% icon hands_on %} Hands-on: Polish
> 1. **pilon** {% icon tool %}:
>    - *"Source for reference genome used for BAM alignments"*: `Use a genome from history`
>    - *"Select a reference genome"*: `flye-assembly.fasta`
>    - *"Type automatically determined by pilon"*: `Yes`
>    - *"Input BAM file"*: `illumina.bam`
>    - *"Variant calling mode"*: `No`
>    - *"Create changes file"*: `Yes`
{: .hands_on}

* This compares the short reads to the assembly, and creates a polished (corrected) assembly file.
* There are two outputs: a `fasta` file and a `changes` file.
* What is in the `changes` file?
* Re-name the fasta output file `polished-assembly.fasta`
* Find and run the tool called "Fasta statistics" on the original flye assembly and the polished version.

> ### {% icon question %} Question
> How does the polished assembly compare to the unpolished assembly?
> > ### {% icon solution %} Solution
> > This will depend on the settings, but as an example: your polished assembly might be about 10-15 Kbp longer. Nanopore reads can have homopolymer deletions - a run of AAAA may be interpreted as AAA - so the more accurate illumina reads may correct these parts of the long-read assembly. In the changes file, there may be a lot of cases showing a supposed deletion (represented by a dot) being corrected to a base.
> {: .solution}
{: .question}

*Optional further steps:*
* Run a second round (or more) of Pilon polishing. Keep track of file naming; you will need to generate a new bam file first before each round of Pilon.
* Run an alternative polishing tool, such as Racon. This uses the long reads themselves to correct the long-read (Flye) assembly. It would be better to run this tool on the Flye assembly before running Pilon, rather than after Pilon.

# View reads

We will look at the original sequencing reads mapped to the genome assembly. In this tutorial, we will import very cut-down read sets so that they are easier to view.

> ### {% icon hands_on %} Hands-on: Import cut-down read sets
> 1. Import from [Zenodo](https://zenodo.org/record/3567224) or a data library (ask your instructor):
>   - FASTQ file with illumina reads: `sweet-potato-chloroplast-illumina-tiny.fastq`
>   - FASTQ file with nanopore reads: `sweet-potato-chloroplast-nanopore-tiny.fastq`
>   - Note: these are the "tiny" files, not the "reduced" files we imported earlier.
>    ```
>    https://zenodo.org/record/3567224/files/sweet-potato-chloroplast-illumina-tiny.fastq
>    https://zenodo.org/record/3567224/files/sweet-potato-chloroplast-nanopore-tiny.fastq
>    ```
{: .hands_on}

> ### {% icon hands_on %} Hands-on: Map the reads to the assembly
> * Map the Illumina reads (the new "tiny" dataset) to the `polished-assembly.fasta`, the same way we did before, using bwa mem.
> * This creates one output file: re-name it `illumina-tiny.bam`
> * Map the Nanopore reads (the new "tiny" dataset) to the `polished-assembly.fasta`. The settings will be the same, except `Select analysis mode` should be `Nanopore`
> * This creates one output file: re-name it `nanopore-tiny.bam`
{: .hands_on}

> ### {% icon hands_on %} Hands-on: Visualise mapped reads
> 1. **JBrowse genome browser** {% icon tool %}:
>    - *"Reference genome to display"*: `Use a genome from history`
>        - *"Select a reference genome"*: `polished-assembly.fasta`
>    - *"Produce Standalone Instance"*: `Yes`
>    - *"Genetic Code"*: `11. The Bacterial, Archaeal and Plant Plastid Code`
>    - *"JBrowse-in-Galaxy Action"*: `New JBrowse instance`
>    - *"Insert Track Group"*
>        - *"Insert Annotation Track"*
>            - *"Track Type"*: `BAM pileups`
>            - *"BAM track data"*: `nanopore-tiny.bam`
>            - *"Autogenerate SNP track"*: `No`
>            - *Leave the other track features as default*
>        - *"Insert Annotation Track"*.
>            - *"Track Type"*: `BAM pileups`
>            - *"BAM track data"*: `illumina-tiny.bam`
>            - *"Autogenerate SNP track"*: `No`
>            - *Leave the other track features as default*
{: .hands_on}

There is one output file: re-name: `assembly-and-reads`
* Click on the eye icon to view. (For more room, collapse Galaxy side menus with corner < > signs).
*  Make sure the bam files are ticked in the left hand panel.
* Choose a contig in the drop down menu. Zoom in and out with + and - buttons.

Here is an embedded snippet showing JBrowse and the mapped reads:

{% include snippets/jbrowse.html datadir="data" %}


> ### {% icon question %} Questions
>
> 1. What are the differences between the nanopore and the illumina reads?
> 2. What are some reasons that the read coverage may vary across the reference genome?
>
> > ### {% icon solution %} Solutions
> > 1. Nanopore reads are longer and have a higher error rate.  
> > 2. There may be lots of reasons for varying read coverage. Some possibilities: In areas of high read coverage: this region may be a collapsed repeat. In areas of low or no coverage: this region may be difficult to sequence; or, this region may be a misassembly.
> {: .solution}
{: .question}

# Annotate the assembly

We can now annotate our assembled genome with information about genomic features.
* A chloroplast genome annotation tool is not yet available in Galaxy, although for an approximation we could use the tool for bacterial genome annotation, Prokka.
* However here we will use a web-based tool designed for chloroplast genomes to annotate our assembly (or, you can upload an example annotation).

Follow these steps to annotate your assembly, or upload an example annotation file from this [Zenodo link](http://doi.org/10.5281/zenodo.4267171) to your current Galaxy history, and skip to the next step.

> ### {% icon hands_on %} Hands-on: Annotate with GeSeq
> * Download `polished.fasta` to your computer (click on the file in your history; then click on the disk icon).
> * In a new broswer tab, go to [Chlorobox](https://chlorobox.mpimp-golm.mpg.de/geseq.html) where we will use the [GeSeq tool](https://academic.oup.com/nar/article/45/W1/W6/3806659) to annotate our sequence.
> * Upload the `fasta` file there. Information about how to use the tool is available on the page.
> * Once the annotation is completed, download the required files.
> * In Galaxy, import the annotation `GFF3` file.
{: .hands_on}

Make a JBrowse file to view the annotations (the GFF3 file) under the assembly (the polished.fasta file).

> ### {% icon hands_on %} Hands-on: View annotations
> 1. **JBrowse genome browser** {% icon tool %}:
>        - *"Reference genome to display"*: `Use a genome from history`
>    - *"Select a reference genome"*: `polished-assembly.fasta`
>    - *"Produce Standalone Instance"*: `Yes`
>    - *"Genetic Code"*: `11. The Bacterial, Archaeal and Plant Plastid Code`
>    - *"JBrowse-in-Galaxy Action"*: `New JBrowse instance`
>    - *"Insert Track Group"*
>        - *"Insert Annotation Track"*
>            - *"Track Type"*: `GFF/GFF3/BED/GBK Features`
>            - *"GFF/GFF3/BED Track Data"*: the `GFF3` file
>            - *Leave the other track features as default*
{: .hands_on}

This may take a few minutes. There is one output file: re-name it `view-annotations`
* Click on the eye icon to view.
* Select the right contig to view, in the drop down box.
* Zoom out (with the minus button) until annotations are visible.

Here is an embedded snippet showing JBrowse and the annotations:

{% include snippets/jbrowse.html datadir="data2" %}


Your annotations may look like this:

![annotations](../../images/annotations1.png "Annotations - zoomed out view")

Zoom in (with the plus button) to see annotation details. Click on an annotation to see its sequence and source (e.g. the tool that predicted it).

![annotations](../../images/annotations2.png "Annotations - zoomed in view")

> ### {% icon question %} Question
> Why might there be several annotations over the same genome region?
> > ### {% icon solution %} Solution
> > One reason is that these are predictions from different tools - such as BLAT or HMMER.
> {: .solution}
{: .question}

# Repeat with new data

*Optional extension exercise*

We can assemble another chloroplast genome using sequence data from a different plant species: the snow gum, *Eucalyptus pauciflora*. This data is from {% cite Wang2018 %}. It is a subset of the original FASTQ read files (Illumina - SRR7153063, Nanopore - SRR7153095).

> ### {% icon hands_on %} Hands-on: Assembly and annotation
> * Get data: at this [Zenodo link](https://doi.org/10.5281/zenodo.3600662), then upload to Galaxy.
> * Check reads: Run Nanoplot on the nanopore reads.
> * Assemble: Use Flye to assemble the nanopore reads, then get Fasta statistics *Note: this may take several hours.*
> * Polish assembly: Use Pilon to polish the assembly with short Illumina reads. *Note: Don't forget to map these Illumina reads to the assembly first using bwa-mem, then use the resulting `bam` file as input to Pilon.*
> * Annotate: Use the GeSeq tool at [Chlorobox](https://chlorobox.mpimp-golm.mpg.de/geseq.html).
> * View annotations:Use <ss>JBrowse to view the assembled, annotated genome.
{: .hands_on}

# Conclusion
{:.no_toc}