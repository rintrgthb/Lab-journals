wget http://augustus.gobics.de/binaries/scripts/getAnnoFasta.pl

perl getAnnoFasta.pl --seqfile=GCA_001949185.1_Rvar_4.0_genomic.fna augustus.whole.gff

grep ">" augustus.whole.aa | wc -l
   16435
