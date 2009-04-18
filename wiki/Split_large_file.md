---
title: Split large file
permalink: wiki/Split_large_file
layout: wiki
tags:
 - Cookbook
---

Problem
-------

With modern sequencing technologies it has become relatively cheap easy
to generate very large datasets. In fact, there are times when one can
have too much data in one file, online resources like
[PLAN](http://bioinfo.noble.org/plan) limit the size of users queries.
In such cases it useful to be able to split a sequence file into a set
of smaller files, each containing a subset of original file's sequences.

Solution
--------

``` python
def batch_iterator(iterator, batch_size) :
    """Returns lists of length batch_size.
 
    This can be used on any iterator, for example to batch up
    SeqRecord objects from Bio.SeqIO.parse(...), or to batch
    Alignment objects from Bio.AlignIO.parse(...), or simply
    lines from a file handle.
 
    This is a generator function, and it returns lists of the
    entries from the supplied iterator.  Each list will have
    batch_size entries, although the final list may be shorter.
    """
    entry = True #Make sure we loop once
    while entry :
        batch = []
        while len(batch) < batch_size :
            try :
                entry = iterator.next()
            except StopIteration :
                entry = None
            if entry is None :
                #End of file
                break
            batch.append(iterator.next())
        yield batch
 
from Bio import SeqIO
record_iter = SeqIO.parse(open("SRR014849.fastq"),"fastq")
for i, batch in enumerate(batch_iterator(record_iter, 10000)) :
    filename = "group_%i.fastq" % (i+1)
    handle = open(filename, "w")
    count = SeqIO.write(batch, handle, "fastq")
    handle.close()
    print "Wrote %i records to %s" % (count, filename)
```

And the output using SRR014849.fastq from this [compressed
file](ftp://ftp.ncbi.nlm.nih.gov/sra/static/SRX003/SRX003639/SRR014849.fastq.gz)
at the NCBI.

`Wrote 10000 records to group_1.fastq`  
`Wrote 10000 records to group_2.fastq`  
`Wrote 10000 records to group_3.fastq`  
`Wrote 10000 records to group_4.fastq`  
`Wrote 7348 records to group_5.fastq`

How it works
------------

It is possible to use list([SeqIO](SeqIO "wikilink").parse(...)) to read
the entire contents of a file into memory then write slices of the list
out as smaller files. For large files (like the ones this recipe is
about) that would take up a big hunk of memory, instead we can define a
generator function, batch\_iterator(), that loads one record at a time
then appends it to a list, repeating the process until the list
containins one file's worth of sequences.

With that function defined it's a matter of giving it an iterator (in
this case a SeqIO.parse(...) instance and writing out the batching of
records it produces.