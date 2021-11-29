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

## FLAIR

### Installation and Dependencies
```shell
$ mkdir FLAIR
$ cd FLAIR
$ git clone https://github.com/BrooksLabUCSC/flair.git
$ cd flair
```
### Alignment Step with minimap2
```shell
$ minimap2 -ax splice <reference_genome.fa> <sample_reads.fa> <aligned_reads.sam> [options]
```

### Converting the read data to .bam format and sorting them by coordinates 
```shell  
$ samtools view -bS <aligned_reads.sam> > <aligned_sample.bam> [options]
$ samtools sort <aligned_sample.bam> -o <sorted_aligned_sample.bam> [options]
```

### Converting the bam files with a python script from FLAIR to bed format
```shell
$ python bam2Bed12.py -i <input.aligned.sorted.bam> > <aligned.sorted.bed>
```

### Subsequently the reads were corrected with the reference annotation
```shell
$ python flair.py correct -q <aligned_sorted.bed> -g <genome.fa> -f <annotation.gtf> -o <corrected_reads.bed> --threads 24
```

### Eventually the corrected reads will be collapsed into a set of non redundant isoforms 
```shell
$ python flair.py collapse -f <reference_annotation.gtf> -g <reference_genome.fa> -q <corrected_reads.bed> -o output_prefix -r <sample_reads.fa> --temp_dir /tmp_folder
```







