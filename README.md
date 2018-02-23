**README**

**Arkansas Workshop, Sept 11-12th, 2017**

**National Center for Genome Analysis Support (NCGAS)**
The National Center for Genome Analysis Support is based in Indiana but caters to a national audience with support from the NSF. We provide computational resources and support for genomics, transcriptomics, and meta- projects. For more information goto https://ncgas.org/

**Jetstream**
Jetstream is a cloud computing resource supported by the National Science Foundation as a component of the Extreme Science and Engineering Development Environment. Jetstream is the first STEM-research cloud with an application base that extends across all areas of NSF-supported activity. For more information goto https://use.jetstream-cloud.org/.

Jetstream instance we will use for the workshop https://use.jetstream-cloud.org/application/images/496

**Software installed in the Jetstream instance** 

  1.	FastQC - https://www.bioinformatics.babraham.ac.uk/projects/fastqc/ 
  2.	Trimmomatic - http://www.usadellab.org/cms/index.php?page=trimmomatic 
  3.	Prinseq - http://prinseq.sourceforge.net/ 
  4.	SPAdes assembler - http://bioinf.spbau.ru/spades 
  5.	Trinity RNAseq assembler - https://github.com/trinityrnaseq/trinityrnaseq/wiki 
  6.	MegaHit assembler - https://github.com/voutcn/megahit 
  7.	BUSCO - http://busco.ezlab.org
  8.	QUAST -  http://bioinf.spbau.ru/quast
  9.	Bowtie2 - http://bowtie-bio.sourceforge.net/bowtie2 
  10.	Samtools - http://www.htslib.org/ 
  11.	MetaBat - https://bitbucket.org/berkeleylab/metabat 
  12.	CheckM - https://github.com/Ecogenomics/CheckM/wiki/Genome-Quality-Commands 
  13.	R - https://www.r-project.org/ 
  14.	Python 2.7- https://www.python.org/downloads/ 

##Workshop Course
**Part 1- Data collection**
*Take a look at the “DataCollection.pptx”*

**Part 2- Data management**
*Take a look at the “Intro.pptx”*

*Organize your directory and file structure
      `mkdir Workshop #make a directory called “Workshop”`
      `cd Workshop`
*Keep README files, with information on date, a small note about what is changing, and an overview of the project, where I got data    from, etc
      `nano README`
*Recoding all the major steps of the analysis using a shell script 
        `nano runTrinity.sh` # to start writing the shell script
        `Trinity –seqTypes… <command>` #add the command
        `bash runTrinity.sh` #to run the shell script
*Long term plans for your data 
      _Where will you keep the final results? 
      _How often will you back up?
      _How will you fight the urge to hoard data?
      _What intermediates can you live without?

**Part 3- RNA-seq analysis**
*Take a look at the “Intro.pptx” and “rnaseq_petra.pptx”*

*FastQC - https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
    To take a quick look at the quality of the sequences 
    `fastqc /opt/datasets/Sp_ds.left.fq` #fastqc /path/to/input files
*Trimmomatic- http://www.usadellab.org/cms/index.php?page=trimmomatic
    Filter low quality sequences
    `java -Xmx15g -jar ~/Ark2017/Trimmomatic-0.36/trimmomatic-0.36.jar PE /opt/datasets/Sp_ds.left.fq /opt/datasets/Sp_ds.right.fq      trimmed_ds_left.fq unpaired_trimmed_ds_left.fq trimmed_ds_right.fq unpaired_trimmed_ds_right.fq SLIDINGWINDOW:15:30 MINLEN:50`
*Trinity assembly- https://github.com/trinityrnaseq/trinityrnaseq/wiki
    RNA-seq assembler 
    `Trinity --seqType fq --max_memory 12G -–left trimmed_ds_left.fq,trimmed_hs_left.fq -–right trimmed_ds_right.fq,trimmed_hs_right.fq`
*Assembly quality- Busco (http://busco.ezlab.org )

*How to run all these steps using a galaxy server 
  Galaxy is a web-based tool for analyzing data (Example https://galaxy.ncgas-trinity.indiana.edu)
    _It provides a set of tools that one can apply to the data
    _It provides a trail of analyses one has run
    _It allows for sharing of data sets, methods, and workflows
For more information and material on galaxy https://ncgas.org/training.php 

**Part 4- Metagenomics analysis**
*Datasets for workshop 
  `/opt/Metagenomics_Data` #filepath to find all input datasets 
   Reads- /opt/Metagenomics_Data/reads
    _Coral1_10000.fna  
    _Coral2_10000.fna
    _Coral3_10000.fna
    _Coral4_10000.fna
    _concat.fasta  (contains the reads from all the 4 samples)
*Pre-processing data- Prinseq http://prinseq.sourceforge.net/
    `export PATH=$PATH:/opt/prinseq-lite-0.20.4` # adding prinseq to path 
    `prinseq-lite.pl -verbose -fasta concat.fasta -stats_all` #checking quality of the input files
    `prinseq-lite.pl -verbose -fasta reads/concat.fasta -min_len 60 -derep 1234 -out_format 1 -out_good concat_prinseq_good.fasta -  out_bad concat_prinseq_bad.fasta` # command to trim the files 

*Assembly- SPAdes (http://bioinf.spbau.ru/spades) and Megahit (https://github.com/voutcn/megahit )
    `export PATH=$PATH:/opt/SPAdes-3.10.1-Linux/bin:/opt/megahit` # adding the scripts to path 
    `python spades.py --iontorrent -s concat.fasta --only-assembler -o spades_output` #spades command 
    `megahit contigs.fasta -o megahit_output` #megahit command 
*Assembly evaluation- QUAST (http://bioinf.spbau.ru/quast) and bowtie2 (http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml)
    `export PATH=$PATH:/opt/quast-4.5`
    `metaquast.py "contigs" -o "output“`
    `./bowtie-build "input fasta" "index-name`
    `./bowtie2 -x index -f fasta file -S file.sam  --al aligned_reads.fasta`
*Taxa identification- FOCUS (http://edwards.sdsu.edu/focus/) 
    `export PATH=$PATH:/opt/FOCUS_0.31_python_2.7.XX`
    `python focus.py -q "input file" -o "output file”`
*Metagenome binning- MetaBat (https://bitbucket.org/berkeleylab/metabat)
    `export PATH=$PATH:/opt/metabat`
    `samtools view -bS <file.sam> | samtools sort - file_sorted –o <sorted bam file>` 
    `jgi_summarize_bam_contig_depths --outputDepth  <depth.txt> *-sorted.bam`
    `metabat2 -i assembly.fasta -a depth.txt -o bins_dir/bin`
•	Checking quality of metagenome bins- CheckM (https://github.com/Ecogenomics/CheckM/wiki/Genome-Quality-Commands) 

Contacts
help@ncgas.org
