# Vcf files after genotype filtering
```
/home/ben/projects/rrg-ben/ben/2021_M_f_aurea/maq_vcfs_after_genotyping/filtered
```
# use bgzip and tabix to compress and index
```
module load tabix
bgzip -c vcffile > vcffile.gz
tabix -p vcf vcffile.gz
```

# then I used vcftools to thin:
```
module load vcftools
vcftools --gzvcf maq_allsites_chr1.g.vcf.gz_chr1.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz --max-missing-count 2 --minQ 30 --recode --recode-INFO-all --out ./maq_allsites_chr1.g.vcf.gz_chr1.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz.recode_maxmissingcount_2.vcf
```
# use bgzip and tabix to compress and index
```
module load tabix
bgzip -c vcffile > vcffile.gz
tabix -p vcf vcffile.gz
```
# these input files are here:
```
/home/ben/projects/rrg-ben/ben/2021_M_f_aurea/maq_vcfs_after_genotyping/filtered/input_for_admixfrog
```
