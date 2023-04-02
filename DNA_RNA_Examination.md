---
title: "DNA Sequencing Examination"
author: "Quynh Nhu Nguyen"
date: "March 26th 2023"
---
# **DNA/RNA SEQUENCING EXAMINATION**
## QUESTION 1
Download the sequences in the ACN list "cytochrom_plasmid.txt"
```bash
# create a directory you work in
$ mkdir NGS_Examination_26_03

# change into the directory
$ cd NGS_Examination_26_03

# install NCBI
$ sudo apt install ncbi-entrez-direct
$ sudo apt install acedb-other
```

```bash
#!/bin/bash
file="cytochrom_plasmid.txt"
num_files=$(wc -l < $file) # Number of files to download

# Loop until we have downloaded enough files
while [ $(ls -l | grep -c "^-" ) -lt $num_files ]
do
    # Read the filename from the txt file entries.
    while read -r filename
    do
        echo "${filename}" | esearch -db nucleotide -query ${filename} | efetch -format fasta > "${filename}.fasta"
        # Check if the download was successful
        if [ $? -eq 0 ]
        then
            echo "Downloaded $filename"
        else
            echo "Failed to download $filename. Retrying..."
            # Try downloading the file again
            echo "${filename}" | esearch -db nucleotide -query ${filename} | efetch -format fasta > "${filename}.fasta"
            if [ $? -eq 0 ]
            then
                echo "Downloaded $filename on retry"
            else
                echo "Failed to download $filename on retry"
            fi
        fi
    done < "$file"
done

echo "Downloaded $(ls | grep -c '\.fasta$') files"
```

## QUESTION 2
Merge them into a big fasta format file
```bash
# concatenate all *.fasta in the current directory into a big fasta format file
$ cat *.fasta > big.fasta

# test information of big.fasta
$ head -20 big.fasta
$ tail -20 big.fasta

# check if the number of downloaded files is equal to the required FASTAs'number in cytochrom_plasmid.txt
$ if [ $(grep -c "^>" big.fasta) -eq $num_files ]; then
    echo "True"
else
    echo "False"
fi
```
## QUESTION 3
Remove the first 12 bases of all the sequences in the fasta and write it to the new file new_5_12.fasta 
```bash
$ sed 's/^[A-Z]\{12\}//' big.fasta > new_5_12.fasta
```
## QUESTION 4
Remove the last 12 bases of all the sequences in the fasta and write it to the new file new_3_12.fasta 
```bash
$ sed 's/^[A-Z]\{12\}$//' big.fasta > new_3_12.fasta
```
## QUESTION 5
Remove the middle 12 bases of all the sequences in the fasta and write it to the new file new_mid_12.fasta 
```bash
$ awk '/^>/ {print;next} {L=length($0);print substr($0,1,int((L-12)/2)) substr($0,int((L+12)/2)+1)}' big.fasta > new_mid_12.fasta 
```
