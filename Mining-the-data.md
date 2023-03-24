# **DNA/RNA SEQUENCING**

## QUESTION 1
Downloads the genbank format of gp120 mRNA
https://www.ebi.ac.uk/ena/browser/api/embl/**Y**?download=true. Uses Accession List: U47807.1.
```bash
$ wget -O "U47807.1.gb" https://www.ebi.ac.uk/ena/browser/api/embl/U47807.1?download=true
 ```
## QUESTION 2
Downloads all genbanks using the Accession List of gp120 mRNA.
```bash
$ file="Accession_List_of_gp120.txt"
$ head $file

## Method 1
$ while read -r line; do echo $line; wget -O "${line}.gb" https://www.ebi.ac.uk/ena/browser/api/embl/${line}?download=true; 
done < "$file"

## Method 2
$ for i in $(cat $file); do echo $i; wget -O "${i}.gb" https://www.ebi.ac.uk/ena/browser/api/embl/${i}?download=true; 
done

## Test information of donwloaded file
$ ls U477*
$ head U477*
$ tail -n3 U477*
```
## QUESTION 3
Makes a table “tb.tsv” with the columns for all records: Accession number; Organism; Product.

### Method 1

Concatenates (i.e., combine) the contents of all files in the current directory that have a .gb file extension.
```bash
$ cat *.gb 
```

Searches for a pattern in a file using extended regular expressions. The -E option enables the use of extended regular expressions instead of the default basic regular expressions.
Eg: "|" : indicate alternation between two patterns.
```bash
$ grep -E 'AC |OS|/product'
```

Uses **tr-** translate command to squeeze (i.e., remove) consecutive spaces in a text stream and replace them with a single space. 
```bash
$ tr -s " "
```

Uses **sed-** stream editor command to perform multiple text transformations on a text stream or file at once, including removing the first space character in each line, removing all occurrences of the |; character sequence, and removing all occurrences of the /product= character sequence.
```bash
$ sed "s/ //1; s/"|;//g; s/\/product=//g'
```

Removes all occurrences of the regular expression pattern AC|OS|FT from the input text.
```bash
$ sed -E 's/AC|OS|FT//g"
```

Merges three lines of input into a single line, with each line separated by a tab character.
```bash
$ paste - - - 
```

Redirects the standard output of a command to a file named table.tsv in the current directory.
```bash
> table.tsv
``` 

**PIPELINE**
```bash
$ cat *.gb | grep -E 'AC |OS|/product'| tr -s " " | sed -E 's/AC|OS|FT//g; s/ //1; s/"|;//g; s/\/product=//g' | paste - - - > table.tsv
```

### Method 2

Tests whether a string matches a regular expression. 
```bash 
=~
```

Indicates the beginning of the string.
```bash 
^
```

Deletes any semicolons (;).
```bash 
$ tr -d ';' 
``` 

Uses **awk** to remove the first field (i.e., the "OS" field) from the line and print the rest of the line. The result is a string that contains only the organism name.
```bash
$ awk '{$1=""; print $0}'
``` 

Uses **sed** to remove any leading spaces from the organism name. This is necessary because awk preserves the leading space that follows the "OS" field.
```bash
$ sed 's/ //'
```

Uses **awk** to extract the value of the "product" field from the line. Uses **-F=** option to specify that the field separator is the equals sign ("=") and then printing the second field (i.e., the value of the "product" field).
```bash
$ awk -F= '{print $2}'
```

