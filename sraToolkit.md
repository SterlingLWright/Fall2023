# SRA ToolKit
Sequencing files are usually stored in databases such as NCBI and the ENA (European Nucleotide Archive). What if we want to download the sequenced and use them in our analysis? The SRA toolkit by NCBI is the solution in this case. SRA toolkit provides tools for downloading data, converting different formats of data into SRA dormat, and the vice versa, turning SRA data into different formats. 

The most frequently used tool from the toolkit would be fastq-dump that fetch SRA files and download them as fastq formats into our own directories. This page will provide information on how to install the SRA toolkit and use fastq-dump to download sequencing files for downstream analysis. 

# Installation
Please note the following instructions is for Ubuntu system only, and if you would like to install the toolkit for other systems, please download and install the correct package listed in this [page](https://github.com/ncbi/sra-tools/wiki/02.-Installing-SRA-Toolkit) for the system of choice to prevent configuration issues. 

1. Fetch the tar files from the NCBI: 
```
wget --output-document sratoolkit.tar.gz https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-ubuntu64.tar.gz
```

2. Extract the contents from the tar file:
```
tar -vxzf sratoolkit.tar.gz
```
- -v verbose shows the files tar works on while the command is running
- -x extract extract one or more items from an archive
- -z read or write compressed archives through bzip2 format
- -f specifies the file, follow by file names

3. we will append the path to the binaries to your PATH environment variable:
```
export PATH="/storage/home/neh5207/scratch/sratoolkit.3.0.7-ubuntu64/bin:$PATH" # Replace 'neh5207' with your PSU ID
```

4. Verify where the binaries are found by the shell
```
which fasterq-dump
```

5. To download as accession file, you will first use the 'prefetch' tool. Prefetch will download all necessary files
```
prefetch SRR000001
```

You should get the following output: 
```
2023-09-22T00:06:33 prefetch.3.0.7: Current preference is set to retrieve SRA Normalized Format files with full base quality scores.
2023-09-22T00:06:33 prefetch.3.0.7: 1) Downloading 'SRR000001'...
2023-09-22T00:06:33 prefetch.3.0.7: SRA Normalized Format file is being retrieved, if this is different from your preference, it may be due to current file availability.
2023-09-22T00:06:33 prefetch.3.0.7:  Downloading via HTTPS...
2023-09-22T00:06:44 prefetch.3.0.7:  HTTPS download succeed
2023-09-22T00:06:44 prefetch.3.0.7:  'SRR000001' is valid
2023-09-22T00:06:44 prefetch.3.0.7: 1) 'SRR000001' was downloaded successfully
```

6. Next we will use the tool 'fasterq-dump' to extract the data from the SRA-accession file and convert it to fasta/fastq format.
```
fasterq-dump SRR000001
```

the command should produce the following output
```
spots read      : 470,985
reads read      : 1,883,940
reads written   : 707,026
reads 0-length  : 468,635
technical reads : 708,279
```

For a list of additional flags use the following:
```
fasterq-dump -h
```

# How to download multiple files at the same time
## The first thing we need is to create an accessing list, which is a file with all the sampleID you want to download. 
Here's an example: 

Create a file called "accession_list.txt" 
```
nano accession_list.txt
```

The first column in this file will contain the sampleID that we will feed into the fasterq-dump to download, the second column is the runID, and the third column is the name of the study just to keep an record. You can  add more information or less information for your purposes but the sampleID is the required information. 
```
SAMN07790141,SRR6178139,Camacho_Ortiz_2017
SAMN03002640,SRR1556545,PRJNA259188
```

## Next we will feed the accession list to the fasterq-dump to download multiple files at the same time
lets use nano to write this into a shell script called "fastq.sh"
```
nano fastq.sh
```

```
while read line; do

# extract run number
SampleID=$(echo $line | cut -d',' -f1)
RunID=$(echo $line | cut -d',' -f2)
echo $SampleID $RunID

if [ ! -f "reads/${SampleID}/${RunID}_1.fastq.gz" ]; then
	mkdir -p reads/$SampleID # create a folder for each Biosample
	cd reads/$SampleID
	echo \-\-\-\> Downloading $SampleID
	fasterq-dump --split-files --gzip $RunID # use fasterq-dump to download its reads
	cd ../../
else
	echo \-\-\-\> $SampleID already downloaded
fi

done <accession_list.txt # move to next sample (ie line of file)
```

```
bash fastq.sh
```
