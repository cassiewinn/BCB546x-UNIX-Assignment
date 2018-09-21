#UNIX Assignment - *Cassie Winn*
Due Friday, September 21st 5pm

##*Data Inspection*

###Determining File Size

```
$ du -h fang_et_al_genotypes.txt 
11M	fang_et_al_genotypes.txt
```
```
$ du -h snp_position.txt 
84K	snp_position.txt
```

###Determining Number of Columns

```
$ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
986
```

```
$ awk -F "\t" '{print NF; exit}' snp_position.txt 
15
```

###Determining Number of Lines, Words and Characters
If we want to look at number of lines, words and bytes (characters) use the wc command.

```
$ wc fang_et_al_genotypes.txt snp_position.txt
	2783  2744038 11051939 fang_et_al_genotypes.txt
	984    13198    82763 snp_position.txt
	3767  2757236 11134702 total
```
Alternatively, if we just want to print the number of lines in the file, use the wc command with the -l option.

```
$ wc -l fang_et_al_genotypes.txt snp_position.txt
	2783 fang_et_al_genotypes.txt
	984 snp_position.txt
	3767 total
```

##*Data Processing*
###Extracting Info from Files
First, I took the column headers and put them in new files. Then, I extracted the maize and teosinte data and appended them to the new files. 

```
$ head -n 1 fang_et_al_genotypes.txt > maize_genotypes.txt
```
```
$ head -n 1 fang_et_al_genotypes.txt > teosinte_genotypes.txt
```
```
$ awk '$3 ~ /ZMMIL|ZMMLR|ZMMMR/ { print $0}' fang_et_al_genotypes.txt >> maize_genotypes.txt
```

```
$ awk '$3 ~ /ZMPBA|ZMPIL|ZMPJA/ { print $0}' fang_et_al_genotypes.txt >> teosinte_genotypes.txt
```
I checked to make sure my extractions worked by looking at the number of lines, words and characters in each file.

```
$ wc maize_genotypes.txt teosinte_genotypes.txt
	1574  1551964  6250961 maize_genotypes.txt
	976   962336  3884185 teosinte_genotypes.txt
	2550  2514300 10135146 total
```
I also checked to see that each file had only the groups that I wanted by cutting the groups from column 3, sorting them and running the command unique and getting a count.

```
$ cut -f 3 maize_genotypes.txt | sort | uniq -c
      1 Group
    290 ZMMIL
   1256 ZMMLR
     27 ZMMMR
```
```
$ cut -f 3 teosinte_genotypes.txt | sort | uniq -c
      1 Group
    900 ZMPBA
     41 ZMPIL
     34 ZMPJA
```
###Tranposing Files
Next, I transposed my genotype data so that the columns became rows. I used the transpose.awk script provided, and checked line, word and character counts. We expect the two transposed files to have new, but matching line counts.

```
$ awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt
```
```
$ awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt
```
```
$ wc transposed_maize_genotypes.txt transposed_teosinte_genotypes.txt 
     986  1550978  6240114 transposed_maize_genotypes.txt
     986   961350  3873338 transposed_teosinte_genotypes.txt
    1972  2512328 10113452 total
```

###Format Genotype Files & Sort
The snp_position.txt file has 984 lines, while the new genotype files each have 986 lines. This is because the genotype files have two lines containing additional info at the beginning of the file. So first, I separated out the two top lines into different files, to be used later. Below is an example using the maize files.

```
$ tail -n +4 transposed_maize_genotypes.txt > maize_tran.txt
```
```
$ head -n 3 transposed_maize_genotypes.txt > maize_header.txt
```

Check that it worked by using the commands cat and head:

```
$ cut -f 1-5 maize_header.txt | column -t | head
```

Then I manually added in column headers ("SNP_ID" "Chromosome" "Position") in header.txt by editing in vi
```
$ vi maize_header.txt
```

Again, I checked the file by using cat and head:

```
$ cut -f 1-5 maize_header.txt | column -t | head

SNP_ID  Chromosome  Position  ZDP_0752a  ZDP_0793a
SNP_ID  Chromosome  Position  JG_OTU     Zmm-LR-ACOM-usa-NM-1_s
SNP_ID  Chromosome  Position  ZMMLR      ZMMLR
```

Next, I sorted the file by SNP_ID (column 1)

```
$ sort -k1,1 maize_tran.txt > maize_tran_sort.txt
```

###Rearrange and Sort SNP file
I then cut out the columns of snp_position.txt that are needed (1, 3 and 4) and compiled them in one file in the order desired (SNP ID, Chromosome, Position).

