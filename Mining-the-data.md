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
```bash
$ cat *.gb | grep -E 'AC |OS|/product'| tr -s " " | sed -E 's/AC|OS|FT//g; s/ //1; s/"|;//g; s/\/product=//g' | paste - - - > table.tsv
```
Explanation:
- `cat *.gb` concatenates (i.e., combine) the contents of all files in the current directory that have a `.gb` file extension.
- `grep -E 'AC |OS|/product'` searches for a pattern in a file using extended regular expressions. The `-E` option enables the use of extended regular expressions instead of the default basic regular expressions.
Eg: `"|"` : indicate alternation between two patterns.
- `tr -s " "` uses `tr-` translate command to squeeze (i.e., remove) consecutive spaces in a text stream and replace them with a single space. 
- `sed "s/ //1; s/"|;//g; s/\/product=//g'` uses `sed-` stream editor command to perform multiple text transformations on a text stream or file at once, including removing the first space character in each line, removing all occurrences of the `|;` character sequence, and removing all occurrences of the `/product=` character sequence.
- `sed -E 's/AC|OS|FT//g"` removes all occurrences of the regular expression pattern `AC|OS|FT` from the input text.
- `paste - - -` merge three lines of input into a single line, with each line separated by a tab character.
- `> table.tsv` redirects the standard output of a command to a file named `table.tsv` in the current directory.

```bash
sed -i 's/original/new/g' file.txt
```
Explanation:
- `sed` = Stream EDitor
- `-i` = in-place (i.e. save back to the original file)
- `-E` = extended regular expression rather than basic regular expressions.
- `s` = the substitute command
- `original` = a regular expression describing the word to replace (or just the word itself)
-  `new` = the text to replace it with
- `g` = global (i.e. replace all and not just the first occurrence)
- `file.txt` = the file name

### Method 2
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
Explanation:
- `=~` tests whether a string matches a regular expression. 
- `^` indicates the beginning of the string.
- `tr -d ';'` deletes any semicolons (;).
- `awk '{$1=""; print $0}'` uses `awk` to remove the first field (i.e., the `OS` field) from the line and print the rest of the line. The result is a string that contains only the organism name.
- `sed 's/ //'` uses `sed` to remove any leading spaces from the organism name. This is necessary because `awk` preserves the leading space that follows the `OS` field.
- `awk -F= '{print $2}'` uses `awk` to extract the value of the `product` field from the line. Uses `-F=` option to specify that the field separator is the equals sign `=` and then printing the second field (i.e., the value of the `product` field).
- `tr -d '"'` deletes any double quotes `"`.

**Checks variables**: `[[ -n "$AC" ]]`: This is a conditional expression that checks if the length of the variable `"$AC"` is greater than zero (i.e., if it is not empty) using the `-n` null operator. If the condition is true (i.e., if `"$AC"` is empty), the variable `"$AC"` is assigned the value `"NULL_AC"` using the assignment operator `=`. If the condition is false (i.e., if `"$AC"` is not empty), the code inside the if block is not executed, and the value of `"$AC"` remains unchanged. This is useful for cases where a missing or empty variable needs to be replaced with a default value.
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

