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
/home/ben/.local/bin/admixfrog-ref --vcf FandM_chr20_mm_0.5_minQ_30_thinned.recode.vcf.gz --out FandM_chr20_mm_0.5_minQ_30_thinned.ref.xz --states AUR FAS ASS --pop-file pops.yaml --chroms chr20
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

# Making Ref file
This worked well for making the ref file:
```
#!/bin/sh
#SBATCH --job-name=AF_ref
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=6:00:00
#SBATCH --mem=2gb
#SBATCH --output=AF_ref.%J.out
#SBATCH --error=AF_ref.%J.err
#SBATCH --account=def-ben

# sbatch admixfrog_make_refs.sh chrXX

# this needs to be done for each chr
module load  nixpkgs/16.09 scipy-stack/2019b


# make the ref file
/home/ben/.local/bin/admixfrog-ref --vcf FandM_${1}_mm_0.5_minQ_30_thinned.recode.vcf.gz --out FandM_${1}_mm_0.5_minQ_30_thinned.recod
e.vcf.gz.xz --states AUR FAS ASS THI --pop-file pops.yaml --chroms ${1}
```

# Make target files
```
#!/bin/sh
#SBATCH --job-name=AF_tar
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=8:00:00
#SBATCH --mem=2gb
#SBATCH --output=AF_tar.%J.out
#SBATCH --error=AF_tar.%J.err
#SBATCH --account=def-ben

module load  nixpkgs/16.09 scipy-stack/2019b

# this needs to be run for each chr

# make the target (input) file
/home/ben/.local/bin/admixfrog-bam --bam /home/ben/projects/rrg-ben/ben/2021_M_f_aurea/data/DRS139837_DRR219369_M_f_aureus/DRR219369_trim_sorted.bam_rg.bam --ref FandM_${1}_mm_0.5_minQ_30_thinned.rec
ode.vcf.gz.xz --out AUR_DRR219369.${1}.in.xz

/home/ben/.local/bin/admixfrog-bam --bam /home/ben/projects/rrg-ben/ben/2021_M_f_aurea/data/DRS139838_DRR219370_M_f_aureus/DRR219370_trim_sorted.bam_rg.bam --ref FandM_${1}_mm_0.5_minQ_30_thinned.rec
ode.vcf.gz.xz --out AUR_DRR219370.${1}.in.xz

/home/ben/.local/bin/admixfrog-bam --bam /home/ben/projects/rrg-ben/ben/2021_M_f_aurea/data/DRR219371_M_fasc_Thai/DRR219371_trim_sorted.bam_rg.bam --ref FandM_${1}_mm_0.5_minQ_30_thinned.recode.vcf.g
z.xz --out FAS_DRR219371_thai.${1}.in.xz

/home/ben/.local/bin/admixfrog-bam --bam /home/ben/projects/rrg-ben/ben/2021_M_f_aurea/data/SRA023855_M_fasc_Viet/SRA023855_trim_sorted.bam_rg.bam --ref FandM_${1}_mm_0.5_minQ_30_thinned.recode.vcf.g
z.xz --out FAS_SRA023855_viet.${1}.in.xz

/home/ben/.local/bin/admixfrog-bam --bam /home/ben/projects/rrg-ben/ben/2021_M_f_aurea/data/SRR1564766_M_fasc_Maurit/SRR1564766_trim_sorted.bam_rg.bam --ref FandM_${1}_mm_0.5_minQ_30_thinned.recode.v
cf.gz.xz --out FAS_SRR1564766_maurit.${1}.in.xz

/home/ben/.local/bin/admixfrog-bam --bam /home/ben/projects/rrg-ben/ben/2021_M_f_aurea/data/SRR2981114_M_ass/SRR2981114_trim_sorted.bam_rg.bam --ref FandM_${1}_mm_0.5_minQ_30_thinned.recode.vcf.gz.xz
 --out ASS_SRR2981114.${1}.in.xz

/home/ben/.local/bin/admixfrog-bam --bam /home/ben/projects/rrg-ben/ben/2021_M_f_aurea/data/SRR1024051_M_thib/SRR1024051_trim_sorted.bam_rg.bam --ref FandM_${1}_mm_0.5_minQ_30_thinned.recode.vcf.gz.x
z --out THI_SRR1024051.${1}.in.xz
```

run the analysis 
```
sbatch admixfrog_do_analysis_allchrs.sh nem_GumGum_female NEM SUM HEC
sbatch admixfrog_do_analysis_allchrs.sh nem_Ngsang_sumatra_female NEM SUM HEC
sbatch admixfrog_do_analysis_allchrs.sh nem_PM1206 NEM SUM HEC
sbatch admixfrog_do_analysis_allchrs.sh nem_PM664 NEM SUM HEC
sbatch admixfrog_do_analysis_allchrs.sh nem_PM665 NEM SUM HEC
sbatch admixfrog_do_analysis_allchrs.sh nem_Sukai_male NEM SUM HEC
```
