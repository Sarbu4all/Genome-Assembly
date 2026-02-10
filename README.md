# Genome-Assembly _Bradyrhizobium japonicum_ TXEA and TXVA isolates
These are the novel draught-tolerant _Bradyrhizobium japonicum_ strains isolated from the soybean rhizosphere grown in the draught-prone region of Texas. These strains cultured in LB media and isolated pure colony was sequenced.

### Filter reads using trimmomatic
#### Installation in lab Linux (v0.39)
`conda install -c bioconda trimmomatic`

#### Make adapter file and run quality filtering
New adapters.fa file was created by combining NexteraPE-PE.fa, TruSeq2-PE.fa, TruSeq3-PE.fa and TruSeq3-PE-2.fa from Trimmomatic and adapters.fa from bbmap.

`trimmomatic PE ../TXEAR1.fastq.gz ../TXEAR2.fastq.gz TXEA_R1_paired_trimmed.fastq.gz TXEA_R1_unpaired_trimmed.fastq.gz TXEA_R2_paired_trimmed.fastq.gz TXEA_R2_unpaired_trimmed.fastq.gz ILLUMINACLIP:adapters.fa:2:30:10 LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:36 -threads 8`

`trimmomatic PE ../TXVAR1.fastq.gz ../TXVAR2.fastq.gz TXVA_R1_paired_trimmed.fastq.gz TXVA_R1_unpaired_trimmed.fastq.gz TXVA_R2_paired_trimmed.fastq.gz TXVA_R2_unpaired_trimmed.fastq.gz ILLUMINACLIP:adapters.fa:2:30:10 LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:36 -threads 8`


### Filter reads using bbduk
`/home/bbmap/bbduk.sh in1=TXEAR1.fastq.gz in2=TXEAR2.fastq.gz out1=TXEA_R1_trimq15_maq20_minlen50.fastq.gz out2=TXEA_R2_trimq15_maq20_minlen50.fastq.gz ref='/home/woo-suk/Downloads/bbmap/resources/adapters.fa' ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=rl trimq=15 ftl=5 ftr=0 ftm=5 maq=20 minlen=50`

`/home/bbmap/bbduk.sh in1=TXVAR1.fastq.gz in2=TXVAR2.fastq.gz out1=TXVA_R1_trimq15_maq20_minlen50.fastq.gz out2=TXVA_R2_trimq15_maq20_minlen50.fastq.gz ref='/home/woo-suk/Downloads/bbmap/resources/adapters.fa' ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=rl trimq=15 ftl=5 ftr=0 ftm=5 maq=20 minlen=50`

### Assembly using SPAdes
  #### Recommendation for assembly of isolated bacterial genomes sequenced with reasonable coverage (~100x and more)
  1. Trimmomatic or any read trimming tool to remove adapters and low quality reads.
  2. Run with --only-assembler option (do not use internal read-correction)
  3. Run in careful mode (use --careful)

  I followed this process.

  `spades.py -1 TXEA_R1_trimq15_maq20_minlen50.fastq.gz -2 TXEA_R2_trimq15_maq20_minlen50.fastq.gz -o spades_careful_assembly -t 10 --only-assembler --careful`
  
  `spades.py -1 TXVA_R1_trimq15_maq20_minlen50.fastq.gz -2 TXVA_R2_trimq15_maq20_minlen50.fastq.gz -o spades_careful_assembly -t 10 --only-assembler --careful`
         
  **OR**
  
  1. We can just use | --isolate mode. It will give the same result.
  
  `spades.py -1 forward_read_corrected.fastq.gz -2 reverse_read_corrected.fastq.gz -o spades_careful_assembly -t <no. of threads> --isolate`
  
### Assembly using SKESA
  
Downloaded SKESA
To Install; (but got the error)  

  ~/SKESA$ make -f Makefile.nongs
