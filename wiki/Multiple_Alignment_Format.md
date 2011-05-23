---
title: Multiple Alignment Format
permalink: wiki/Multiple_Alignment_Format
layout: wiki
tags:
 - Formats|Formats
---

MafIndex
--------

Biopython provides an interface for fast access to the multiple
alignment of several sequences across an arbitrary interval: for
example, chr10:25,079,604-25,243,324 in mm9. As MAF files are available
for entire chromosomes, they can be indexed by chromosome position and
accessed at random. This functionality is available in the class
MafIO.MafIndex.

### Creating or loading a MAF index

Indexes are created by determining the chromosome start and end position
for a specific sequence name (generally a species), which must appear in
every alignment block in the file. In whole-genome alignments generated
by Multiz, the chromosome of one species is generally used as the
reference to which other species are aligned. This reference species
will appear in every block, and should be used as the *target\_seqname*
parameter. For UCSC multiz files, the form of **species.chromosome** is
used.

To index a MAF file, or load an existing index, create a new MafIndex
object. If the index database file *sqlite\_file* does not exist it will
be created, otherwise it will be loaded.

    from Bio.AlignIO import MafIO.MafIndex

    idx = MafIO.MafIndex(sqlite_file, maf_file, target_seqname)

### Retrieving alignments overlapping a given interval

The *search(starts, ends)* generator function accepts a list of start
and end positions, and yields MultipleSeqAlignment objects that overlap
the given intervals. This is particularly useful for obtaining
alignments over the multiple exons of a single transcript, eliminating
the need to retrieve an entire locus.

    results = idx.search([25079604], [25243324])

    for multiple_alignment in results:
        print multiple_alignment

### Retrieving a pre-spliced alignment over a given set of exons

The *get\_spliced(starts, ends, strand = "+")* function accepts a list
of start and end positions representing exons, and returns a single
MultipleSeqAlignment object of the *in silico* spliced transcript from
the reference and all aligned sequences. If part of the sequence range
is not found in a particular species in the alignment, dashes ("-") are
used to fill the gaps, or "N"s if the sequence is not present in the
reference (*target\_seqname*) sequence. If *strand* is opposite that in
the reference sequence, all sequences in the returned alignment will be
reverse complemented.

    # convert the alignment for mouse Foxo3 (NM_019740) from MAF to FASTA

    from Bio.AlignIO import MafIO.MafIndex
    from Bio import SeqIO
    import sys

    idx = MafIO.MafIndex("chr10.mafindex", "chr10.maf", "mm9.chr10")

    multiple_alignment = idx.get_spliced([41905591, 41916271, 41994621, 41996331],
                                         [41906101, 41917707, 41995347, 41996548],
                                         strand = "+")

    for seqrec in multiple_alignment:
        SeqIO.write(seqrec, sys.stdout, "fasta")

### Example

chr10:7,350,034-7,383,048) In this example, we are going to download the
30-way alignment of various species to mouse chromosome 10, generate an
index file, and load the multiple alignment across the Pcmt1 locus.

The MAF file is available at:
<http://hgdownload.cse.ucsc.edu/goldenPath/currentGenomes/Mus_musculus/multiz30way/maf/chr10.maf.gz>

<code> from Bio.AlignIO import MafIO.MafIndex as MafIndex

idx = MafIndex("chr10.mafindex", "chr10.maf", "mm9.chr10")

idx