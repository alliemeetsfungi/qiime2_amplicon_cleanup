# qiime2_amplicon_cleanup
## Test 1
### Test 2
reg text


## Purpose: 
To make pipeline for SSU & 16S amplicon sequencing cleanup and taxonomic identification

## Step 1: Prepare Files & Environment
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

### Using HPC

Upload all fastq.gz files onto HPC
```
scp -r "path/to/folder" \ 
user@koa.its.hawaii.edu:/home/user/path/to/directory/for/files
```

<ins>Installing Qiime2 on HPC</ins>

To access KOA on command line run the code below, then enter your UH password & designate two factor authentication preference
<br>NOTE: password is invisible and does not show key strokes!
```
ssh userj@koa.its.hawaii.edu
```

<br>Start interactive job
```
srun -p shared --mem=100G -c 4 -t 06:00:00 --pty /bin/bash
```

<br>Stay on you home directory for Qiime2
<br>Load anaconda module (this is already installed on the HPC for all users)
```
module load lang/Anaconda3/2024.02-1
```

<br>Install Qiime2 
<br>NOTE: line after -n is what the environment will be named, in the code below it is "qiime2" but could be anything! 
<br>You will use this name to activate Qiime2 on the HPC, so take note of whatever you name it.
``` 
conda env create -n qiime2 --file https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.10-py310-linux-conda.yml
```

<br>Check conda environments to make sure it installed correctly by looking for what you named your Qiime2 package during installation.
```
conda info - e
#conda environments:                             
#qiime2    /home/alliej/.conda/envs/qiime2
#base      /opt/apps/software/lang/Anaconda3/2024.02-1  
```

<br>Activating Qiime2 on HPC
```
source activate qiime2
```
<ins>NOTE: this is different than how conda is activated on a personal computer!</ins>

<br>Set working directory along the path to where the directory where the sequences are stored.
<br>For example, if your sequences are found in /home/project/sequences, set your working directory to /home/project
```
cd /path/to/working/directory
```
## Step 2: Importing Data Into Qiime2
Instructions on importing sequences into a qiime2 artifact can be found [here](https://docs.qiime2.org/2024.10/tutorials/importing/)
<br>Qiime2 visualization files (.qzv) can be viewed [here](https://view.qiime2.org/?src=e96f979f-4cc6-46fc-800f-abe58740e4ea)

**Import paired-end sequences using Casava 1.8 paired-end demultiplexed fastq method**
```
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path path/to/directory/containing/fastq.qz/files \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path path/to/where/qiime2/artifact/will/be/saved/file-name.qza
```
Convert qiime artifcat into visualization file 
```
qiime demux summarize \
  --i-data path/to/where/qiime2/artifact/was/saved/file-name.qza \
  --o-visualization path/to/where/qiime2/visual/file/will/be/saved/file-name.qzv
```
Go to Qiime2 Viewer on browser and upload the .qzv file and check to see if Forward and Reverse read counts are the same! [Here](https://forum.qiime2.org/t/demultiplexed-sequence-length-summary-identical-forward-and-reverse-for-emp-paired-end-reads/20692) is a forum with more information on this.

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

<br>2. Open per-sample-fastq-counts.tsv and search for samples where forward reads and reverse reads aren't the same
<br>Check the sample file for those that don't match up
```
zcat /path/to/file.fastq.gz | echo $((`wc -l`/4))
```

<br>3. Check locally stored files and re-upload any files that were found to be 0 bytes, or have mismatched reads in the per-sample-fastq-counts.tsv file
```
scp "/local/path/to/file.fastq.gz" \
user@koa.its.hawaii.edu:/hpc/path/to/file/directory
```
Check to make sure they re-uploaded correctly
```
zcat /path/to/file.fastq.gz | echo $((`wc -l`/4))
```
4. Re-rerun import code above and check the .qzv file to see if the problem was fixed


