Deletes any double quotes (").
```bash 
$ tr -d '"' 
``` 

**PIPELINE**
```bash 
# Function pare_AOP()
$ function pare_AOP(){
	while read -r line; do
		# Accession number
		if [[ $line =~ ^AC ]]; then   
			AC=$(echo $line| awk '{print $2}'| tr -d ';')    
		fi
		# ORGANISM
		if [[ $line =~ ^OS ]]; then   
			ORG=$(echo $line| awk '{$1=""; print $0}'| sed 's/ //')    
		fi
		# /product
		if [[ $line =~ ^FT && $line =~ /product= ]]; then   
			product=$(echo $line| awk -F= '{print $2}'| tr -d '"')    
		fi 
	done < $1
}
``` 

**Checks variables**: **[[ -n "$AC" ]]**: This is a conditional expression that checks if the length of the variable $AC is greater than zero (i.e., if it is not empty) using the **-n** null operator. If the condition is true (i.e., if $AC is empty), the variable $AC is assigned the value "NULL_AC" using the assignment operator =. If the condition is false (i.e., if $AC is not empty), the code inside the if block is not executed, and the value of $AC remains unchanged. This is useful for cases where a missing or empty variable needs to be replaced with a default value.
```bash 
# NULL_AC
$ if [[ -n "$AC" ]]
then
    AC="NULL_AC"
fi
# NULL_ORGANISM
$ if [[ -n "$ORG" ]]
then
    ORG="NULL_ORGANISM"
fi
# NULL_product
$ if [[ -n "$product" ]]
then
    product="NULL_product"
fi
``` 

**echo -e** displays a line of text to enable interpret backslash escapes. This prints out the value of the $AC, $ORG, $product variable, delimited by "===" characters and followed by a tab-separated character **(\t)**. The output is sent to the console.
```bash 
$ echo -e "==="$AC"\t==="$ORG"\t==="$product
```

**echo** without options to print out the values of the $AC, $ORG, and $product variables separated by tabs and sends the output to a file named "tb.tsv". The **>** symbol is used for output redirection to the file.
```bash 
$ echo -e $AC"\t"$ORG"\t"$product > tb.tsv
``` 

Adds header to tb.tsv.
```bash 
$ echo -e ACCESSION_NUMBER"\t"ORGANISM"\t"Product > tb.tsv
```

Searches the specified directory and any subdirectories for files with the extension ".gb" and return a list of their file paths.
```bash 
$ find $path -name "*.gb"
```

Does the same task pare_AOP for all the Accession List.
```bash 
$ path="/home/lucianhu/NG3/Assignments/DNA_RNAsequencing"
$ for i in 'find $path -name "*.gb"'; do
	echo $i
	pare_AOP $i
	echo -e "==="$AC"\t==="$ORG"\t==="$product
	echo -e $AC"\t"$ORG"\t"$product >> tb.tsv
done

$ cat tb.tsv
```  
### Method 3

Uses **printf** to print the string "Title" followed by each element of the $title array on a new line. The **%s** format specifier is used to print each element of the array as a string, and the **\n** character is used to insert a newline after each element. The **[@]** notation expands to all elements of the array, and the double quotes **{}** are used to preserve any whitespace or special characters in the array elements.
```bash
$ printf '%s\n' Title "${title[@]}") 
```

The command will take input text that is already separated into columns by tab characters, and format it into a more readable table format by adjusting the width of each column. The output will be displayed in the terminal. **-s** defines the column delimiter for output. **-t** applies for creating a table by determining the number of columns. **$'\t'** tells column to use the tab character as the column separator.
```bash
$ column -ts $'\t' 
```

**PIPELINE**
```bash
# Title 
$ title=$(while read -r l; do echo $l; done < access_list)
# accession number 
$ a=$(grep -w  -e  'ID' *.gb | awk -F "; " '{print $1}' | awk '{print $2}')
# organism 
$ b=$(grep -e 'organism' *.gb | awk -F "=" '{print $2}' |  tr -d '"')
# product
$ c=$(grep -e 'product' *.gb | awk -F "=" '{print $2}' |  tr -d '"')
# creat file 
$ paste <(printf '%s\n' Title "${title[@]}") \
      <(printf '%s\n' Assecsion_Number "${a[@]}") \
      <(printf '%s\n' Organism "${b[@]}") \
      <(printf '%s\n' Product "${c[@]}") \
| column -ts $'\t' > tb.tsv
```

## QUESTION 4
### Method 1
#### Creating FASTA (nucleotide) file from gb file.

Concatenates all files in the current directory with the extension ".gb".
```bash
$ cat *.gb
```

The regular expression "^ |AC " matches lines that start with a space or have "AC " in them. This filters out lines that don't match the desired pattern.
```bash
$ grep -E '^ |AC '
```

Removes all spaces from each line.
```bash
s/ //g
```

Removes any numbers that appear at the end of each line.
```bash
s/[0-9]+$//g
```

Removes all semicolons from each line.
```bash
s/;//g
```

Replaces all instances of "AC" with ">" because of the beginning of FASTA sequences being ">".
```bash
s/AC/>/g
```

Converts all lowercase letters to uppercase letters.
```bash
s/[a-z]/\U&/g
```

**PIPELINE**
```bash 
$ cat *.gb | grep -E '^ |AC ' | sed -E 's/ //g; s/[0-9]+$//g; s/;//g; s/AC/>/g; s/[a-z]/\U&/g' > mgp120.fa
```

#### Multi-line fasta to Single-line FASTA

**awk** command's **BEGIN** block, which is executed before any input is processed. Sets the **record separator (RS)** to ">", which means that each sequence in the file will be treated as a separate record. Sets the **output field separator (OFS)** to a tab character so that the output will be formatted as tab-separated values.
```bash 
$ awk 'BEGIN{RS=">";OFS="\t"}'
```

Matches all records except the first one (which is typically the header).
```bash 
$ NR>1
```

**awk** command substitutes the first occurrence of the newline character **\n** with a tab character **\t**.
```bash 
$ sub("\n","\t")
```

Globally substitutes (i.e., replaces all occurrences of) the newline character **\n** with an empty string. 
```bash 
$ gsub("\n","")
```

**PIPELINE**
```bash 
$ awk 'BEGIN{RS=">";OFS="\t"}; (NR>1){sub("\n","\t"); gsub("\n",""); print $1,$2} ' mgp120.fa
```

#### Length of every sequence
```bash 
$ awk 'BEGIN{RS=">";OFS="\t"}; (NR>1){sub("\n","\t"); gsub("\n",""); print $1,length($2)} ' mgp120.fa
```

### Method 2

This **$..$ command appends the contents of $line to the contents of $seq. 

The **${}** syntax is used to access the value of a variable in bash.

The **//** operator is used for **pattern substitution**, which means that all occurrences of the regular expression [0-9] (i.e., any digit) in $line will be replaced with an empty string, effectively removing all digits from the string. The resulting modified version of $line is then appended to the end of $seq.

```bash 
# Function pare_seq()
$ function pare_seq(){
	seq=''
	flag='off'
	while read -r line; do
		# Get sequence
		if [[ $flag =~ 'on' ]]; then   
			seq="${seq}${line//[0-9]/}"   
		fi
		# Meet SQ then start to read and header of sequence
		if [[ $line =~ ^SQ ]]; then   
			header=$line
			flag='on'    
		fi
        # Meet // then stop reading
	    if [[ $line =~ // ]]; then
		    flag='off'    
	    fi
	done < $1
}
```

Removes (i.e., deletes) a file or directory. In this case, rm gp120.fa would delete the file named "gp120.fa" if it exists.
```bash 
$ rm gp120.fa
```

Creates a new empty file named "gp120.fa" if it does not exist, or update the modification time of the file if it does exist.
```bash 
$ touch gp120.fa
```

A bash parameter expansion removes the ".gb" extension from the filename in $i. It replaces the first occurrence of ".gb" in the variable with an empty string.
```bash 
${i/.gb/}
```

**basename** extracts the filename from a file path. It takes a path as input and returns only the last component of the path (i.e., the filename).

**$(...)** command substitution in bash. It allows the output of a command to be substituted in place of the command itself.

Extracts the filename from the modified file path (without the ".gb" extension) using the basename command. The resulting filename is then returned as the output of the command substitution.
```bash 
$(basename ${i/.gb/})
```

**fold -** will split long lines of text into multiple lines, with each line containing no more than 20 characters. The line breaks will be inserted at the nearest whitespace character (e.g., space, tab) to the specified width. This is useful for displaying text in a fixed-width format, or for formatting text to fit a specific output device (e.g., a terminal window). For example, given an input file with a long line of text like "The quick brown fox jumps over the lazy dog", this command would split the line into "The quick brown fox jum", "ps over the lazy dog".
Does the same task pare_seq() for all the ACCs

**PIPELINE**
```bash
$ path="/home/lucianhu/NG3/Assignments/DNA_RNAsequencing"
$ for i in `find $path -name "*.gb"`; do
	echo $i
	pare_seq $i
	echo ">$(basename ${i/.gb/})" >> gp120.fa 
	echo -e ${seq// /}| tr -d '/' >> gp120.fa
done

$ cat gp120.fa| fold -20
```

