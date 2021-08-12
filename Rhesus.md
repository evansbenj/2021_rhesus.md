# Rhesus genomez

Path
```
/home/ben/projects/rrg-ben/ben/done_rhesus_genomes
```

Genomez
```
SAMD00215308 (DRS139837) M. fasc. aureus
fastq-dump -I --split-files DRR219369

SAMD00215309 (DRS139838) M. fasc. aureus
fastq-dump -I --split-files DRR219370

M. assamensis SRR2981114

M. thibetana SRR1024051

M. fasc Mauritius male SRR1564766	

M. fasc Thai DRR219371	

SRA023855_M_fasc_Viet
```

# download using NCBI tools
```
#!/bin/sh
#SBATCH --job-name=fastq_dump
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=18:00:00
#SBATCH --mem=2gb
#SBATCH --output=fastq_dump.%J.out
#SBATCH --error=fastq_dump.%J.err
#SBATCH --account=def-ben

# run by passing an argument like this (in the directory with the files)
# sbatch 2020_fastq_dump.sh SSR_number

module load StdEnv/2020  gcc/9.3.0 sra-toolkit/2.10.8

fastq-dump -I --split-files ${1}

module load nixpkgs/16.09  intel/2016.4 tabix
bgzip -c ${1}_1.fastq > ${1}_1.fastq.gz
bgzip -c ${1}_2.fastq > ${1}_2.fastq.gz
```
# Compress
```
#!/bin/sh
#SBATCH --job-name=fastq_dump
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=8:00:00
#SBATCH --mem=2gb
#SBATCH --output=fastq_dump.%J.out
#SBATCH --error=fastq_dump.%J.err
#SBATCH --account=def-ben

# run by passing an argument like this (in the directory with the files)
# sbatch 2020_fastq_dump.sh SSR_number

module load StdEnv/2020  gcc/9.3.0 sra-toolkit/2.10.8

#fastq-dump -I --split-files ${1}

module load nixpkgs/16.09  intel/2016.4 tabix
bgzip -c ${1}_1.fastq > ${1}_1.fastq.gz
bgzip -c ${1}_2.fastq > ${1}_2.fastq.gz
```
...and then delete non compressed files

# Trim using trimmomatic
```
#!/bin/sh
#SBATCH --job-name=trimmomatic
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=96:00:00
#SBATCH --mem=8gb
#SBATCH --output=trimmomatic.%J.out
#SBATCH --error=trimmomatic.%J.err
#SBATCH --account=def-ben

# run by passing a directory argument like this
# sbatch ./2020_trimmomatic.sh ../raw_data/plate1


module load StdEnv/2020
module load trimmomatic/0.39

#v=1
#  Always use for-loop, prefix glob, check if exists file.
for file in $1/*_1.fastq.gz; do         # Use ./* ... NEVER bare *
  if [ -e "$file" ] ; then   # Check whether file exists.
    echo java -jar $EBROOTTRIMMOMATIC/trimmomatic-0.39.jar PE ${file::-11}_1.fastq.gz ${file::-11}_2.fastq.gz ${file::-11}_tr
im.R1.fq.gz ${file::-11}_trim.R1_single.fq.gz ${file::-11}_trim.R2.fq.gz ${file::-11}_trim.R2_single.fq.gz ILLUMINACLIP:TruSe
q2_and_3-PE-2.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
	java -jar $EBROOTTRIMMOMATIC/trimmomatic-0.39.jar PE ${file::-11}_1.fastq.gz ${file::-11}_2.fastq.gz ${file::-11}_tri
m.R1.fq.gz ${file::-11}_trim.R1_single.fq.gz ${file::-11}_trim.R2.fq.gz ${file::-11}_trim.R2_single.fq.gz ILLUMINACLIP:TruSeq
2_and_3-PE-2.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
  fi
done 
```

# Map to ref
```
sbatch 2021_bwa_samtools_map_to_ref.sh /home/ben/projects/rrg-ben/ben/2021_rhemac_v10/rheMac10.fa ../data/SRA023855_M_fasc_Viet
```
# add readgroups
```

```
# Genotype each individual
```
sbatch 2021_HaplotypeCaller.sh ../../2021_rhemac_v10/rheMac10.fa ../data/DRR219371_M_fasc_Thai/ chr1
```
# Combine gvcfs

# VariantFiltration

# SelectVariants
