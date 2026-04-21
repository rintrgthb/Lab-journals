# The list of all the commands used during the execution of Project4: Tardigrades: from genestealers to space marines

**The project aimed at understanding the mechanisms of effective repair of DNA damage in tardigrade of the species Ramazzottius varieornatus, which are resistant to ionizing radiation**

For this project I used a sequence of the Ramazzottius varieornatus, the YOKOZUNA-1 strain (sequenced in the University of Tokyo and named after the highest rank in professional sumo). I downloaded assembled genome via link (http://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/949/185/GCA_001949185.1_Rvar_4.0/GCA_001949185.1_Rvar_4.0_genomic.fna.gz). I also downloaded precomputed AUGUSTUS protein annotation (https://drive.google.com/file/d/1hCEywBlqNzTrIpQsZTVuZk1S9qKzqQAq/view?usp=sharing, https://drive.google.com/file/d/12ShwrgLkvJIYQV2p1UlXklmxSOOxyxj4/view?usp=sharing).

**Predicted protein sequences were extracted from the GFF file** using the getAnnoFasta.pl script:

wget http://augustus.gobics.de/binaries/scripts/getAnnoFasta.pl

perl getAnnoFasta.pl --seqfile=GCA_001949185.1_Rvar_4.0_genomic.fna augustus.whole.gff

I **checked the number of proteins**:

grep ">" augustus.whole.aa | wc -l
     
      16435

Renamed the file for convenience:

mv augustus.whole.aa Rvar_proteins.faa

For a **local alignment-based search** I created a local database from my protein fasta file using BLAST+ (parameters: qcov_hsp_perc 30; evalue 0.1; outfmt 6):

makeblastdb -in Rvar_proteins.faa -dbtype prot -out rvar_proteins_db

blastp -db rvar_proteins_db \
       -query peptides.fa \   
       -outfmt "6 qseqid sseqid evalue qcovs pident stitle" \
       -evalue 0.1 -qcov_hsp_perc 30 \
       -out chromatin_matches_all.tsv -num_threads 8

After that I **extracted proteins of interest from the initial file**:

cut -f2 chromatin_matches.tsv | sort | uniq > chromatin_protein_ids.txt

samtools faidx Rvar_proteins.faa - индексация протеинов
xargs samtools faidx Rvar_proteins.faa < chromatin_protein_ids.txt > chromatin_proteins.faa

I **checked the number of proteins again**:

grep ">" chromatin_proteins_all.faa | wc -l
     
      13

# At this stage, work in the Terminal is completely finished and we are moving to online applications

I used WoLF PSORT (organism type = Animal) and TargetP Server (Organism group = Non-plant) to predict where these proteins are found in the cell based on their sequences. 

I used Protein BLAST for functional annotation of the chromatin proteins (“UniProtKB/Swiss-Prot” database). 

And finally I used HMMER (->search) to predict the function of the proteins in which orthologous sequences were not found in databases with Blast.

You can do the same and share your result, I created the summary table of expanded annotation, predicted Pfam domains, probable localization(s) according to WoLF PSORT and TargetP of nuclear and other proteins. You can find it in this repository **(my_summary.csv)** and compare your results with mine.

Also you can compare your results with the related articles (https://drive.google.com/open?id=1jwGSmhyapfd0ctUfVWxhh7qrQdfROmIE). 











