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

# then more filtering
```
vcftools --gzvcf maq_allsites_chr20.g.vcf.gz_chr20.vcf.gz_filtered.vcf.gz_filtered_removed.vcf.gz.recode_maxmissingcount_2.vcf.recode.vcf.gz --max-missing 0.5 --minQ 30 --recode --recode-INFO-all --out ./FandM_chr20_mm_0.5_minQ_30
```
# these files are here:
```
/home/ben/projects/rrg-ben/ben/2021_M_f_aurea/maq_vcfs_after_genotyping/filtered/input_for_admixfrog/final_input
```

# check for missing data
```
vcftools --vcf FandM_chr1_mm_0.5_minQ_30.recode.vcf --missing-indv
```
check output
```
cat out.imiss
```
This shows that each of the aurea samples have very little missing data (<1.5%) which is great!

So we will skip to another thinning step:
```
vcftools --vcf FandM_chr9_mm_0.5_minQ_30.recode.vcf --thin 500 --recode --recode-INFO-all --out FandM_chr9_mm_0.5_minQ_30_thinned
```
this gives us about the number of variable positions we want; e.g. for chr1:
```
more FandM_chr1_mm_0.5_minQ_30_thinned.log
```
```
After filtering, kept 393071 out of a possible 4550779 Sites
```

# compress and index
```
module load bcftools/1.9
bgzip -c FandM_chr20_mm_0.5_minQ_30_thinned.recode.vcf > FandM_chr20_mm_0.5_minQ_30_thinned.recode.vcf.gz
bcftools index FandM_chr20_mm_0.5_minQ_30_thinned.recode.vcf.gz
tabix -p vcf FandM_chr20_mm_0.5_minQ_30_thinned.recode.vcf.gz
```

# Now create a reference file for each individual. Before running admixfrog, do this:

```
module load scipy-stack/2019b
```

then for each sample do this:
```
admixfrog-ref [-h] --outfile OUTFILE [--states [STATES [STATES ...]]]
                     [--state-file STATE_FILE] [--cont-id CONT_ID]
                     [--ancestral ANCESTRAL]
                     [--random-read-samples [RANDOM_READ_SAMPLES [RANDOM_READ_SAMPLES ...]]]
                     [--vcf-ref FandM_chr01_mm_0.5_minQ_30_exclude_missingness_thinned.vcf.gz
```
This works make the ref file
```
/home/ben/.local/bin/admixfrog-ref --vcf FandM_chr20_mm_0.5_minQ_30_thinned.recode.vcf.gz --out FandM_chr20_mm_0.5_minQ_30_thinned.ref.xz --states AUR FAS ASS --pop-file pops.yaml 
```
where the pops.yaml file looks like this:
```
AUR:
    - DRR219369_trim_sorted.bam
    - DRR219370_trim_sorted.bam
FAS:
    - DRR219371_trim_sorted.bam
    - SRA023855_trim_sorted.bam
    - SRR1564766_trim_sorted.bam
ASS:
    - SRR2981114_trim_sorted.bam
THI:
    - SRR1024051_trim_sorted.bam
```
