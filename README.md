# Fall 2023 Workshop 1 September 22, 2023

## Table of contents 

**Part 1. Basic set up of the computational environment**

1. Roar configuration
2. Accessing the Roar 
3. Submitting jobs to the Roar
4. Conda configuration

**Part 2. Metagenomics data processing**

1. Quality check and adapter removal
2. Taxonomy alignment (by MALT (MEGAN alignment tool))
3. Other taxonomy alignment/classification tools

---------------

## Part 1. Basic set up of the computational environment

### Roar configuration

Let's go over our [Wiki](https://github.com/DAWGPSU/DAWG_workshop/wiki/Roar-configuration) page to set up the Roar

***Check points***
```
1. Do you have your account requested and approved?
```

----------------

### Accessing the Roar Using ACI shell access

You can connect to the submission portal via web-based shell access. It is very similar to SSH, so consider using this approach if you have a problem connecting via SSH. 

1. Go to https://portal2.aci.ics.psu.edu
2. Click "Clusters" from the top menu, then click ">_ACI Shell Access"
3. You will see a very similar screen than you connect via SSH. Once you open the windows, the system will ask you for 2-factor authentication via the Duo app. Once authentication is completed, you should be connected to Roar.

#### Using interactive desktop

Unlike SSH or shell access to submit.aci.ics.psu.edu, opening an interactive desktop means you are **[requesting resources](https://www.icds.psu.edu/computing-services/roar-user-guide/roar-system-layout/)** from the Roar. You can run your code, get the results, and monitor your job. It is a "virtual" computer you are borrowing from the university. 

1. Go to https://portal2.aci.ics.psu.edu
2. Click "Interactive Apps" from the top menu, then click "ACI RHEL7 Interactive Desktop."
3. You can launch an interactive desktop using the below configurations - 
  * Desktop Environment - MATE Gnome2
  * Allocation - Open (OR choose paid allocation if you have any)
  * Number of hours - You can use up to 48 hours if using Open allocation, and more time can be used if you use paid allocation.
  * Node type - ACI-i is only available for open account submissions. Other node types can be used too.

***Check point***
```
1. Did you launch an interactive desktop for 2 hours for this workshop?
```

----------------

### Submitting jobs to the Roar

See here for multiple ways to submit jobs to Roar [Wiki page](https://github.com/DAWGPSU/DAWG_workshop/wiki/Submitting-Job-to-Roar)

Today, we will use the interactive desktop to code in real time. 

----------------

### Conda configuration

![image](https://user-images.githubusercontent.com/126822453/222797077-f2a1cafb-9b79-4e0f-9396-77df31504515.png)

We need many bioinformatics programs to be installed to process metagenomics sequencing data. [Conda](https://docs.conda.io/en/latest/) is a package and environment manager to make installing, executing, and managing multiple packages easier. 

We are following instructions from [Conda website](https://conda.io/projects/conda/en/latest/user-guide/getting-started.html)

**Install miniconda3**

```
cd ~/scratch
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh 
sh Miniconda3-latest-Linux-x86_64.sh -b -p /storage/work/[yourPSUID]/miniconda3
conda init
source ~/.bashrc
```

ALTERNATIVES: USE ANACONDA3 from the ROAR module

```
module load anaconda3
conda init
```
Please note that you need to modified your path of installing packages other than your home directory, because your home directory has limited storage (~10gb). 


Add [bioconda](https://bioconda.github.io) and [conda-forge](https://conda-forge.org) channel to the conda package manager

```
conda config --add channels bioconda
conda config --add channels conda-forge
```

**Set up conda environment with required packages - and additional programs installation with initial configuration**

A conda environment is where you can install a collection of conda packages. The main advantage of using an environment is to avoid the different versions of the dependencies from various bioinformatics tools because you only can use the installed packages in the specific environment. 

Create an environment

```
conda create -y --name DAWG python=3.8 # Create an environment named "DAWG" and install "python (version 3.8)"
conda activate DAWG # Activate your environment; now you only use packages installed in your environment
```
**package installation**

Now you can use the conda installer to install bioinformatics packages. You can search and find if your bioinformatics tool was uploaded in conda repository (https://anaconda.org). 

1. Go to https://anaconda.org

![image](https://user-images.githubusercontent.com/126822453/222825235-44ee3599-2e6f-4392-8072-c6baf9ac58e5.png)

2. Search the package you want to install
3. Click the package name

4. follow the conda install instructions; please replace "conda" with "mamba" if you want to use the mamba installer

![image](https://user-images.githubusercontent.com/126822453/222825138-00e8aa3e-1ca4-43e7-8b43-0d57b517410e.png)

For example, if you want to install the "AdapterRemoval" package, you can type below -

```
## using conda installer
conda install -c bioconda adapterremoval

## using mamba install
mamba install -c bioconda adapterremoval
```

The results should be the same. One fantastic thing about conda/mamba is that they download and install all the required and optional dependencies together with the package, so you don't have to have a headache installing individual dependencies. 


5. Check your installation 
```
adapterremoval --help
```


***Check point***
```
1. Did you install conda and mamba successfully?
2. Did you install AdapterRemoval successfully?
```

------------------------------------

Now this is the end of the part 1. Let's have a deep breath! 

![image](https://user-images.githubusercontent.com/126822453/224905728-39bab550-e933-42f0-876c-3150ab5b7cfa.png)

-------------------------------------

## Part 2. Set up computational environment

### Download data into your "scratch" folder

```
cd ~/scratch
wget --no-check-certificate "https://pennstateoffice365-my.sharepoint.com/:u:/g/personal/tuc289_psu_edu/EfPeT7R3E2VHmod0UcrNZlgBNKBHH-K5jvbIGY4PXorToA?e=OsM0L7&download=1" -O DAWG_031723.zip
unzip DAWG_031723.zip -d DAWG_031723
cd DAWG_031723
```

### Data structure

What is fastq file? 

a text-based format for storing both a biological sequence and its corresponding quality scores. It encodes phred scores as ASCII characters 

![image](https://user-images.githubusercontent.com/126822453/225454077-dfeee831-961e-403e-9c52-e6b824ffc6a4.jpeg)

Phred Quality Score - Used to compute the probability of a basecall being incorrect by the formula 

![image](https://user-images.githubusercontent.com/126822453/225454283-6675eddf-b6eb-467a-8d77-3a30df9b0e4c.png)


### Quality check and adapter removal

After you receive your sequence files from the sequencer, you first need to check the quality of your data. Then, you need to remove non-biological sequences from your file (i.e., Illumina adapter, index sequences, etc.)

#### Quality check by FastQC

Fastqc is a quality control tool for high throughput sequence data. It provides a simple way to do quality checks on your raw sequence files before and after adapter removal. Fastqc provides an HTML report to visualize your data's quality.

1. load fastqc from the [Roar software stack](https://www.icds.psu.edu/user-guide-roar-software/) 

```
module avail
module load fastqc
```

2. Running fastqc using the command line

```
fastqc *.fastq -o ./

./ - current director
*.fastq - list all files in the current directory ending with ".fastq"
```

Let's check the results .html file by either downloading or opening in the interactive desktop.

# copy .html files to your local machine 
rsync -P svw5689@submit.hpc.psu.edu:/storage/home/svw5689/Bioinformatic_Programs/*html .
 
****(optional)****
MultiQC

MultiQC is a tool to combine multiple fastqc report into one .html file.
```
mamba install -c bioconda multiqc
multiqc ./
```

****Check point****
```
1. Did you get your fastqc (or multiqc) report?
2. Did you get familiar with using software from the Roar?
3. Did you get used to installing bioinformatics package using mamba (or conda)?
```

#### Removing adapters

We already installed adapterremoval from the previous part. Now we will use adapterremoval to remove Illumina adapters from the data. 

AdapterRemoval removes adapter seqeunces from FASTQ files while also trim low quality bases. This program can process both single end and paired end data. It can also be used to merge overlapping paired-ended reads into (longer) consensus sequences. Furthermore, AdapterRemoval can generate a consensus adapter sequence for paired-ended reads. The program can also demultiplex data. 

In short, AdapterRemoval is a comprehensive tool that exhibits good performance in terms of sensitivity and specificity. 

To run AdapterRemoval on a single sample 
```  
AdapterRemoval --file1 P1_PortugeseKing.pair1_100k.fastq --file2 P1_PortugeseKing.pair2_100k.fastq --trimns \
--trimqualities --minquality 20 --collapse --minlength 25 \
--output1 P1_PortugeseKing_trimmed_R1.fastq --output2 P1_PortugeseKing_trimmed_R2.fastq \
--outputcollapsed P1_PortugeseKing_trimmed_merged.fastq --settings P1_PortugeseKing_AdapterRemoval.txt;done;

AdapterRemoval --file1 P2_PortugeseKing.pair1_100k.fastq --file2 P2_PortugeseKing.pair2_100k.fastq --trimns \
--trimqualities --minquality 20 --collapse --minlength 25 \
--output1 P2_PortugeseKing_trimmed_R1.fastq --output2 P2_PortugeseKing_trimmed_R2.fastq \
--outputcollapsed P2_PortugeseKing_trimmed_merged.fastq --settings P2_PortugeseKing_AdapterRemoval.txt;done;
```

To run AdapterRemoval in a loop  

```
arr=(
"P1_PortugeseKing"
"P2_PortugeseKing")

for i in "${arr[@]}";
 do
        echo "$i"
AdapterRemoval --file1 "$i".pair1_100k.fastq --file2 "$i".pair2_100k.fastq --trimns \
--trimqualities --minquality 20 --collapse --minlength 25 \
--output1 "$i"_trimmed_R1.fastq --output2 "$i"_trimmed_R2.fastq \
--outputcollapsed "$i"_trimmed_merged.fastq --settings "$i"_AdapterRemoval.txt;done;
```

Run AdapterRemoval in a pbs script

First, create a text document and give it a .pbs extension
```
nano adapterremoval.pbs
```
Next, paste the following code into the document. Note: your -A option will be diffierent. The lsw132_d_g_sc_default is an allocation that is specific to my lab's group. You will have to change this in order for the code to work. I also created a conda environment for AdapterRemoval to run. You will also need to change the path for where you want the script to be ran. 
```
#!/bin/bash
#PBS -l nodes=1:ppn=4
#PBS -l walltime=10:00:00
#PBS -l pmem=60gb
#PBS -A lsw132_d_g_sc_default

# AdapterRemoval on NextSeq data

set -uex

#activating AdapterRemoval enviornment
module load anaconda3

source activate AdapterRemoval

# you will need to uncomment and change $PATH to match your path where you store your files 
#cd $PATH

arr=(
"P10_PortugeseKing"
"P11_EBC1_PortugeseKing"
"P12_EBC2_PortugeseKing"
"P1_PortugeseKing"
"P2_PortugeseKing"
"P3_PortugeseKing"
"P4_PortugeseKing"
"P6_PortugeseKing"
"P7_PortugeseKing"
"P8_PortugeseKing"
"P9_PortugeseKing")

## This script uses a for loop to run the pipeline on each sample in the working directory
for i in "${arr[@]}";
 do
        echo "$i"
AdapterRemoval --file1 "$i".pair1.fastq --file2 "$i".pair2.fastq --trimns \
--trimqualities --minquality 20 --collapse --minlength 25 \
--output1 "$i"_trimmed_R1.fastq --output2 "$i"_trimmed_R2.fastq \
--outputcollapsed "$i"_trimmed_merged.fastq --settings "$i"_AdapterRemoval.txt;done
```
Once you are happy with the script, you need to save and close out. 
To run a pbs script, use the following code: 
```
qsub -A lsw132_d_g_sc_default adapterremoval.pbs
```

Once the script is submitted, you can then check whether the script is being runned. 
```
qstat -n1tu svw5689
```
You will need to change the Penn State Access ID (i.e. svw5689) to match yours. This will show you something like this: 
```
                                                                                  Req'd       Req'd       Elap
Job ID                  Username    Queue    Jobname          SessID  NDS   TSK   Memory      Time    S   Time
----------------------- ----------- -------- ---------------- ------ ----- ------ --------- --------- - ---------
41967608.torque01.util  svw5689     batch    adapterremoval    --      1      4      60gb  10:00:00 Q 
```
If the letter under is a Q, it means that your script is being queued. If it is a R, the script is being ran. If it is a C, it means that the script was cancelled and there were an error.

Once the adapters for all your samples have been removed, you are now read to perform taxonomic alignment and other downstream analyses (e.g., phylogenomics). 


### Taxonomy alignment by MALT (Megan Alignment tool)

Taxonomic classification, that is, identifying the microbial species present within a sample is a standard step in microbiome research. Targeted amplification of the 16S ribosomal RNA encoding gene is one approach for taxonomic classification. However, 16S rRNA sequencing has limitations in regards to providing functional and genomic information. As such, many researchers have turned to shotgun sequencing approaches to reconstruct microbial communities. 

Methods for analyzing shotgun sequencing can be placed into three broad categories: assembly-based, alignment-free, and alignment-based. Assembly-based approaches require merging overlapping DNA fragments into longer sequences which can be assembled into whole genomes. However, this is not always possible. Alignment-free methods involve the DNA sequences themselves, such as matches of k-mers between reference genomes and the DNA sequences within a sample. This approach has shown to be produce a high number of misclassifications and false positives. Alignment-based methods, on the other hand, require the alignment of DNA fragments to previously characterized reference sequences using alignment algorithms. Many alignment tools exist, but it has been shown that MALT (MEGAN Alignment tool) outperforms the other ones. 

MALT is an alignment-based tool that allows researchers to query DNA sequences against reference databases. MALT can align nucleotide sequences to nucleotide databases (MALTn) or nucleotide to amino acid databases (MALTx). 

In this section, you will be given the code that is used to index the reference database, align sequences to the database, and convert these into RMA files. RMA files store reads, alignments, as well as taxonomic and functional classification information for a given sample. The file is indexed so that it is easy to access the reads and alignments. 

Link to malt installation: https://uni-tuebingen.de/fakultaeten/mathematisch-naturwissenschaftliche-fakultaet/fachbereiche/informatik/lehrstuehle/algorithms-in-bioinformatics/software/malt/

Index the reference database 
```
malt-build -i RefSeqONLY-Assembly-Arch-Bact-Genome-Chromosome-Scaffold-27-7-17/CATTED-arch-bact.fna.gz -d MALT_BUILD \
--sequenceType DNA > malt-build.log
```

Make sure the malt environment is active 
```
source env_malt_038.sh
```

#### Align sequences against database using malt-run
```
# set variables
# directories where you store the samples 
SAMPLE_FOLDER=Samples

# path where you store the database 
DATABASE=/store/common/data/MALT_BUILD_v_0.6.1

# save the RMA output files to a specific folder 
mkdir RMA_FILES
RMA=RMA_FILES

# save the BLAST output files to a specific folder 
mkdir BLAST_FILES
BLAST_FILES=BLAST_FILES

# begin malt-run
malt-run -i $SAMPLE_FOLDER/P1_PortugeseKing_trimmed_merged.fastq -d $DATABASE --mode BlastN -at SemiGlobal --minPercentIdentity 90 --minSupport 0 --maxAlignmentsPerQuery 25 --gapOpen 7 --gapExtend 3 --band 4 -o RMA_FILES --memoryMode load --verbose true -t 40 --alignments RMA_FILES --format Text

malt-run -i $SAMPLE_FOLDER/P2_PortugeseKing_trimmed_merged.fastq -d $DATABASE --mode BlastN -at SemiGlobal --minPercentIdentity 90 --minSupport 0 --maxAlignmentsPerQuery 25 --gapOpen 7 --gapExtend 3 --band 4 -o RMA_FILES --memoryMode load --verbose true -t 40 --alignments RMA_FILES --format Text
```

#### options for malt-run 
-m --mode; legal values include BlastN, BlastP, BlastX 

-at --alignmentType; type of alignment to be performed. Default value is Local. Legal values include Local, SemiGlobal. 

--minPercentIdentity; minimum percent identity used by LCA algorithm (default: 0) 

--minSupport; minimum support value for LCA algorithm 

--maxAlignmentsPerQuery; maximum alignments per query (default: 25)

--gapOpen; gap open penalty (default: 11)

--gapExtend; gap extension penalty (default: 1) 

--band; band width for banded alignment (default: 4)

-o; output RMA files 

--memoryMode; memory mode (default: load, page, map)

--verbose; read out each command line option (default: false) 

--alignments; output alignment files 

--format; output alignment format (default: SAM), legal values: SAM, Tab, Text

#### Run blast2rma 
blast2rma is similar to the sam2rma program as it generates RMA files from BLAST output. 
```
blast2rma -i $RMA/P1_PortugeseKing.blastn -r $SAMPLE_FOLDER/P1_PortugeseKing_trimmed_merged.fastq -o $BLAST_FILES/P1_PortugeseKing.blast2rma -ms 44 -me 0.01 -f BlastText -supp 0.1 -alg weighted -lcp 80 -t 40 -v --blastMode BlastN

blast2rma -i $RMA/P2_PortugeseKing.blastn -r $SAMPLE_FOLDER/P2_PortugeseKing_trimmed_merged.fastq -o $BLAST_FILES/P1_PortugeseKing.blast2rma -ms 44 -me 0.01 -f BlastText -supp 0.1 -alg weighted -lcp 80 -t 40 -v --blastMode BlastN
```

##### options for blast2rma 
-i; input BLAST files 

-r; reads file(s) (fasta or fastq)

-o; output file

-ms, --minScore; Min score (default: 50.0) 

-me, --maxExpected; Max expected (default: 0.01)

-f; format, input file format 

-supp, --minSupportPercent; min support as percet of assigned reads (default: 0.01)

-alg, --lcaAlgorithm; set the LCA algorithm to use for taxonomic assignment, legal values: naive, weighted, longReads

-lcp, --lcaCoveragePercent; set thhe percent for the LCA to cover (default: 100.0)

-t; threads 

-v; verbose commands 

--blastMode; legal values BlastN, BlastP, BlastX, Classifier


#### Run malt-run and blast2rma in a loop on all samples 
```
nohup cat Samples.txt | while read samples; do malt-run -i $samples -d $DATABASE --mode BlastN -at SemiGlobal --minPercentIdentity 90 --minSupport 0 --maxAlignmentsPerQuery 25 --gapOpen 7 --gapExtend 3 --band 4 -o RMA_FILES --memoryMode load --verbose true -t 40 --alignments RMA_FILES --format Text ; blast2rma -i $RMA/$samples.blastn.gz -r $samples -o $BLAST_FILES/$samples.blast2rma -ms 44 -me 0.01 -f BlastText -supp 0.1 -alg weighted -lcp 80 -t 40 -v --blastMode BlastN ; done
```

Link to MEGAN: https://uni-tuebingen.de/fakultaeten/mathematisch-naturwissenschaftliche-fakultaet/fachbereiche/informatik/lehrstuehle/algorithms-in-bioinformatics/software/megan6/


### Other taxonomy alignment/classification tools

#### Kraken2 and Bracken

You can find detailed introduction of Kraken2 and Bracken [here](https://github.com/DAWGPSU/DAWG_workshop/wiki/Taxonomy-alignment-tools#kraken2-and-bracken)

**Installation**
```
mamba install -c bioconda kraken2
mamba install -c bioconda bracken
```

**Classification**
```
kraken2 --db kraken_db P1_PortugeaseKing_trimmed_R1.fastq P1_PortugeaseKing_trimmed_R2.fastq --output P1.kraken --paired --threads 20 --report P1.kraken.report
kraken2 --db kraken_db P2_PortugeaseKing_trimmed_R1.fastq P2_PortugeaseKing_trimmed_R2.fastq --output P2.kraken --paired --threads 20 --report P1.kraken.report 
```

**Bracken abundance estimation**

```
bracken -d kraken_db -i P1.kraken.report -r 150 -l S -o P1.bracken -t 8
bracken -d kraken_db -i P2.kraken.report -r 150 -l S -o P2.bracken -t 8
```

Check the [output](https://github.com/DAWGPSU/DAWG_workshop/tree/fc9c1f8f516b5f839c14d03af1a76abf8de61202/Example_output_Mar17/Kraken_bracken)


#### mOTU

You can find detailed introduction of mOTUs [here](https://github.com/DAWGPSU/DAWG_workshop/wiki/Taxonomy-alignment-tools#motu)


**Insatllation**

```
mamba install -c bioconda motus
# Download the mOTUs database (~3.5GB)
motus downloadDB
```

**Automated pipeline from raw reads to profile**

```
## Change -l (minimum length to align) as 45 [default:75] because we are using 150bp, not 250bp
## Change -g as 1 [default:3] to do more generous classification (higher FPR)
motus profile -f P1_PortugeseKing_trimmed_R1.fastq -r P1_PortugeseKing_trimmed_R2.fastq -t 8 -l 45 -g 1 > P1.motus.txt
motus profile -f P2_PortugeseKing_trimmed_R1.fastq -r P2_PortugeseKing_trimmed_R2.fastq -t 8 -l 45 -g 1 > P2.motus.txt

## Merge results into one file
motus merge -i P1.motus.txt,P2.motus.txt > all_sample_profiles.txt
```

**Results**

Check the [output](https://github.com/DAWGPSU/DAWG_workshop/tree/b74fc2a583e727bcd4fb740550e160a59100acd1/Example_output_Mar17/motus)


#### MetaPhlAn

You can find detailed introduction of MetaPhlAn [here](https://github.com/DAWGPSU/DAWG_workshop/wiki/Taxonomy-alignment-tools#metaphlan)

**Installation**

```
mamba install -c conda-forge -c bioconda metaphlan

## Install database 
metaphlan --install --bowtie2db ./
```

**Classification**

```
metaphlan P1_PortugeseKing_trimmed_R1.fastq,P1_PortugeseKing_trimmed_R2.fastq --bowtie2out P1.out.bowtie2.bz2 --nproc 8 --input_type fastq --unclassified_estimation -o P1_metagnome.txt --bowtie2db ./

metaphlan P2_PortugeseKing_trimmed_R1.fastq,P2_PortugeseKing_trimmed_R2.fastq --bowtie2out P2.out.bowtie2.bz2 --nproc 8 --input_type fastq --unclassified_estimation -o P2_metagnome.txt --bowtie2db ./

merge_metaphlan_tables.py P1_metagnome.txt P2_metagnome.txt > merged_abundance_table.txt
```

**Results**

Check the [output](https://github.com/DAWGPSU/DAWG_workshop/tree/f8965379de89bec0e06ed79db15384377a70504e/Example_output_Mar17/metaphalan)

## References 
Schubert, M., Lindgreen, S. and Orlando, L., 2016. AdapterRemoval v2: rapid adapter trimming, identification, and read merging. BMC research notes, 9(1), pp.1-7.

Huson, D.H., Beier, S., Flade, I., Górska, A., El-Hadidi, M., Mitra, S., Ruscheweyh, H.J. and Tappu, R., 2016. MEGAN community edition-interactive exploration and analysis of large-scale microbiome sequencing data. PLoS computational biology, 12(6), p.e1004957.

Eisenhofer, R. and Weyrich, L.S., 2019. Assessing alignment-based taxonomic classification of ancient microbial DNA. PeerJ, 7, p.e6594.

Velsko, I.M., Frantz, L.A., Herbig, A., Larson, G. and Warinner, C., 2018. Selection of appropriate metagenome taxonomic classifiers for ancient microbiome research. Msystems, 3(4), pp.e00080-18.

Vågene, Å.J., Herbig, A., Campana, M.G. et al. Salmonella enterica genomes from victims of a major sixteenth-century epidemic in Mexico. Nat Ecol Evol 2, 520–528 (2018). https://doi.org/10.1038/s41559-017-0446-6

Wood, D.E., Lu, J. & Langmead, B. Improved metagenomic analysis with Kraken 2. Genome Biol 20, 257 (2019). https://doi.org/10.1186/s13059-019-1891-0

Lu J, Breitwieser FP, Thielen P, Salzberg SL. (2017) Bracken: estimating species abundance in metagenomics data. PeerJ Computer Science 3:e104, doi:10.7717/peerj-cs.104 

Ruscheweyh, HJ., Milanese, A., Paoli, L. et al. Cultivation-independent genomes greatly expand taxonomic-profiling capabilities of mOTUs across various environments. Microbiome 10, 212 (2022). https://doi.org/10.1186/s40168-022-01410-z

Blanco-Míguez, A., Beghini, F., Cumbo, F. et al. Extending and improving metagenomic taxonomic profiling with uncharacterized species using MetaPhlAn 4. Nat Biotechnol (2023). https://doi.org/10.1038/s41587-023-01688-w





