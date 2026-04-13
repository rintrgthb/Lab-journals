# The list of all the commands used during the execution of Project3 "E. coli outbreak investigation"

**This project will focus on how to do *de novo* assembly and determine which genes have appeared by horizontal gene transfer (HGT) in *E. coli*, causing it to become highly pathogenic**

Once you have downloaded the data you need for analysis (forward and reverse Illumina reads of *E.coli X* strain), you can start by **examining data quality using fastQC**:

fastqc SRR292678sub_S1_L001_R1_001.fastq SRR292678sub_S1_L001_R2_001.fastq

The quality of the reads was high, so I sефrted *de novo* assembly
I **assessed the K-mer and genome size profile using Jellyfish**.I used k-mer sizes of 31, counted k-mers and created a histogram:

conda install -c bioconda jellyfish

jellyfish count -m 31 -C -s 100M -t 4 -o srr292678_k31.jf SRR292678sub_S1_L001_R1_001.fastq SRR292678sub_S1_L001_R2_001.fastq

jellyfish histo srr292678_k31.jf > srr292678_k31.histo

The histogram can tell us about the genome length, heterozygosity and etc, so we can assess that we are looking at a bacterial genome and know its size before assembly to avoid errors.

**Assembling** was performed using **assembler SPAdes** in the paired-end mode, providing paired reads of *E. coli X* from the library SRR292678 (Illumina):

conda install -c bioconda sra-tools
conda install -c bioconda spades

spades.py \
  -1 SRR292678_1.fastq \
  -2 SRR292678_2.fastq \
  -t 4 \
  --isolate \
  --memory 16

After it I checked the **quality of the assembly using QUAST**:

conda install -c bioconda quast

quast.py scaffolds.fasta -o quast_report --threads 4

I also performed **assemply in hybrid mode** with the addition of PacBio long reads of the same sequence:

spades.py \
  -1 SRR292678sub_S1_L001_R1_001.fastq \
  -2 SRR292678sub_S1_L001_R2_001.fastq \
  --pacbio SRR1980037*.fastq \
  -o hybrid_assembly \
  -t 4 \
  --memory 16

And **checked the quality again**:

quast.py scaffolds_2.fasta -o quast_report_2 --threads 4

Tthere is an advantage in using long reads, since the library with a small insert size can resolve short repeats only. 

When assembled in hybrid mode, the performance improved, namely:
The number of contigs became smaller as shorter ones merged into longer ones;
The length of the largest contig became half the bacterial chromosome;
N50 increased ~ 9 times, which means that the length of the longest contig covering 50% of the genome has increased;
L50 became smaller, only 2 scaffolds cover 50% of the genome;
Total length preserved, which means we didn’t lose anything;
The number of unknown nucleotides decreased.

After that I performed **annotation using Prokka**:

conda install -c bioconda prokka

prokka scaffolds_2.fasta \       
  --outdir ecoli_prokka \
  --prefix Ecoli_hybrid \
  --cpus 4 \
  --kingdom Bacteria \
  --genus Escherichia \
  --species coli

*This step is optional*

You can **select 16S ribosomal RNA** and look at the strain that is most similar to the one being studied using **Barrnap**:

conda install -c bioconda barrnap

barrnap --updatedb

barrnap scaffolds_2.fasta --kingdom bac --outseq ecoli_16S.fasta --updatedb

But this tool have some troubles, so you can **try with Prokka** (as me). First, you need to **download the 16S rRNA database**:

curl -o ecoli_16S.fasta "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nucleotide&id=NC_000913.3&rettype=fasta&retmode=text&seq_start=1516&seq_stop=3034"

Next you have to **create BLAST database**:

makeblastdb -in scaffolds_2.fasta -dbtype nucl

blastn -db scaffolds_2 -query ecoli_16S.fasta -outfmt 6 -evalue 1e-50 -out blast_16S.txt
head -1 blast_16S.txt

After that you **extract scaffolds with 16S rRNA**:

awk 'NR==FNR{lines[$1]++; next} /^>/{if(lines[$1]) print; next} {print;}' blast_16S.txt scaffolds_2.fasta > ecoliX_16S_raw.fna

head -1 blast_16S.txt | awk '{print $1 "\t" $9-500 ":" $10+500}' > coords.txt
seqtk subseq scaffolds_2.fasta coords.txt > ecoliX_16S.fna

So you have 16S rRNA from your *E.coli X* strain. Whats next? You should **go to the NCBI BLAST homepage, upload your sequence and determine the closest strain to yours**. Halfway here!
I found out that the closest strain mine is *Escherichia coli 55989*. 

*This step is optional*

You can download Mauve and try to find some differences in 2 genomes, but also you can read about the strain you have found and find out what its pathogenicity is without using Mauve (because mine didnt work at all!)

I found out that the toxicity of the *E. coli 55989* was due to the presence of Shiga toxin genes in it. So I decided to **look for Shiga toxin genes in my *E. coli X***.

grep "stx" prokka_out/ecoliX.gff

And also I have to understand the changes, so I need to **find some genes near the toxin genes**:

grep -A5 -B5 "stx[A-B]" prokka_out/ecoliX.gff