Adds header to tb.tsv.
```bash 
$ echo -e ACCESSION_NUMBER"\t"ORGANISM"\t"Product > tb.tsv
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
Explanation:
- `find $path -name "*.gb"` searches the specified directory and any subdirectories for files with the extension `".gb"` and return a list of their file paths.
- `echo -e "==="$AC"\t==="$ORG"\t==="$product`: `echo -e` displays a line of text to enable interpret backslash escapes. This prints out the value of the `$AC`, `$ORG`, `$product` variable, delimited by `"==="` characters and followed by a tab-separated character `(\t)`. The output is sent to the console.
- `echo -e $AC"\t"$ORG"\t"$product > tb.tsv` prints out the values of the `$AC`, `$ORG`, and `$product` variables separated by tabs `"\t"` and sends the output to a file named `tb.tsv`. 
- The `>>` operator is used for redirecting the output of a command to a file in append mode.


### Method 3
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
Explanation:
- `printf '%s\n' Title "${title[@]}")` uses `printf` to print the string `Title` followed by each element of the `$title` array on a new line. The `%s` format specifier is used to print each element of the array as a string, and the `\n` character is used to insert a newline after each element. The `[@]` notation expands to all elements of the array, and the double quotes `{}` are used to preserve any whitespace or special characters in the array elements.
- `column -ts $'\t'` takes input text that is already separated into columns by tab characters, and format it into a more readable table format by adjusting the width of each column. The output will be displayed in the terminal. `-s` defines the column delimiter for output. `-t` applies for creating a table by determining the number of columns. `$'\t'` tells column to use the tab character as the column separator.

## QUESTION 4
### Method 1
#### Creating FASTA (nucleotide) file from gb file.
```bash 
$ cat *.gb | grep -E '^ |AC ' | sed -E 's/ //g; s/[0-9]+$//g; s/;//g; s/AC/>/g; s/[a-z]/\U&/g' > mgp120.fa
```
Explanation:
- `cat *.gb` concatenates all files in the current directory with the extension `.gb`.
- `grep -E '^ |AC '` The regular expression `^ |AC ` matches lines that start with a space or have `AC ` in them. This filters out lines that do not match the desired pattern.
- `s/ //g` removes all spaces from each line.
- `s/[0-9]+$//g` removes any numbers that appear at the end of each line.
- `s/;//g` removes all semicolons from each line.
- `s/AC/>/g` replaces all instances of `AC` with `>` because of the beginning of FASTA sequences being `>`.
- `s/[a-z]/\U&/g` converts all lowercase letters to uppercase letters.

#### Multi-line fasta to Single-line FASTA
```bash 
$ awk 'BEGIN{RS=">";OFS="\t"}; (NR>1){sub("\n","\t"); gsub("\n",""); print $1,$2} ' mgp120.fa
```
Explanation:
- `awk 'BEGIN{RS=">";OFS="\t"}'`: `awk` command's `BEGIN` block, which is executed before any input is processed. Sets the `record separator (RS` to `">"`, which means that each sequence in the file will be treated as a separate record. Sets the `output field separator (OFS)` to a tab characterv `"\t"`so that the output will be formatted as tab-separated values.
- `NR>1` matches all records except the first one (which is typically the header).
- `sub("\n","\t")` replaces only the first occurrence of the newline character `"\n"` with a tab character `"\t"`.
- `gsub("\n","")` replaces occurrences in whole string of the newline character `"\n"` with an empty string `""`.

#### Length of every sequence
```bash 
$ awk 'BEGIN{RS=">";OFS="\t"}; (NR>1){sub("\n","\t"); gsub("\n",""); print $1,length($2)} ' mgp120.fa
```

### Method 2
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
Explanation:
- `$..$` command appends the contents of `$line` to the contents of `$seq`. 
- `${}` syntax is used to access the value of a variable in bash.
- `//` operator is used for **pattern substitution**, which means that all occurrences of the regular expression `[0-9]` (i.e., any digit) in `$line` will be replaced with an empty string, effectively removing all digits from the string. The resulting modified version of `$line` is then appended to the end of `$seq`.

`rm gp120.fa` removes (i.e., deletes) a file or directory. In this case, rm gp120.fa would delete the file named "gp120.fa" if it exists.

`touch gp120.fa` creates a new empty file named "gp120.fa" if it does not exist, or update the modification time of the file if it does exist.

Does the same task pare_seq() for all the ACCs
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
Explanation:
- `${i/.gb/}`: A bash parameter expansion removes the `.gb` extension from the filename in `$i`. It replaces the first occurrence of `.gb` in the variable with an empty string.
- `basename` extracts the filename from a file path. It takes a path as input and returns only the last component of the path (i.e., the filename).
- `$(...)` command substitution in bash. It allows the output of a command to be substituted in place of the command itself.
- `(basename ${i/.gb/})` extracts the filename from the modified file path (without the `".gb"` extension) using the basename command. The resulting filename is then returned as the output of the command substitution.
- `fold -` will split long lines of text into multiple lines, with each line containing no more than 20 characters. The line breaks will be inserted at the nearest whitespace character (e.g., space, tab) to the specified width. This is useful for displaying text in a fixed-width format, or for formatting text to fit a specific output device (e.g., a terminal window). For example, given an input file with a long line of text like `The quick brown fox jumps over the lazy dog`, this command would split the line into `The quick brown fox jum` and `over the lazy dog`.
