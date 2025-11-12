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



















