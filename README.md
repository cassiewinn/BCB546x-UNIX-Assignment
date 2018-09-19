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
$ wc fang_et_al_genotypes.txt 
2783  2744038 11051939 fang_et_al_genotypes.txt
```

```
$ wc snp_position.txt 
  984 13198 82763 snp_position.txt
```

Alternatively, if we just want to print the number of lines in the file, use the wc command with the -l option.

```
$ wc -l fang_et_al_genotypes.txt 
2783 fang_et_al_genotypes.txt
```
```
$ wc -l snp_position.txt 
984 snp_position.txt
```

##*Data Processing*

First, I extracted the maize and teosinte data and put them in new files. 

```
$ awk '$3 ~ /ZMMIL|ZMMLR|ZMMMR/ { print $0}' fang_et_al_genotypes.txt > maize_genotypes.txt
```

```
awk '$3 ~ /ZMPBA|ZMPIL|ZMPJA/ { print $0}' fang_et_al_genotypes.txt > teosinte_genotypes.txt
```
I checked to make sure my extractions worked by looking at the number of lines in each file.

```
$ wc -l maize_genotypes.txt 
1573 maize_genotypes.txt
```
```
wc -l teosinte_genotypes.txt 
975 teosinte_genotypes.txt
```
I also checked to see that each file had only the groups that I wanted by sorting the files by column 3 and running the command unique.