c++ -std=c++11 -c -o skesa.o skesa.cpp -D NO_NGS -Wall -Wno-format-y2k  -pthread -fPIC -O3 -finline-functions -fstrict-aliasing -fomit-frame-pointer -msse4.2 
skesa.cpp:27:10: fatal error: boost/program_options.hpp: No such file or directory
 #include <boost/program_options.hpp>
          ^~~~~~~~~~~~~~~~~~~~~~~~~~~
Compilation terminated.
make: *** [Makefile.nongs:64: skesa.o] Error 1

To install BOOST:
  `~/SKESA$ sudo apt-get install libboost-all-dev`
 
Again:
  `~/SKESA$ sudo apt-get install libboost-all-dev`
  (this also didn't work)
  
Finally:
   `conda install -c bioconda skesa`
   (worked!)

##### Running assembly:
  skesa --reads R1.fastq,R2.fastq --cores 8 --memory 50 --contigs_out sekesa_contigs.fa
  
  `TXEA$ skesa --reads TXEAtrimq201.fastq.gz,TXEAtrimq202.fastq.gz --contigs_out skesa_contigs.fa`
  
  `TXVA$ skesa --reads TXVAtrimq201.fastq.gz,TXVAtrimq202.fastq.gz --contigs_out skesa_contigs.fa`

### Assembly using SAUTE - Sequence Assembly Using Target Enrichment  
  (base) woo-suk@Chang-lab:/media/woo-suk/Data/C_peterson/TXEA$ ~/SKESA/saute --reads TXEA_R1_trimq15_maq20_minlen50.fastq.gz,TXEA_R2_trimq15_maq20_minlen50.fastq.gz  --targets ~/Downloads/Bradyrhizobium_diazoefficience_USDA110_ref_genome.fna.gz --gfa saute_assembly/graph.gfa --all_variants saute_assembly/assembly.fa

  SAUTE assembly did not work. Did not produce any contigs.

  
### Check assembly with quast:

`~/TXEA$ python ~/quast-5.0.2/quast.py -o quast_results_spades -r '/media/woo-suk/Data/C_peterson/reference_genome/GCF_001642675.1_ASM164267v1_genomic.fna.gz' -g '/media/woo-suk/Data/C_peterson/reference_genome/GCF_001642675.1_ASM164267v1_genomic.gff.gz' -l "spd15-15-50, spd15-15-100, spd15-20-50, spd15-20-100, spd20-20-50, spd20-20-100" bbduk_trimq15_maq15_minlen50/spades_assembly/scaffolds.fasta bbduk_trimq15_maq15_minlen100/spades_assembly/scaffolds.fasta bbduk_trimq15_maq20_minlen50/spades_assembly/scaffolds.fasta bbduk_trimq15_maq20_minlen100/spades_assembly/scaffolds.fasta bbduk_trimq20_maq20_minlen50/spades_assembly/scaffolds.fasta bbduk_trimq20_maq20_minlen100/spades_assembly/scaffolds.fasta --glimmer`

`~/TXVA$ python ~/quast-5.0.2/quast.py -o quast_results_spades -r '/media/woo-suk/Data/C_peterson/reference_genome/GCF_001642675.1_ASM164267v1_genomic.fna.gz' -g '/media/woo-suk/Data/C_peterson/reference_genome/GCF_001642675.1_ASM164267v1_genomic.gff.gz' -l "spd15-15-50, spd15-15-100, spd15-20-50, spd15-20-100, spd20-20-50, spd20-20-100" bbduk_trimq15_maq15_minlen50/spades_assembly/scaffolds.fasta bbduk_trimq15_maq15_minlen100/spades_assembly/scaffolds.fasta bbduk_trimq15_maq20_minlen50/spades_assembly/scaffolds.fasta bbduk_trimq15_maq20_minlen100/spades_assembly/scaffolds.fasta bbduk_trimq20_maq20_minlen50/spades_assembly/scaffolds.fasta bbduk_trimq20_maq20_minlen100/spades_assembly/scaffolds.fasta --glimmer`

## [Read the full paper here] https://doi.org/10.1128/mra.00467-22
