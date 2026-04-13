fastqc SRR292678sub_S1_L001_R1_001.fastq SRR292678sub_S1_L001_R2_001.fastq

jellyfish count -m 31 -C -s 100M -t 4 -o srr292678_k31.jf SRR292678sub_S1_L001_R1_001.fastq SRR292678sub_S1_L001_R2_001.fastq

jellyfish histo srr292678_k31.jf > srr292678_k31.histo

conda install -c bioconda sra-tools
conda install -c bioconda spades

spades.py \
  -1 SRR292678_1.fastq \
  -2 SRR292678_2.fastq \
  -t 4 \
  --isolate \
  --memory 16


conda install -c bioconda quast


quast.py contigs.fasta -o quast_report --threads 4



quast.py scaffolds.fasta -o quast_report --threads 4

spades.py \
  -1 SRR292678sub_S1_L001_R1_001.fastq \
  -2 SRR292678sub_S1_L001_R2_001.fastq \
  --pacbio SRR1980037*.fastq \
  -o hybrid_assembly \
  -t 4 \
  --memory 16

quast.py scaffolds_2.fasta -o quast_report_2 --threads 4

conda install -c bioconda prokka

prokka scaffolds_2.fasta \       
  --outdir ecoli_annotated \
  --prefix Ecoli_hybrid \
  --cpus 4 \
  --kingdom Bacteria \
  --genus Escherichia \
  --species coli

conda install -c bioconda barrnap
barrnap --updatedb
barrnap scaffolds_2.fasta --kingdom bac --outseq ecoli_16S.fasta --updatedb

curl -o "ecoli_16S.fasta" "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nucleotide&id=NC_000913.3&rettype=fasta&retmode=text&seq_start=1516&seq_stop=3034"

makeblastdb -in scaffolds_2.fasta -dbtype nucl -out scaffolds_db
blastn -db scaffolds_db -query ecoli_16S.fasta -outfmt 6 -evalue 1e-50 -out blast_16S.txt
head -1 blast_16S.txt
ecoli % awk 'NR==FNR{lines[$1]++; next} /^>/{if(lines[$1]) print;}' blast_16S.txt scaffolds_2.fasta > ecoliX_16S_raw.fna
blastn -db scaffolds_db -query ecoli_16S.fasta -outfmt 6 -evalue 1e-20 -perc_identity 80 -out blast_16S_soft.txt
samtools faidx scaffolds_2.fasta NODE_1_length_2579755_cov_79.346722:2257655-2259173 > ecoliX_16S.fna

grep "stx" prokka_out/ecoliX.gff

grep -A5 -B5 "stx[A-B]" prokka_out/ecoliX.gff
