# Genome-Assembly

## TXVA and TXEA isolates

### Filter reads using trimmomatic
####Installation in lab Linux (v0.39)
$ conda install -c bioconda trimmomatic

#### Make adapter file
New adapters.fa file was created by combining NexteraPE-PE.fa, TruSeq2-PE.fa, TruSeq3-PE.fa and TruSeq3-PE-2.fa from Trimmomatic and adapters.fa from bbmap.

####Running quality filtering
trimmomatic PE forward_read.fastq.gz reverse_read.fastq.gz forward_read_paired.fastq.gz forward_read_unpaired.fastq.gz reverse_read_paired.fastq.gz reverse_read_unpaired.fastq.gz ILLUMINACLIP:adapters.fa:2:30:10 LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:36 -threads 8 

(base) woo-suk@Chang-lab:/media/woo-suk/Data/C_peterson/TXEA/trimmomatic_filtered$ trimmomatic PE ../TXEAR1.fastq.gz ../TXEAR2.fastq.gz TXEA_R1_paired_trimmed.fastq.gz TXEA_R1_unpaired_trimmed.fastq.gz TXEA_R2_paired_trimmed.fastq.gz TXEA_R2_unpaired_trimmed.fastq.gz ILLUMINACLIP:adapters.fa:2:30:10 LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:36 -threads 8


### Assembly 1 using SPAdes
#### Read Error Correction:
spades.py -1 forward_read.fastq.gz -2 reverse_read.fastq.gz -o spades_error_corrected_reads -t <no. of threads> -m <RAM> --only-error-correction

#### Assembly
spades.py -1 forward_read_corrected.fastq.gz -2 reverse_read_corrected.fastq.gz -o spades_careful_assembly -t <no. of threads> --only-assembler --careful

### Assembly 2 using SPAdes - Recommendation for assembly of isolated bacterial genomes sequenced with reasonable coverage (~100x and more)
  1. Trimmomatic or any read trimming tool to remove adapters and low quality reads.
  2. Run with --only-assembler option (do not use internal read-correction)
  3. Run in careful mode (use --careful)

  
  
### Assembly using SKESA
  skesa --reads R1.fastq,R2.fastq --cores 8 --memory 50 > sekesa.fa
  
### Assembly using SAUTE - Sequence Assembly Using Target Enrichment  
