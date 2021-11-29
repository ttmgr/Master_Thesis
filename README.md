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

### The final output are high-confidence isoforms in several formats, however interesting for us are only the <isoforms.fasta> and the <isoforms.gtf>.



## StringTie2

### Installation and Dependencies
```shell
$ git clone https://github.com/mpertea/stringtie2
$ cd stringtie2
$ make release
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

### Afterwards StringTie2 is applied to assemble the transcriptome. The -L option is obligatory for long read based assemblies

```shell  
$ stringtie -L <sorted_aligned_sample.bam> -o <assembly.gtf> [options]
```



# Short read data

### Installation and Dependencies
```shell
wget https://github.com/alexdobin/STAR/archive/2.7.9a.tar.gz
tar -xzf 2.7.9a.tar.gz
cd STAR-2.7.9a
```

### The first step for aligning paired-end reads with STAR is generating a genome index for it
```shell
STAR --runThreadN 10  --runMode genomeGenerate --genomeDir /path/to/STAR/genome/folder --genomeFastaFiles /path/to/genome/folder
```

### The alignment step is then executed with respect to generated index. Here the option --outSAMstrandField intronMotif is necessary for the transcriptome assembly downstream. 
```shell
STAR --runThreadN 10 --runMode alignReads --outSAMtype BAM Unsorted --genomeDir /path/to/STAR/genome/folder --outFileNamePrefix {sample name} --readFilesIn <read1_file.fa> <read2_file.fa> --sdjdbGTFfile <reference_annotation.gtf> --twopassMode Basic --outSAMstrandField intronMotif
```

### Now the reads have again to be sorted by coordinates with samtools. In theory sorting it on-the-fly within STAR is possible, but it requires a lot of computational resources.
```shell
samtools sort <aligned_reads.bam> -o <sorted_reads.bam> [options]
```

### Afterwards StringTie2 is applied to assemble the transcriptome. The -L option is obligatory for long read based assemblies
```shell  
$ stringtie <sorted_reads.bam> -o <assembly.gtf> [options]
```



# Hybrid Approaches

## StringTie2 offers an option which takes aligned and sorted reads from short and long read sequencing. The long reads have to be the second read file but no -L option is necessary. 
```shell
stringtie --mix <short_reads.bam> <long_reads.bam> -o <mixed.assembly.gtf>
```



















