# Qiime2 Amplicon Cleanup
All of the code used here is compatible with the package Qiime2 amplicon version 2024.10

## Purpose: 
To make pipeline for SSU & 16S amplicon sequencing cleanup and taxonomic identification

## STEP 1: Prepare Files & Environment
### Using local drive
Make sure sequences are downloaded in an accessible location on the local drive.


<ins>Download Qiime2 on local drive</ins>
```
Need code for this
```
<br>Activate Qiime2 in terminal command line
```
conda activate /Users/yo/miniconda/envs/qiime2-amplicon-2024.10
```

<br>Set working directory along the path to where the directory where the sequences are stored. <br>
For example, if your sequences are found in /home/project/sequences, set your working directory to /home/project
```
cd /path/to/working/directory
```

<br>### Using HPC

Upload all fastq.gz files onto HPC
```
scp -r "path/to/folder" \ 
user@koa.its.hawaii.edu:/home/user/path/to/directory/for/files
```
<br><ins>Installing Qiime2 on HPC</ins>

To access KOA on command line run the code below, then enter your UH password & designate two factor authentication preference. NOTE: password is invisible and does not show key strokes!
```
ssh userj@koa.its.hawaii.edu
```
Start interactive job
```
srun -p shared --mem=100G -c 4 -t 06:00:00 --pty /bin/bash
```

<br><ins>Stay on you home directory for Qiime2 installation</ins>
<br>Load anaconda module (this is already installed on the HPC for all users)
```
module load lang/Anaconda3/2024.02-1
```

<br>Install Qiime2 
``` 
conda env create -n qiime2 --file https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.10-py310-linux-conda.yml
```
<br>NOTE: line after -n is what the environment will be named, in the code below it is "qiime2" but could be anything! You will use this name to activate Qiime2 on the HPC, so take note of whatever you name it.
<br><br>Check conda environments to make sure it installed correctly by looking for what you named your Qiime2 package during installation.
```
conda info - e
#conda environments:                             
#qiime2    /home/alliej/.conda/envs/qiime2
#base      /opt/apps/software/lang/Anaconda3/2024.02-1  
```

<br><ins>Activating Qiime2 on HPC</ins>
<br>NOTE: this is different than how conda is activated on a local drive or personal computer!
<br>For each new session, make sure you perform the following prior to activating Qiime2:
  1. Start an interactive job
  2. Load anaconda module
```
source activate qiime2
```

<br>Set working directory along the path to where the directory where the sequences are stored.
<br>For example, if your sequences are found in /home/project/sequences, set your working directory to /home/project
```
cd /path/to/working/directory
```

