---
title: "Basic Unix Commands"
author: "Quynh Nhu Nguyen"
date: "May 2nd 2023"
---
# **BASIC UNIX COMMANDS: PRACTICAL EXERCISES**

## How many chromosomes are there in the genome?
```bash
$ grep ">" apple.genome | wc -l

# Or
$ grep -c ">" apple.genome
```

## How many genes and transcript variants?
```bash
# Gene names is listed in 1st column, Transcript variants is listed in 2nd column 
$ cut -f1 apple.genes | uniq | wc -l
$ cut -f2 apple.genes | more | wc -l #Eevery variant was independently, because indeed the file contains only one line for each variant of a gene.

# Or
$ cut -f1 apple-genes | sort -u | wc -l
$ cut -f2 apple-genes | sort -u | wc -l
```
### How many genes have a single variants?
```bash
$ cut -f1 apple.genes | uniq -c | grep -c " 1 "
```

### How many genes have multiple variants?
```bash
$ cut -f1 apple.genes | uniq -c | grep -v " 1 " | wc -l
```

## How many genes are there on the ‘+’ strand?
```bash
$ cut -f4 apple.genes | grep "+" | wc -l
```

## How many genes are there on the ‘-’ strand?
```bash
$ cut -f4 apple.genes | grep "-" | wc -l
```

## How many genes are there on chromosome chr1, chr2, ch3?
```bash
$ cut -f1,3 apple.genes | grep "chr1" | sort -u | wc -l

$ cut -f1,3 apple.genes | grep "chr2" | sort -u | wc -l

$ cut -f1,3 apple.genes | grep "chr3" | sort -u | wc -l
```

## How many transcripts are there on chr1, chr2, chr3?
```bash
$ cut -f2,3 apple.genes | grep -c "chr1"

$ cut -f2,3 apple.genes | grep -c "chr2"

$ cut -f2,3 apple.genes | grep -c "chr3"
```

## What plant systems contain a Smell gene?
```bash
$ grep Smell */*.genes # give us all the lines that contain somewhere within the line, the word smell.
```

## What genes are in common between toy apple and toy pear?
```bash
$ cut -f1 apple/apple.genes | sort -u > applegenes

$ cut -f1 pear/pear.genes | sort -u > peargenes

$ comm -1 -2 applegenes peargenes | wc -l

# Or
$ cat applegenes peargenes | sort | uniq -c | grep -v " 1 " | wc -l
```

### What genes are specific to each?
```bash
# Genes are specific to the apple species
$ comm -2 -3 applegenes peargenes 

# Genes are specific to the pear species
$ comm -1 -3 applegenes peargenes 
```