```
$ cut -f 1 snp_position.txt > snp.txt
$ cut -f 3 snp_position.txt > snp_chrom.txt
$ cut -f 4 snp_position.txt > snp_pos.txt
```
```
$ paste snp.txt snp_chrom.txt > snp_plus_chrom.txt
$ paste snp_plus_chrom.txt snp_pos.txt  > snp_all.txt
```
Before the SNP file can be joined with genotype files, it must be sorted by SNP_ID as well (column 1).

```
$ sort -k1,1 snp_all.txt > snp_all_sort.txt
```

###Join SNP and Genotype Files

Now that the SNP and Genotype files have both been formatted and sorted, it is time to join them.
 
```
$ join -1 1 -2 1 -t $'\t' -e 'empty' snp_all_sort.txt maize_tran_sort.txt > maize_join.txt
```
We then need to cat the joined file with the header file.

```
$ cat maize_header.txt maize_join.txt > maize.txt
```

###Extract Data for Input Files

Create 10 files (1 for each chromosome) where SNPs are ordered based on increasing position and missing data encoded by this symbol: ?
The awk and cat commands below are run 10 times for each chromosome to create 10 _increasing.txt files.

```
$ sort -k2,2n -k3,3n maize.txt > maize_sort.txt
```
```
$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' maize_sort.txt > maize_chrom_01_noheader.txt
```
```
$ cat maize_header.txt maize_chrom_01_noheader.txt > maize_chrom_01_increasing.txt
```
Create 10 files (1 for each chromosome) where SNPs are ordered based on decreasing position and missing data is encoded by this symbol: -
The awk and cat commands below are run 10 times for each chromosome to create 10 _decreasing.txt files.

```
$ sort -k2,2n -k3,3nr maize.txt > maize_sort_decrease.txt
```
```
$ sed 's/?/-/g' maize_sort_decrease.txt > maize_decrease.txt
```
```
$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' maize_decrease.txt > maize_chrom_01_noheader_decrease.txt
```
```
$ cat maize_header.txt maize_chrom_01_noheader_decrease.txt > maize_chrom_01_decreasing.txt
```
Create 1 file with all SNPs with unknown positions in the genome.

```
$ awk '{ if ($2 ~ /unknown/ || $3 ~ /unknown/) { print } }' maize_sort.txt > maize_unknown_noheader.txt
```
```
$ cat maize_header.txt maize_unknown_noheader.txt > maize_unknown.txt
```
Create 1 file with all SNPs with multiple positions in the genome.

```
$ awk '{ if ($2 ~ /multiple/ || $3 ~ /multiple/) { print } }' maize_sort.txt > maize_multiple_noheader.txt
```
```
$ cat maize_header.txt maize_chrom_multiple_noheaderz.txt > maize_chrom_mult.txt
```
###Repeat Format, Join and Extract Steps for Teosinte Files (22 files created)
Format:

```
$ tail -n +4 transposed_teosinte_genotypes.txt > teosinte_tran.txt
$ head -n 3 transposed_teosinte_genotypes.txt > teosinte_header.txt
$ vi teosinte_header.txt
$ sort -k1,1 teosinte_tran.txt > teosinte_tran_sort.txt
```
Join:

```
$ join -1 1 -2 1 -t $'\t' -e 'empty' snp_all_sort.txt teosinte_tran_sort.txt > teosinte_join.txt
$ cat teosinte_header.txt teosinte_join.txt > teosinte.txt
```
Extract:

```
$ sort -k2,2n -k3,3n teosinte.txt > teosinte_sort.txt
$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' teosinte_sort.txt > teosinte_chrom_01_noheader.txt
$ cat teosinte_header.txt teosinte_chrom_01_noheader.txt > teosinte_chrom_01_increasing.txt
$ sort -k2,2n -k3,3nr teosinte.txt > teosinte_sort_decrease.txt
$ sed 's/?/-/g' teosinte_sort_decrease.txt > teosinte_decrease.txt
$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' teosinte_decrease.txt > teosinte_chrom_01_noheader_decrease.txt
$ cat teosinte_header.txt teosinte_chrom_01_noheader_decrease.txt > teosinte_chrom_01_decreasing.txt
$ awk '{ if ($2 ~ /unknown/ || $3 ~ /unknown/) { print } }' teosinte_sort.txt > teosinte_unknown_noheader.txt
$ cat teosinte_header.txt teosinte_unknown_noheader.txt > mteosinte_unknown.txt
$ awk '{ if ($2 ~ /multiple/ || $3 ~ /multiple/) { print } }' teosinte_sort.txt > teosinte_multiple_noheader.txt
$ cat teosinte_header.txt teosinte_chrom_multiple_noheader.txt > teosinte_chrom_mult.txt
```