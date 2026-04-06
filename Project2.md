# The list of all the commands used during the execution of Project2 "What causes antibiotic resistance?"
Once you have downloaded the data you need for analysis (the reference genome and the sequences being studied), you can start by examining data quality using fastQC:

conda install -c bioconda fastqc
fastqc -o . amp_res_1.fastq amp_res_2.fastq

As a result of executing the command, you will receive HTML files containing all the information about the data quality.

My data had low per base sequence quality, low per tile sequence quality, unevenness at the beginning of the sequence of per base sequence content, so I decided to use Trimmomatic to cut out all imperfections:

conda install -c bioconda trimmomatic
trimmomatic PE \
  -threads 4 \
  amp_res_1.fastq amp_res_2.fastq \
  forward_paired.fastq forward_unpaired.fastq \
  reverse_paired.fastq reverse_unpaired.fastq \
  LEADING:20 TRAILING:20 SLIDINGWINDOW:10:20 MINLEN:20

Next, I checked the quality of the data again and I was satisfied with it:

astqc forward_paired.fastq reverse_paired.fastq






