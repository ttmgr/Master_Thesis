# Master_Thesis - Tim Reska - 11182193
# Tim Reska - 11182193 - Hybrid Assembly - Myo Dataset 

# Short summary written Master Thesis

- Introduction
- Background Information
  - Mouse Model - Why mice, advantages - applications
  - Myogenesis - Why muscle proliferation is good for sequencing experiments and helpful in science in general
- Sequencing 
  - Illumina vs ONT   
  - Hybrid Sequencing approaches 
- Materials and Methods
  - Materials
- Methods 
  - Qualitative Methods summary 
  - Quantitative Methods summary 
  - Isoform analysis and differential transcript usage summary
- Results
- Qualitative Analysis
  - Reference versus Assemblies
  I start with the quality control of sqanti, of the Myo samples versus the reference. I am explaining which kind of matches are the most important, and which are an indicator that the assemblies might be contaminated by artifacts or misassemblies e.g. high percentage of intergenic genes. 
  - Cross-comparing assemblies
  Afterwards comes the numerical comparison of the samples with respect to the assembly as well as the cross-comparison between the different assemblies in terms of precision and intron chain level. Lastly, I will show examples of transcripts only found by SR, LR or Mix in form of IGV Screenshots.
- Quantitative Analysis
The quantitative analysis will rely on generating quantitative expression scores and explaining these for the samples, as well as basic statistics for the read counts: avg. count per transcript, how many transcripts each method found.
- Differential Analysis
  - Up- and down-regulated genes
  Here i will describe the differential gene expression, find examples of up- or down-regulated genes of which are necessary for the proliferation from myoblast to myotube. Show the differential gene expression of long reads and the mixed variants with the help of volcano plots, trying to highlight certain genes.  
  - Differential isoform usage
  Show examples for differential isoform usage, show differences in isoforms between flair and salmon transcripts
- Outlook and Discussion
  







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
-RStudio
-gffread
-subreads
-salmon

## DESEQ
-DESeq2


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
$ stringtie -L <sorted_aligned_sample.bam> -o <long_assembly.gtf> [options]
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
$ stringtie <sorted_reads.bam> -o <short_assembly.gtf> [options]
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
 ```
 
 Finally, before running the quality control pipeline, cDNA_Cupcake must also be downloaded, installed and the two folders  /SQANTI3/cDNA_Cupcake/sequence and /SQANTI3/cDNA_Cupcake/ have to be added to $PYTHONPATH
 ```shell
(SQANTI3.env)$ git clone https://github.com/Magdoll/cDNA_Cupcake.git
(SQANTI3.env)$ cd cDNA_Cupcake
(SQANTI3.env)$ python setup.py build
(SQANTI3.env)$ python setup.py install
(SQANTI3.env)$ export PYTHONPATH=$PYTHONPATH:/PATH:TO/SQANTI3/cDNA_Cupcake/sequence/
(SQANTI3.env)$ export PYTHONPATH=$PYTHONPATH:/PATH:TO/MYO/SQANTI3/cDNA_Cupcake/
```

### Necessary Input Assembled Transcriptome as GTF/GFF3 (with the --gtf option); but stringtie generates unwanted textlines within the gtf which have to be removed to conduct the quality control. Furthermore, before we cann apply SQANTI3 transcripts without strandinformation have to be removed from the transcriptome.

```shell
(SQANTI3.env) $ awk '$7 != "."' <assembly.gtf> > <cleaned_assembly.gtf>
(SQANTI3.env) $ python sqanti3_qc.py --gtf <TranscriptomeOfInterest.Sample.gtf|gff3> <Annotation.gtf|gff3> <ReferemceGenome.Fasta> -t 24 
```

SQANTI3 generates different files, however most useful is the <report.pdf>, where metrics such as number of annotated genes, isoforms, and read distribution as well as splices matches with respect to the annotation are visible.



## The next part of the qualitative evaluation was divided into several steps. First the long, short, and hybrid assemblies were compared to the annotation, secondly the overlaps between the different methods were removed. This was done in order to see which transcripts are found either by the short, long, or hybrid approaches. As sanity check, the short and long read assemblies were concatenated and compared to the hybrid approach.

### Installation and dependencies
```shell
$ git clone https://github.com/gpertea/gffcompare
$ cd gffcompare
$ make release
```

### Separating the reads into unique groups based on their assembly method

First we use gffcompare to match the short read assemblies to the long read assemblies and vice versa.
```shell
$ gffcompare -r <long_read.gtf> -o LR.vs.SR <short_read.gtf>  
$ gffcompare -r <short_read.gtf> -o SR.vs.LR <long_read.gtf>
```

Second, we extract the transcript matches from the and .tmap generated by the step above, which are either unique to short reads, long reads or are common to both. For the latter one we take the .refmap instead of the .tmap.
```shell
$ cat <SR.vs.LR.SR.gtf.tmap> | awk '$3!="="{print $5}' | sort > Sample.UniqueLR.txt
$ cat <SR.vs.LR.LR.gtf.refmap> | awk '$3=="="{print $2}' | sort > Sample.Common.txt
$ cat <LR.vs.SR.SR.gtf.tmap> | awk '$3!="="{print $5}' | sort > Sample.UniqueSR.txt
```

Third, gtfFilter is applied to filter out unique and common transcripts and create the corresponding assembly files
```shell
gtfFilter -m whiteT -l <common.txt> <long_read.gtf> <common.gtf>
gtfFilter -m whiteT -l <long_read_unique.txt> <long_read_assembly.gtf> <long_read_unique.gtf>
gtfFilter -m whiteT -l <short_read_unique.txt> <short_read_assembly.gtf> <short_read_unique.gtf>
```

Lastly, gffcompare is used to compare the different assemblies to the reference and and among themselves.
```shell
gffcompare -r <annotation.gtf> -o Reference.vs.ShortRead <short_read_assembly.gtf> 
gffcompare -r <annotation.gtf> -o Reference.vs.ShortReadUnique <short_read_unique.gtf> 
gffcompare -r <annotation.gtf> -o Reference.vs.LongRead LongRead.gtf <long_read_assembly.gtf
gffcompare -r <annotation.gtf> -o Reference.vs.LongReadUnique <long_read_unique.gtf>
gffcompare -r <annotation.gtf> -o Reference.vs.CommonReads <common.gtf>
gffcompare -r <annotation.gtf> -o Reference.vs.MixedRead <mixed_assembly.gtf>
gffcompare -r <mixed_assembly.gtf> -o MixedRead.vs.Concatenated <concatenated.gtf>
```



