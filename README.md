#UNIX Assignment - *Cassie Winn*

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
head -n 1 fang_et_al_genotypes.txt > maize_genotypes.txt
```
```
head -n 1 fang_et_al_genotypes.txt > teosinte_genotypes.txt
```
```
$ awk '$3 ~ /ZMMIL|ZMMLR|ZMMMR/ { print $0}' fang_et_al_genotypes.txt >> maize_genotypes.txt
```

```
awk '$3 ~ /ZMPBA|ZMPIL|ZMPJA/ { print $0}' fang_et_al_genotypes.txt >> teosinte_genotypes.txt
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
awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt
```
```
awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt
```
```
$ wc transposed_maize_genotypes.txt transposed_teosinte_genotypes.txt 
     986  1550978  6240114 transposed_maize_genotypes.txt
     986   961350  3873338 transposed_teosinte_genotypes.txt
    1972  2512328 10113452 total
```

###Format Genotype Files & Sort
Recall that the snp_position.txt file has 984 lines, while the new genotype files has 986 lines. This is because the genotype files have two lines containing additional info. So separate out the two top lines into different files.
```
$ tail -n +4 transposed_maize_genotypes.txt > maize_tran.txt
```
```
$ head -n 3 transposed_maize_genotypes.txt > maize_header.txt
```

Check that it worked by cat and head
```
$ cut -f 1-5 maize_header.txt | column -t | head
```
Then manually add in column headers for 

```
$ sort -k1,1 maize_tran.txt > maize_tran_sort.txt
```

###Rearrange and Sort SNP file
Cut out the columns of snp_position.txt that are needed and compile them in one file in the order desired (SNP_ID, Chromosome, Position).

```
$ cut -f 1 snp_position.txt > snp.txt
$ cut -f 3 snp_position.txt > snp_chrom.txt
$ cut -f 4 snp_position.txt > snp_pos.txt
```
```
$ paste snp.txt snp_chrom.txt > snp_plus_chrom.txt
$ paste snp_plus_chrom.txt snp_pos.txt  > snp_all.txt
```
Before the SNP file can be joined with genotype files, it must be sorted.

```
$ sort -k1,1 snp_all.txt > snp_all_sort.txt
```

###Join SNP and Genotype Files

###Extract Data for Input Files