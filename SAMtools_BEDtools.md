---
title: "Sequences and Genomic Features"
author: "Quynh Nhu Nguyen"
date: "May 4th 2023"
---
# **SEQUENCES AND GENOMIC FEATURES: SAM tools and BED tools**

## 1. SAMtools: 
Next-generation sequencing alignments are often represented in SAM format. Another compressed format that is available to us is called BAM.

### Look at the number of alignments  
```bash
$ samtools flagstat athal_wu_0_A.bam
```

### SAMtool sort
```bash
$ nohup samtools sort athal_wu_0_A.bam -o athal_wu_0_A.sorted.bam &
```

### SAMtool index
```bash
$ samtools index -M [-bc] [-m INT] <in1.bam> <in2.bam>
$ samtools index -M athal_wu_0_A.sorted.bam
```

### SAMtool merge
```bash
$ samtools merge [options] -o <out.bam> [options] <in1.bam> ... <inN.bam>
```

### SAMtool view
```bash
# convert from BAM to SAM without header information
$ samtools view athal_wu_0_A.bam | more # 

# with header information
$ samtools view -h in.bam
$ samtools view -h athal_wu_0_A.bam | more 

# save BAM file to SAM file
$ samtools view -h athal_wu_0_A.bam > athal_wu_0_A.sam # save this input file *.sam 

# see the header of the file
$ samtools view -H athal_wu_0_A.bam

# convert from SAM to BAM?
$ samtools view -bT ref.fa in.sam.gz
$ samtools view -bT ~/Downloads/bioinformatics_softwares/dnaseq_work/work/ref_genome/hs38DH.fa athal_wu_0_A.sam > athal_wu_0_A.sam.bam

# extract the alignments within a range
## The file has to be sorted and indexed. The word "coordinate" in the header information of bam file denotes sorting of the file. Next, applies the index. And this creating the file example.bam.bai. Now, we can proceed with our range extraction, samtools view example.bam and then we are giving it the range.
$ samtools index -M athal_wu_0_A.sorted.bam
$ ls -l athal_wu_0_A.sorted.bam.bai

$ samtools view athal_wu_0_A.sorted.bam "Chr3:11777000-11794000"

## check if the alignment is within the required range
$ samtools view athal_wu_0_A.sorted.bam "Chr3:11777000-11794000" | head 

$ samtools view athal_wu_0_A.sorted.bam "Chr3:11777000-11794000" | tail

## extract the alignments within a input regions in a BED-format file
$ samtools view -L athal_wu_0_A_annot.bed athal_wu_0_A.sorted.bam
```

### Practical exercises:
How many sequences are in the genome file?
```bash
$ samtools view -H athal_wu_0_A.bam | grep "@SQ" | wc -l
```

How many alignments does the set contain? 
```bash
$ samtools flagstat athal_wu_0_A.bam
# Or
$ samtools view athal_wu_0_A.bam | wc -l
```

How many alignments show the read’s mate unmapped?
```bash
$ samtools view athal_wu_0_A.bam | cut -f7 | grep "*" | wc -l
```

How many alignments contain a deletion (D)? 
```bash
$ samtools view athal_wu_0_A.bam | cut -f6 | grep "D" | wc -l
```

How many alignments show the read’s mate mapped to the same chromosome?
```bash
$ samtools view athal_wu_0_A.bam | cut -f7 | grep "=" | wc -l
```

How many alignments are spliced? 
```bash
# calculate skipped region (intron)
$ samtools view athal_wu_0_A.bam | cut -f6 | grep "N" | wc -l
```

## 2. BEDtools: 
BEDtools is a versatile suite that includes tools for general arithmetic, format conversion, and sequence manipulation, such as extracting sequences inside a specific range or interval.

In the `gtf` format, every line was a separate feature, separate exon.

In the `BED` format, one gene is represented by one line with multiple intervals.

### Extract all the exons or all the genes, or all the transcripts that contain "Alus or in other words it overlap Alus.
```bash
# -wo: reported only the lines that contain the overlaps and the extent of the overlap. 
$ bedtools intersect -wo -a RefSeq.gtf -b Alus.bed | more
```

### How many gene features are there?
```bash
$ bedtools intersect -wo -a RefSeq.gtf -b Alus.bed | wc -l
```

### How many genes do they have correspond to?
```bash
$ bedtools intersect -wo -a RefSeq.gtf -b Alus.bed | cut -f9 | cut -d ' ' -f2 | sort -u | wc -l
```

### Split command line: each exon is going to be considered as a separate interval.
```bash
$ bedtools intersect -split -wo -a RefSeq.bed -b Alus.bed | wc -l
```

