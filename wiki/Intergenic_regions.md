---
title: Intergenic regions
permalink: wiki/Intergenic_regions
layout: wiki
---

Problem
-------

Extract intergenic regions from a sequence. Sometimes, reading between
the genes is more interesting....

Solution
--------

The following ready-to-run script reads a genbank file, which is
probably a genomic or chromosomal one. It uses the CDS feature to
discover the 5' and 3' ends of ORFs. Yes, ORFs are not exactly
synonymous with genes, but this is the way we did it. Also, you may want
to swap the CDS feature for the 'gene' feature, if you are also
interested in RNA coding genes. The "intergene\_length" variable is a
threshold on the minimal length of intergenic regions to be analyzed,
and is set by default to 100. The program outputs to a file with the
suffix ".ign" The program outputs the + strand or the reverse-complement
based on the genbank file annotation. The output is in FASTA format, and
the header includes the intergenic region coordinates, and unique ID,
and whether the sequence was derived from the + or - strand.

``` python
#!/usr/bin/env python
import sys
import Bio
from Bio import SeqIO, SeqFeature
from Bio.SeqRecord import SeqRecord
import os
# Copyright(C) 2009 Iddo Friedberg & Ian MC Fleming
# Released under Biopython license. http://www.biopython.org/DIST/LICENSE
# Do not remove this comment
def get_interregions(genbank_path,intergene_length=100):
    seq_record = SeqIO.parse(open(genbank_path), "genbank").next()
    
    cds_list = []
    intergenic_records = []
    # Loop over the genome file, get the 
    for fnum, feature in enumerate(seq_record.features):
        if feature.type == 'CDS':  
            mystart = feature.location._start.position
            myend = feature.location._end.position
            cds_list.append((mystart,myend,feature.strand))
    for i,pospair in enumerate(cds_list[1:]):
        # Compare current start position to previous end position
        last_end = cds_list[i][1]
        this_start = pospair[0]
        strand = pospair[2]
        if this_start - last_end >= intergene_length:
            intergene_seq = seq_record.seq[last_end:this_start]
            if strand == -1:
                intergene_seq = intergene_seq.reverse_complement()
                strand_string = "-"
            else:
                strand_string = "+"
            intergenic_records.append( 
                  SeqRecord(intergene_seq,id="%s-ign-%d" % (seq_record.name,i),
                  description="%s %d-%d %s" % (seq_record.name, last_end+1,
                                                        this_start,strand_string)))
    outpath = os.path.splitext(os.path.basename(genbank_path))[0] + ".ign"
    SeqIO.write(intergenic_records, open(outpath,"w"), "fasta")

          
if __name__ == '__main__':
    if len(sys.argv) == 2:
         get_interregions(sys.argv[1])
    elif len(sys.argv) == 3:
         get_interregions(sys.argv[1],int(sys.argv[2]))
    else:
         print "Usage: get_intergenic.py gb_file [intergenic_length]"
         sys.exit(0)

 
```

Running
-------

<bash> ./get\_intergene mygenbankfile.gb 1 </bash>

Run on the genbank file. The intergenic sequences will appear in the
file mygenbank.ign in FASTA format.

[`category:Cookbook`](category:Cookbook "wikilink")