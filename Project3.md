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

grep "stx" prokka_out/ecoliX.gff