### In this case, if you're looking at the last, in this case, we have all the features in A represented whether they do have an overlap or not. And you will see that why the first line shows an overlap feature. The second line here will show you the gene, followed by a no entry in the alu file, which is represented by a ., -1, -1, and so on. So that's a way of representing no feature
```bash
$ bedtools intersect -split -wao -a RefSeq.bed -b Alus.bed | wc -l
```

### BAMtoBED: Converts feature records to BED format.
```bash
$ bedtools bamtobed -cigar -i athal_wu_0_A.bam > athal_wu_0_A.bed
```

### BAMtoBED with the split option will split the alignments, spliced alignments into a number of blocks that correspond to the number of exons that are contained in that aren't covered by that align.
```bash
$ bedtools bamtobed -split -i athal_wu_0_A.bam | grep GAII02:4:13:207:1907#0/1
```

### BEDtoBAM: Converts feature records to BAM format.
```bash
# The output contains one line for every exon, represented on the line. 
$ bedtools bedtobam -i RefSeq.gtf -g hg38c.hdrs > refseq.bam

# The output contains one line now, one alignment, for each gene. But notice the length. The cigar string corresponding to the first alignment, 59,129. This is because it contains the inference. So it is shown as an alignment that starts at the beginning of the gene and ending at this very last nucleotide. But containing all the intervening inferring positions. So if we want to show a more realistic picture of the alignment that shows introns that separate exons, then we need to specify what we want, that the input.
$ bedtools bedtobam -i RefSeq.bed -g hg38c.hdrs > refseq.bam

# If we want to show a more realistic picture of the alignment that shows introns that separate exons, then we need to specify what we want, that the input.
$ bedtools bedtobam -bed12 -i RefSeq.bed -g hg38c.hdrs > refseq.bam
```
### getfasta : Extract DNA sequences from a fasta file based on feature coordinates.
```bash
$ bedtools getfasta -fi ~/Downloads/bioinformatics_softwares/dnaseq_work/work/ref_genome/hs38DH.fa -bed gencode.GRCh38.p13.v43.chr_patch_hapl_scaff.annotation.gtf -fo ref.GRCh38.gtf.fa

# This output corresponds to the genomic sequence containing introns and exons and introns at that particular genes block.
$ bedtools getfasta -fi ~/Downloads/bioinformatics_softwares/dnaseq_work/work/ref_genome/hs38DH.fa -bed gencode.GRCh38.p13.v43.chr_patch_hapl_scaff.annotation.bed -fo ref.GRCh38.bed.fa

# This output corresponds to exonic blocks.
$ bedtools getfasta -split -fi ~/Downloads/bioinformatics_softwares/dnaseq_work/work/ref_genome/hs38DH.fa -bed gencode.GRCh38.p13.v43.chr_patch_hapl_scaff.annotation.bed -fo ref.GRCh38.exonic.bed.fa
```

### Practical exercises:
How many overlaps (each overlap is reported on one line) are reported?
```bash
$ bedtools bamtobed -i athal_wu_0_A.bam > athal_wu_0_A.bed

$ bedtools intersect -wo -a athal_wu_0_A_annot.gtf -b athal_wu_0_A.bed | wc -l
```

How many of these are 10 bases or longer?
```bash
$ bedtools intersect -wo -a athal_wu_0_A_annot.gtf -b athal_wu_0_A.bed | cut -f16 > count.txt

$ awk '$1>9{c++} END{print c+0}' count.txt
```
This is an `awk` command that reads the file "count.txt" and counts the number of lines where the first field (the first column of the line) is greater than 9. The `awk` command increments a variable `c` for each line that meets this criterion. At the end of reading the file, the `END `block is executed, which simply prints the value of `c`. The `+0` at the end is used to convert the value of `c` to a numeric value, in case `c` is empty, which would otherwise result in a "0" string output. Overall, this command is counting the number of lines where the first field is greater than 9 and printing that count

How many alignments overlap the annotations?
```bash
$ bedtools intersect -wo -a athal_wu_0_A_annot.gtf -b athal_wu_0_A.bed | wc -l
```

Conversely, how many exons have reads mapped to them?
```bash
$ bedtools intersect -wo -a athal_wu_0_A_annot.gtf -b athal_wu_0_A.bed | cut -f4 | sort -u | wc -l
```

If you were to convert the transcript annotations in the file “athal_wu_0_A_annot.gtf” into BED format, how many BED records would be generated?
```bash
$ bedtools intersect -wo -a athal_wu_0_A_annot.gtf -b athal_wu_0_A.bed | cut -f9 | cut -d " " -f4 | sort -u | wc -l
```