## STEP 2: Importing Data Into Qiime2
Instructions on importing sequences into a qiime2 artifact can be found [HERE](https://docs.qiime2.org/2024.10/tutorials/importing/).
<br>Qiime2 visualization files (.qzv) can be viewed [HERE](https://view.qiime2.org/?src=e96f979f-4cc6-46fc-800f-abe58740e4ea).

**Import paired-end sequences using Casava 1.8 paired-end demultiplexed fastq method**
```
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path path/to/directory/containing/gzipped/fastq/files \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path path/to/where/qiime2/artifact/will/be/saved/file-name.qza
```
Convert qiime artifcat into visualization file 
```
qiime demux summarize \
  --i-data path/to/where/qiime2/artifact/was/saved/file-name.qza \
  --o-visualization path/to/where/qiime2/visual/file/will/be/saved/file-name.qzv
```
Go to Qiime2 Viewer on browser and upload the .qzv file and check to see if Forward and Reverse read counts are the same! [HERE](https://forum.qiime2.org/t/demultiplexed-sequence-length-summary-identical-forward-and-reverse-for-emp-paired-end-reads/20692) is a forum with more information on this.
<br><ins>Record total reads</ins>

<br>*Go to the "Interactive Quality Plot" tab*
<br>Here, you can observe the quality of reads (y-axis) at a specific sequence length (x-axis) and assess (1) wether sequences should be trimmed and (2) how much they should be trimmed. The quality score tells us how confidently the nucleotide was called at that area, higher quality scores indicate higher confidence that the detected nucleotide is correct and not an error. Generally, quality scores below 20 are considered poor and unreliable reads which can lead to inacurate taxonomic identification later on.  
<br><ins>Approach to deciding how much to trim:</ins>
1. Observe how consistent the quality is by assessing the sequence length at which the quality begins to drop abrubptly, as well as how steep the drop in quality is.
2. Determine at what sequence length the quality score begins to drop under 20.
3. You can zoom in and out of the quality plots by highlighting regions of interest, and double clicking the plot to zoom back out again.
4. Note that Reverse reads are generally lower in quality and may require more trimming than Forward reads.
5. Take a screen shot of the quality plots for your records.

If you want to record or observe reads per sample, scroll to the very bottom of the "Overview" page and click "Download as TSV" to download per-sample-fastq-counts.tsv

<ins>If total read counts are different for Forward and Reverse sequences, check the following:</ins>

1. All files were uploaded correctly (should not be 0 bytes).
<br>Go to directory where sequencing files are kept
```
cd path/to/sequencing/files.fastq.gz
```
Check file sizes by listing files from largest to smallest (smallest will be at the bottom), are any 0 bytes?
```
ls -lhS
``` 
2. Open per-sample-fastq-counts.tsv and search for samples where forward reads and reverse reads aren't the same.
<br>Check the sample file for those that don't match up
```
zcat /path/to/file.fastq.gz | echo $((`wc -l`/4))
```
3. Check locally stored files and re-upload any files that were found to be 0 bytes, or have mismatched reads in the per-sample-fastq-counts.tsv file
```
scp "/local/path/to/file.fastq.gz" \
user@koa.its.hawaii.edu:/hpc/path/to/file/directory
```
Check to make sure they re-uploaded correctly
```
zcat /path/to/file.fastq.gz | echo $((`wc -l`/4))
```
4. Re-rerun import code above and check the .qzv file to see if the problem was fixed

<br>**Import Forward sequences using Casava 1.8 single-end demultiplexed fastq method**
<br><br><ins>Copy all foward sequences into new directory</ins>
<br>All forward sequence files share a common suffix in their file names such as READ1.fastq.gz or R1_001.fastq.gz. Use this shared suffix, unique from the reverse reads (READ2.fastq.gz or R2_001.fastq.gz), to differentiate forward from reverse reads within the code.
```
cp /path/to/directory/*R1_001.fastq.gz \
/path/to/new/directory
```
Import sequences into a Qiime2 artifact (.qza)
```
qiime tools import \
  --type 'SampleData[SequencesWithQuality]' \
  --input-path path/to/directory/containing/gzipped/forward/fastq/files \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path path/to/where/qiime2/artifact/will/be/saved/file-name.qza
```
Convert qiime artifcat into visualization file 
```
qiime demux summarize \
  --i-data path/to/where/qiime2/artifact/was/saved/file-name.qza \
  --o-visualization path/to/where/qiime2/artifact/will/be/saved/file-name.qzv
```
See paired-end section for next steps using the Qiime2 visualization file (.qzv)
<br><br>If running multiple tests on the same set of sequences, single end and paired-end outputs should have the output parameters with the exception of no Reverse section being shown.

## STEP 3: Trim Primers From Sequences
This section uses cutadapt, the handbook can be found [HERE](https://docs.qiime2.org/2024.10/plugins/available/cutadapt/index.html).
<br>For background on trimming Golay barcodes see [THIS](https://forum.qiime2.org/t/cutadapt-adapter-vs-front/15450) forum page.

**Primers commonly used in our studies:**
<br><ins>Bacterial small ribosomal subunit (16S) V4 amplicon using 515F/806R primers</ins>
  <br>515F (Forward primer): GTGYCAGCMGCCGCGGTAA
  <br>806R (Reverse primer): GGACTACNVGGGTWTCTAAT
<br><br><ins>Fungal small ribosomal subunit (18s) amplicon using WANDA/AML2 primers</ins>
  <br>WANDA (Forward Primer): CAGCCGCGGTAATTCCAGC
  <br>AML2 (Reverse Primer): GAACCCAAACACTTTGGTTTCC

<br>**Paired-end sequences**
<br>This method trims primers based on primer sequence rather than length
```
qiime cutadapt trim-paired \
   --i-demultiplexed-sequences path/to/where/qiime2/import/artifact/was/saved/file-name.qza \
   --p-front-f FWDPRIMERSEQUENCE \
   --p-front-r REVPRIMERSEQUENCE \
   --o-trimmed-sequences path/to/where/file/will/be/saved/file-name.qza \
   --verbose
```
--verbose tells Qiime2 to print out the logging output to the terminal while it runs the command. This can help trouble shoot any errors that may occur during the run, but is not necessary for trimming process.
<br><br>Convert qiime artifcat into visualization file
```
qiime demux summarize \
  --i-data path/to/where/file/was/saved/file-name.qza \
  --o-visualization path/to/where/file/will/be/saved/file-name.qzv
```
Go to Qiime2 Viewer on browser and upload the .qzv file, <ins>record total reads and any other pertinent information</ins>. Compare these outputs to your imported outputs from STEP 2. There shouldn't be any differences if everything worked correctly!
<br><br>Scroll to the very bottom of the "Overview" page and click "Download as TSV" to download per-sample-fastq-counts.tsv post primer trimming if desired.

<br>**Single-end sequences**
<br>This method is similar to trimming primers off of paired end sequences, with minor differences.
```
qiime cutadapt trim-single \
   --i-demultiplexed-sequences path/to/where/qiime2/import/artifact/was/saved/file-name.qza \
   --p-front FWDPRIMERSEQUENCE \
   --o-trimmed-sequences path/to/where/file/will/be/saved/file-name.qza \
   --verbose
```
Convert qiime artifcat into visualization file
```
qiime demux summarize \
  --i-data path/to/where/file/was/saved/file-name.qza \
  --o-visualization path/to/where/file/will/be/saved/file-name.qzv
```
See paired-end section for next steps using the Qiime2 visualization file (.qzv)
<br><br>If running multiple tests on the same set of sequences, single end and paired-end outputs should have the output parameters with the exception of no Reverse section being shown.





## STEP 4: DADA2 Trimming And Denoising
## STEP 5: Export Feature Table For Culling
## STEP 6: Import Databases For Taxonomic Identification
## STEP 7: Taxonomic Assignment To Features
## STEP 8: Filtering Taxonomic Tables
## STEP 9: Merging Taxonomic Tables And Classification Files
## STEP 10: Export Final Tables And Representative Sequences




