---
title: 'Oddly Specific, Potentially Useful Code - Data Analysis Unix snippets'
date: 2023-08-04
permalink: /posts/2023/08/blog-post-3/
tags:
  - code
  - unix
  - tutorial
---

This is a collection of oddly specific, potential useful code snippets that I use frequently enough in my general data analysis work in unix that I wanted to put them in one place as a reference. Some of these things can be a bit of a pain to figure out or look up, so I wanted to share in case it helps someone else.

Each of the below code sets is completely independent from one another if under a separate comment (#), even within a given code block.

Fun things with for loops
=====
```unix
# General for loop structure for custom list of variables
var=myfile1.txt
for met in var1 var2 var3 var4 var5 var6
do
    sumstats=myfile2.gz
    output=myoutputfile.gz
    zgrep -wFf ${var} ${sumstats} | gzip -c > ${output}
    echo $met
    echo $sumstats
    echo $output
done

# General for loop structure for piping the contents of a file into the for loop as variables    
while read p; do
echo $p
done <myfile.txt

# Rename all files in a directory, three different scenarios
for x in *.log; do y=$(sed 's/_/-/2' <<< "$x"); mv "$x" "$y"; done # with a certain extension by changing the second occurrence of hyphen to a dash
for x in *; do y=$(sed 's/all_metabolites/all_metabolites164/' <<< "$x"); mv "$x" "$y"; done # switch all files that start with all_metabolites to starting with all_metabolites164
for filename in *.txt.gz; do mv -i "$filename" "${filename#*_*_}"; done # delete everything before and including second underscore

# Move all files that meet a certain criteria in one folder to another
for x in myfileprefix*.gz; do y="myfileprefix/"; mv $x ${y}${x}; done

# Print out all file names for files that have over a certain number of lines
for i in *.txt
do
num=`wc -l $i | tr ' ' '\n' | head -n 1`
if (( num > 50)); then
    echo $i
fi
done
    
# Loop through pairwise combinations but not include repeat pairs (if have a-b don’t have b-a)
while read met1; do
  while read met2; do
    if [ "$met1" \< "$met2" ]
    then
     echo "Pairs $met1 and $met2"
    fi
    done <myfile1.txt
  done <myfile1.txt
    
# Loop through files and reformat some of the column header names and add new columns based on dividing/summing other columns
while read t; do
echo $t
file_name=${t}_myfilesuffix.txt.gz
in_file_path=/path/to/my/file/
in_file=${in_file_path}${file_name}
out_file_path=/path/to/write/file/to/
out_file=${out_file_path}${file_name}
sed -e '1s/rsid/SNP/' -e '1s/#CHR/CHR/' -e '1s/POS/BP/' -e '1s/ALT/A1/' -e '1s/REF/A2/' $in_file > temp.txt.gz # add custom header
zcat temp.txt.gz | awk 'BEGIN { FS = "\t" ; OFS = "\t"} ; {if(NR == 1) print $0,"Z","N"; else if(NR > 1) print $0, $7/$8,$12+$13}' > $out_file
rm temp.txt.gz
done <traits.txt
    
# Progressively add extra columns from dif files to one file
in_file=/my/path/filename1.tsv.gz
out_file=/my/output/path/filename2.tsv.gz
zcat ${in_file} | cut -f 1,7 > ${out_file}
while read t; do
echo $t
file_name=fileprefix.${t}.filesuffix.gz
in_file_path=/my/path/
in_file=${in_file_path}${file_name}
out_file2=/my/output/path/filename3.txt
zcat ${in_file} | cut -f 11,16 | sed -e '1s/BETA/BETA_'"${t}"'/' -e '1s/P_BOLT_LMM/P_BOLT_LMM_'"${t}"'/' | paste $out_file - > $out_file2 # add custom header to munged file
mv $out_file2 $out_file
done <filename4.txt
    
# Go through files with an extension and rename them to not have their extension, then gzip them
ls *.txt.gz | while read file ; do mv $file $(basename $file .gz); gzip $(basename $file .gz); done
```

Code for exploring and getting a feel for directories and files
======
```unix
# Find all subdirectories in the current directory that are empty
find . -mindepth 1 -maxdepth 1 -empty -type d

# Find all subdirectories in the current directory that are NOT empty
find . -mindepth 1 -maxdepth 1 -not -empty -type d

# Find all files with a certain substring in the filename
find . -name "*.HDL_L.*"

# Count how many directories have more than one file in them (can remove the | wc -l if want to view which they are) and such
find . -type f -printf '%h\n' | sort | uniq -d | wc -l
find . -type f -printf '%h\n' | sort | uniq -c # or see how many files each directory has
find . -type f -printf '%h\n' | sort | uniq -c | grep -w 1 # or search for which directories have a specific number of files (1 in this case)

# Count files + folders in a directory
ls | wc -l
```

Munging files and directories
======
```unix
# Remove all slurm files in a directory
find . -name "slurm*" -exec rm {} \; # can do `find . -name "slurm*"` first to see what will be deleted

# Cleanly list the difference between two files
diff metabolite_list.txt all_metabolites_list.txt | grep "<" | sed 's/^<//g'

# Filter file by another file
awk -F'\t' 'NR==FNR{a[$1];next} $1 in a' keep.tsv fullfile.tsv > filtered.txt # filters fullfile.tsv to only keep rows that match value in its col 1 with the keep.txt file's col 1

# Combine files
awk 'NR==FNR{A[$2]=$0; next} $3 in A {print A[$3], $0}' file2 file1 > newfile # This keeps the header. `A[$2]=$0` applies to file2, where A stores the entire file2, indexed by its second column. `$3 in A` checks if the third column of file1 is in A, if so, append the rows.
    
# Split the specified file into new files named by the value in the specified column (6 here)
awk -F'\t' '{print>$6}' allsig_metvarpairs_1e6_annotated.tsv

# Delete all files and directories (or just delete files if add -type f) from last x minutes (here 3) or days
find . -cmin -3 -delete # from last x minutes (here 3)
find . -type f -mtime +7 -name '*.log' -execdir rm -- '{}' + # delete all files in this directory from older than 7 days

# Add header to all files (can do *.txt if just want txt files; change from CHROM to Gene_names_allannotated with the header of choice)
sed -i '1i CHROM POS Effect_allele Other_allele P MetaboliteName SNP Consequence Gene_names_allannotated' *

# Make a new file with a specific row deleted based on a string in that line
sed '/rs10778203/d' ./Phenylalanine_levels > temp_phe_nors10778203
```
