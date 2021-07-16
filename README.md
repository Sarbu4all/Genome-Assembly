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


### Filter reads using bbduk
(base) woo-suk@Chang-lab:/media/woo-suk/Data/C_peterson/TXEA$ '/home/woo-suk/Downloads/bbmap/bbduk.sh' in1=TXEAR1.fastq.gz in2=TXEAR2.fastq.gz out1=TXEA_R1_trimq15_maq20_minlen50.fastq.gz out2=TXEA_R2_trimq15_maq20_minlen50.fastq.gz ref='/home/woo-suk/Downloads/bbmap/resources/adapters.fa' ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=rl trimq=15 ftl=5 ftr=0 ftm=5 maq=20 minlen=50

(base) woo-suk@Chang-lab:/media/woo-suk/Data/C_peterson/TXVA$ /home/woo-suk/Downloads/bbmap/bbduk.sh in1=TXVAR1.fastq.gz in2=TXVAR2.fastq.gz out1=TXVA_R1_trimq15_maq20_minlen50.fastq.gz out2=TXVA_R2_trimq15_maq20_minlen50.fastq.gz ref='/home/woo-suk/Downloads/bbmap/resources/adapters.fa' ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=rl trimq=15 ftl=5 ftr=0 ftm=5 maq=20 minlen=50

### Assembly 1 using SPAdes
#### Read Error Correction:
spades.py -1 forward_read.fastq.gz -2 reverse_read.fastq.gz -o spades_error_corrected_reads -t <no. of threads> -m <RAM> --only-error-correction

#### Assembly
spades.py -1 forward_read_corrected.fastq.gz -2 reverse_read_corrected.fastq.gz -o spades_careful_assembly -t <no. of threads> --only-assembler --careful


(base) woo-suk@Chang-lab:/media/woo-suk/Data/C_peterson/TXVA$ ~/Downloads/SPAdes-3.15.2-Linux/bin/spades.py -1 TXVA_R1_trimq15_maq20_minlen50.fastq.gz -2 TXVA_R2_trimq15_maq20_minlen50.fastq.gz -o spades_assembly --isolate

  
#### Assembly 2 using SPAdes
  ##### Recommendation for assembly of isolated bacterial genomes sequenced with reasonable coverage (~100x and more)
  1. Trimmomatic or any read trimming tool to remove adapters and low quality reads.
  2. Run with --only-assembler option (do not use internal read-correction)
  3. Run in careful mode (use --careful)

  
  
### Assembly using SKESA
  
Downloaded SKESA
To Install; (but got the error)  

  ~/SKESA$ make -f Makefile.nongs
c++ -std=c++11 -c -o skesa.o skesa.cpp -D NO_NGS -Wall -Wno-format-y2k  -pthread -fPIC -O3 -finline-functions -fstrict-aliasing -fomit-frame-pointer -msse4.2 
skesa.cpp:27:10: fatal error: boost/program_options.hpp: No such file or directory
 #include <boost/program_options.hpp>
          ^~~~~~~~~~~~~~~~~~~~~~~~~~~
compilation terminated.
make: *** [Makefile.nongs:64: skesa.o] Error 1

To install BOOST:
  ~/SKESA$ sudo apt-get install libboost-all-dev
 
Again:
  ~/SKESA$ sudo apt-get install libboost-all-dev (this also didn't work)
  
Finally:
   conda install -c bioconda skesa (worked!)

##### Running assembly:
  skesa --reads R1.fastq,R2.fastq --cores 8 --memory 50 --contigs_out sekesa_contigs.fa
  
  (base) woo-suk@Chang-lab:/media/woo-suk/Data/C_peterson/TXEA$ skesa --reads TXEAtrimq201.fastq.gz,TXEAtrimq202.fastq.gz --contigs_out skesa_contigs.fa

### Assembly using SAUTE - Sequence Assembly Using Target Enrichment  
  (base) woo-suk@Chang-lab:/media/woo-suk/Data/C_peterson/TXEA$ ~/SKESA/saute --reads TXEA_R1_trimq15_maq20_minlen50.fastq.gz,TXEA_R2_trimq15_maq20_minlen50.fastq.gz  --targets ~/Downloads/Bradyrhizobium_diazoefficience_USDA110_ref_genome.fna.gz --gfa saute_assembly/graph.gfa --all_variants saute_assembly/assembly.fa

  SAUTE assembly did not work. Did not produce any contigs.
  
### Check assembly with quast:

  python ~/quast-5.0.2/quast.py -o quast_results_spades -r '/media/woo-suk/Data/C_peterson/reference_genome/GCF_001642675.1_ASM164267v1_genomic.fna.gz' -g '/media/woo-suk/Data/C_peterson/reference_genome/GCF_001642675.1_ASM164267v1_genomic.gff.gz' -l "spades, skesa" scaffolds.fasta skesa_contigs.fa --glimmer

  (base) woo-suk@Chang-lab:/media/woo-suk/Data/C_peterson/TXVA$ python ~/quast-5.0.2/quast.py -o quast_results_spades -r '/media/woo-suk/Data/C_peterson/reference_genome/GCF_001642675.1_ASM164267v1_genomic.fna.gz' -g '/media/woo-suk/Data/C_peterson/reference_genome/GCF_001642675.1_ASM164267v1_genomic.gff.gz' -l "spd15-15-50, spd15-15-100, spd15-20-50, spd15-20-100, spd20-20-50, spd20-20-100" bbduk_trimq15_maq15_minlen50/spades_assembly/scaffolds.fasta bbduk_trimq15_maq15_minlen100/spades_assembly/scaffolds.fasta bbduk_trimq15_maq20_minlen50/spades_assembly/scaffolds.fasta bbduk_trimq15_maq20_minlen100/spades_assembly/scaffolds.fasta bbduk_trimq20_maq20_minlen50/spades_assembly/scaffolds.fasta bbduk_trimq20_maq20_minlen100/spades_assembly/scaffolds.fasta --glimmer
