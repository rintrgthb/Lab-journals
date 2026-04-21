wget http://augustus.gobics.de/binaries/scripts/getAnnoFasta.pl

perl getAnnoFasta.pl --seqfile=GCA_001949185.1_Rvar_4.0_genomic.fna augustus.whole.gff

grep ">" augustus.whole.aa | wc -l
   16435

mv augustus.whole.aa Rvar_proteins.faa

BLAST database creation
makeblastdb -in Rvar_proteins.faa -dbtype prot -out rvar_proteins_db

blastp -db rvar_proteins_db \
       -query peptides.fa \   
       -outfmt "6 qseqid sseqid evalue qcovs pident stitle" \
       -evalue 1e-3 -qcov_hsp_perc 50 -out chromatin_matches.tsv \
       -num_threads 8

# Уникальные ID белков-хитов (первые 2 колонки)
cut -f2 chromatin_matches.tsv | sort | uniq > chromatin_protein_ids.txt

Извлечение полных последовательностей
samtools faidx Rvar_proteins.faa - индексация протеинов
xargs samtools faidx Rvar_proteins.faa < chromatin_protein_ids.txt > chromatin_proteins.faa
