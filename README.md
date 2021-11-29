# Master_Thesis - Tim Reska - 11182193

The following github presents a pipeline for the assembly of transcriptomes based on short or long reads as well as 
combined approach.


# Applied Sofwarte
## Alignment Software
-minimap2 for long reads
-STAR for short reads

## Sorting program
-samtools applied on both to sort the reads by coordinates

## Transcriptome assembler
-StringTie2 was used in separate approaches for both long and short read data
-FLAIR was used solely on long reads

## Qualitative Evaluation
-Gffcompare and SQANTI3 were used for evaluation of all assemblies

## Additional requirements
-bedtools
-python v3.0+ and python modules: intervaltree,kerneltree,tqdm,pybedtools and pysamv0.8.4+


# Long read data

##FLAIR

### Installation and Dependencies
```shell
$ mkdir FLAIR
$ cd FLAIR
$ git clone https://github.com/BrooksLabUCSC/flair.git
$ cd flair
```











