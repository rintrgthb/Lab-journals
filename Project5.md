#The list of all the commands used during the execution of Project
conda install -c bioconda plink


plink --23file 23_and_me_input.txt --recode vcf --out snps_clean --output-chr MT --snps-only just-acgt

conda install -c bioconda bcftools

bcftools view -i 'GT!="0/0"' snps_clean.vcf > output.vcf

grep -E "(rs12913832|rs1545397|rs1426654|rs16891982|rs885479|rs6119471|rs12203592|rs12896399)" output.vcf

awk -F'\t' '$38!="-"' VEP_results.txt | grep risk_factor | cut -f 1-3 | sort | uniq > risk_snps.tsv
