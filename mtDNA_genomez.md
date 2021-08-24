# Trying to map reads, extract mapped reads, and do denovo assembly.

# Trim ends
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
for file in $1/*_1.fq.gz; do         # Use ./* ... NEVER bare *
  if [ -e "$file" ] ; then   # Check whether file exists.
    echo java -jar $EBROOTTRIMMOMATIC/trimmomatic-0.39.jar PE ${file::-8}_1.fq.g
z ${file::-8}_2.fq.gz ${file::-8}_trim.R1.fq.gz ${file::-8}_trim.R1_single.fq.gz
 ${file::-8}_trim.R2.fq.gz ${file::-8}_trim.R2_single.fq.gz ILLUMINACLIP:TruSeq2
_and_3-PE-2.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
	java -jar $EBROOTTRIMMOMATIC/trimmomatic-0.39.jar PE ${file::-8}_1.fq.gz
 ${file::-8}_2.fq.gz ${file::-8}_trim.R1.fq.gz ${file::-8}_trim.R1_single.fq.gz 
${file::-8}_trim.R2.fq.gz ${file::-8}_trim.R2_single.fq.gz ILLUMINACLIP:TruSeq2_
and_3-PE-2.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
  fi
done 
```
# Map reads
```
#!/bin/sh
#SBATCH --job-name=bwa_align
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=4
#SBATCH --time=96:00:00
#SBATCH --mem=32gb
#SBATCH --output=bwa_align.%J.out
#SBATCH --error=bwa_align.%J.err
#SBATCH --account=def-ben

# run by passing an argument like this (in the directory with the files)
# sbatch 2020_align_paired_fq_to_ref.sh pathandname_of_ref path_to_paired_fq_fil
ez
# sbatch 2020_align_paired_fq_to_ref.sh ../../2020_XL_v9.2_refgenome/XENLA_9.2_g
enome.fa.gz pathtofqfilez

module load bwa/0.7.17
module load samtools/1.10


for file in ${2}/*_trim.R1.fq.gz ; do         # Use ./* ... NEVER bare *    
    if [ -e "$file" ] ; then   # Check whether file exists.
	echo bwa mem ${1} ${file::-14}_trim.R1.fq.gz ${file::-14}_trim.R2.fq.gz 
-t 16 | samtools view -Shu - | samtools sort - -o ${file::-14}_sorted.bam
	bwa mem ${1} ${file::-14}_trim.R1.fq.gz ${file::-14}_trim.R1.fq.gz -t 16
 | samtools view -Shu - | samtools sort - -o ${file::-8}_sorted.bam
	samtools index ${file::-14}_sorted.bam
  fi
done
```

#Extract mapped reads from bam files 
Using bam2fastq (https://github.com/jts/bam2fastq)
```
./bam2fastq --aligned --force --strict -o mapped#.fq ../SRR9611188_trim._sorted.bam 
```
