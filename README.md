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

## Quantitative analysis

## DESEQ


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

## StringTie2
### Offers an option which takes aligned and sorted reads from short and long read sequencing. The long reads have to be the second read file but no -L option is necessary. 
```shell
$ stringtie --mix <short_reads.bam> <long_reads.bam> -o <hybrid_assembly.gtf>
```

## FLAIR
### FLAIR's hybrid approach is based on correcting the long reads with short read splice junctions obtained. In this case, the short read splice junctions are generated by STAR aligner in two pass mode as <outfile_prefix.SJ.out.tab>. 
```shell
$ python flair.py correct -j <short_read_splice_junctions.SJ.out.tab> -g <genome.fa> -q <query.bed> -o <corrected_reads.bed> --threads 24
```

### However, if the short read corrected bed file is then used for the FLAIR collapse step, no <isoform.gtf> is generated. Therefore, we have to use the python script provided by flair to create it.
```shell
$ python flair.py collapse -j <short_read_splice_junctions.SJ.out.tab> -g <reference_genome.fa> -q <corrected_reads.bed> -o output_prefix -r <sample_reads.fa> --temp_dir /tmp_folder --threads 24

$ python psl_to_gtf.py <collapse.output.bed> > <collapsed_isoforms.gtf>
```



# Qualitative Evaluation 

## The first part of the quality control was runnning SQANTI3 to get a first impression how good the assemblies are and if it is viable to conduct further downstream analysis on them


### Installation and Dependencies

Building from source , creating an environment, and activating it
```shell
 $ git clone https://github.com/ConesaLab/SQANTI3.git
 $ cd SQANTI3
 $ conda env create -f SQANTI3.conda_env.yml
 $ source activate SQANTI3.env
 ```
 
 GTF to gene prediction is a obligatory utility for SQANTI3. It has to be downloaded directly into the SQANTI3/utilities folder
 ```shell
(SQANTI3.env)$ wget http://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/gtfToGenePred -P <PATH:TO>/SQANTI3/utilities/chmod +x <PATH:TO>/SQANTI3/utilities/gtfToGenePred
 
 Finally, before running the quality control pipeline, cDNA_Cupcake must also be downloaded, installed and the two folders  /SQANTI3/cDNA_Cupcake/sequence and /SQANTI3/cDNA_Cupcake/ have to be added to $PYTHONPATH
 ```shell
(SQANTI3.env)$ git clone https://github.com/Magdoll/cDNA_Cupcake.git
(SQANTI3.env)$ cd cDNA_Cupcake
(SQANTI3.env)$ python setup.py build
(SQANTI3.env)$ python setup.py install
(SQANTI3.env)$ export PYTHONPATH=$PYTHONPATH:/PATH:TO/SQANTI3/cDNA_Cupcake/sequence/
(SQANTI3.env)$ export PYTHONPATH=$PYTHONPATH:/PATH:TO/MYO/SQANTI3/cDNA_Cupcake/
```

