# The list of all the commands used during the execution of Project2 "What causes antibiotic resistance?"

**This project will focus on how to determine which genes have mutated in E. coli, causing it to become resistant to antibiotics.**

Once you have downloaded the data you need for analysis (the reference genome and the sequences being studied), you can start by **examining data quality using fastQC**:

conda install -c bioconda fastqc
fastqc -o . amp_res_1.fastq amp_res_2.fastq

As a result of executing the command, you will receive HTML files containing all the information about the data quality.

My data had low per base sequence quality, low per tile sequence quality, unevenness at the beginning of the sequence of per base sequence content, so I decided to **use Trimmomatic to cut out all imperfections**:

conda install -c bioconda trimmomatic
trimmomatic PE \
  -threads 4 \
  amp_res_1.fastq amp_res_2.fastq \
  forward_paired.fastq forward_unpaired.fastq \
  reverse_paired.fastq reverse_unpaired.fastq \
  LEADING:20 TRAILING:20 SLIDINGWINDOW:10:20 MINLEN:20

Next, I **checked the quality of the data again** and I was satisfied with it:

fastqc forward_paired.fastq reverse_paired.fastq

**Reference assembly**

We will perform the assembly using BWA, for this we will index the reference and collect forward and reverse reads into one SAM file:

bwa index GCF_000005845.2_ASM584v2_genomic.fna
bwa mem GCF_000005845.2_ASM584v2_genomic.fna forward_paired.fastq reverse_paired.fastq > alignment.sam

Next, you need to **convert the SAM file to BAM format**:

samtools view -b -S alignment.sam | samtools sort -o alignment_sorted.bam

**Let's look at the statistics of our assembly!**

samtools index alignment_sorted.bam
samtools flagstat alignment_sorted.bam

I got a good build (99% fit well), so I continued researching and started making an annotation.

**Annotation**

I used Mpileup from samtools **to count the number of bases that do not match the reference genome and to understand how many reads have a mutation at the same position**:

samtools mpileup -f GCF_000005845.2_ASM584v2_genomic.fna alignment_sorted.bam > my.mpileup

And then I produced the **SNP calling** using VarScan:

conda install varscan
varscan mpileup2snp -h
varscan mpileup2snp my.mpileup --min-var-freq 0.50 --variants --output-vcf 1 > VarScan_results.vcf

*Reading input from my.mpileup*
*4641343 bases in pileup file*
*9 variant positions (6 SNP, 3 indel)*
*0 were failed by the strand-filter*
*6 variant positions reported (6 SNP, 0 indel)*

**The SNP annotation** was done using snpEff:

conda install snpEff

We need to create empty text file snpEff.config, and add just one string:
k12.genome : ecoli_K12

echo "k12.genome : ecoli_K12" > snpEff.config

You also need to download the sequence and annotation of our reference, to build a custom database (https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.gbff.gz)

**Create database and annotate**:

snpEff build -genbank -v k12
snpEff ann k12 VarScan_results.vcf > VarScan_results_annotated.vcf

And we look at which genes have undergone mutations, as well as the type of these mutations:

% grep -v "^#" VarScan_results_annotated.vcf | cut -f8 | grep -o "ANN=[^;]*"

**You will see smth like that:**

NC_000913.3	4390754	.	G	T	.	PASS ADP=15;WT=0;HET=0;HOM=1;NC=0;ANN=T|synonymous_variant|LOW|rsgA|b4161|transcript|b4161|protein_coding|1/1|c.756C>A|p.Ala252Ala|756/1053|756/1053|252/350||WARNING_TRANSCRIPT_NO_START_CODON,T|upstream_gene_variant|MODIFIER|mscM|b4159|transcript|b4159|protein_coding||c.-1384C>A|||||1384|WARNING_TRANSCRIPT_NO_START_CODON,T|upstream_gene_variant|MODIFIER|psd|b4160|transcript|b4160|protein_coding||c.-394C>A|||||394|WARNING_TRANSCRIPT_NO_START_CODON,T|upstream_gene_variant|MODIFIER|orn|b4162|transcript|b4162|protein_coding||c.-850G>T|||||850|,T|upstream_gene_variant|MODIFIER|yjeV|b4670|transcript|b4670|protein_coding||c.-2138G>T|||||2138|,T|upstream_gene_variant|MODIFIER|nnr|b4167|transcript|b4167|protein_coding||c.-3312G>T|||||3312|,T|upstream_gene_variant|MODIFIER|tsaE|b4168|transcript|b4168|protein_coding||c.-4831G>T|||||4831|,T|downstream_gene_variant|MODIFIER|yjeO|b4158|transcript|b4158|protein_coding||c.*4736G>T|||||4736|,T|downstream_gene_variant|MODIFIER|queG|b4166|transcript|b4166|protein_coding||c.*2174C>A|||||2174|	GT:GQ:SDP:DP:RD:AD:FREQ:PVAL:RBQ:ABQ:RDF:RDR:ADF:ADR	1/1:81:16:15:0:15:100%:6,4467E-9:0:36:0:0:8:7

**Now think about what mutation led to resistance**

# Great job!












