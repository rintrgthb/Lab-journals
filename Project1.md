# The list of all the commands used during the execution of Project1 "Discovery of SARS-CoV-2 virus using meta-transcriptomic sequencing of bronchoalveolar lavage fluid"


At first I downloaded the file with the sequence from the link (https://drive.google.com/file/d/1PU6gQiF2CvxYhmWxVCWuESnG1XCZvibf/view?usp=drive_link). 

***Checking the quality of the assembly using QUAST***

conda install quast -c bioconda -c conda-forge

quast spades_scaffolds.fasta -o quast_out

After running these commands in the terminal, you will get several files in the folder quast_out, don't worry, you need only txt report

***Selection of contigs greater than 3000 bp***

samtools faidx spades_scaffolds.fasta

cat spades_scaffolds.fasta.fai

more spades_scaffolds.fasta.fai | awk '$2 >= 3000 {print $1}' | \
    xargs samtools faidx spades_scaffolds.fasta > 3000.fa

These commands help index the source file, check its integrity, and select contigs longer than 3000 bp (because virus genome is around 3000-5000 bp).

***BLAST time!***

Upload the file we received from the previous step (3000.fa) to BLAST, wait a minute and get your result! Here you can find the closiest sequence to yours.

***Annotation with PROKKA***

I found out that my largest contig is close to the Bat SARS-like coronavirus, so I used samtools one more time to single it out specifically.

samtools faidx spades_scaffolds.fasta NODE_1_length_29907_cov_150.822528  > contig.fa

And when used prokka - **you should add --kingdom Viruses, because default is Bacteria**

prokka contig.fa --kingdom Viruses --outdir contig_prokka --prefix contig --cpus 8 --force

After running these commands in the terminal, you will get several files in the folder contig_prokka, don't worry, you need only tsv file. Here you can check the CDS that was annotated, check your results with the reference and etc.

Thanks for reading it till the end, hope you recognized your virus **<3**








