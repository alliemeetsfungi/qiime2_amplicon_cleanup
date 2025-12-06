# Qiime2 Amplicon Sequence Cleanup
All of the code used here is compatible with the package <ins>Qiime2 amplicon version 2024.10.</ins><br>
The instructions and code for installing on a local drive are for macOS (Apple Silicon), and linux for installing on an HPC, but can be applied to other machines with modification.

## Purpose: 
This SOP is designed to assist those with using the Qiime2 package for (i) performing an initial quality control (cleaning) using DADA2 of all sequences by removing low quality sequences (quality filtering) and chimeric sequences, correcting sequencing errors that are present (denoising), grouping duplicate sequences, and merging paired end sequences (if maintaining both the forward and reverse sequences for all samples), and (ii) assigning taxonomy to reamining sequences using various databases. This procedure focuses specifically on fungal 18S and bacterial 16S bacterial sequences, but can be modified for other amplicons as needed.

### Inisghts Into Bioinformatics Terminology:
For the purposes of this SOP, a local drive referes to your personal computer and the High Perfomance Computer (HPC) that is used is the University of Hawaiʻi at Mānoa's cluster named KOA which can be accessed through the online interface [HERE](https://koa.its.hawaii.edu/) or through the command line (described below). In order to get access to the cluster (KOA) you need to be associated with the University of Hawaiʻi at Mānoa, register for an account [HERE](https://datascience.hawaii.edu/eligibility-sign-up/), and take the required onboarding. Once those steps are completed you can access the cluser with your UH username (described below).
<br><br>
You will often see the word "path" which tells you the location of a folder (called a directory) or a file. For example, if you see something like this "path/to/folder/file.ext" this would indicate that the file "file.ext" is inside of the folder "folder" which is inisde of the folder "to" which is inside of the folder "path". The extension of the file (.ext) indicates what kind of file it is, you've seen this before for commone files such as PDF (.pdf) and Microsoft Office (.docx) files that contain different extensions.
<br><br>
## STEP 1: Prepare Files & Environment
<ins>Preparing Files For Qiime2</ins><br>
To begin, make sure sequences are downloaded in an accessible location as fastq.gz files on the local drive.<br>
If you're sequences are in the .fastq format, you can compress each .fastq file in a directory into .fastq.gz files:
```
gzip path/to/fastq/sequence/files/*.fastq
```
If you plan on using the HPC, theses .fast.qz files will need to be uploaded onto the cluster:
```
scp -r "path/to/folder" \ 
user@koa.its.hawaii.edu:/home/user/path/to/directory/for/files
```
### Installing Qiime2 on Local Drive (Apple Silicon)
Further information for downloading Qiime2 on other interfaces can be found on the Qiime2 website found [HERE](https://docs.qiime2.org/2024.10/install/native/). 
<br><br>
Open the terminal and run the code line below.<br>
NOTE: The line after "-n" is what the environment will be named, in the code below it is "qiime2-amplicon-2024.10" but this can be changed to anything! You will use this name to activate Qiime2 on your local drive, **take note of whatever you name it!**
```
CONDA_SUBDIR=osx-64 conda env create -n qiime2-amplicon-2024.10 --file https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.10-py310-osx-conda.yml
conda activate qiime2-amplicon-2024.10
conda config --env --set subdir osx-64
```
<ins>The following output (or very similar) should be printed on the terminal window:</ins>
<br>*Channels:*
<br> *- https://packages.qiime2.org/qiime2/2024.10/amplicon/released*
<br> *- conda-forge*
<br> *- bioconda*
<br> *- defaults*
<br> *- mamba*
<br>*Platform: osx-64*
<br>*Collecting package metadata (repodata.json): done*
<br>*Solving environment: done*
<br><br>*Downloading and Extracting Packages:*
*openjdk-22.0.1       | 168.9 MB  | #####################################################################################4                   |  82%*
<br><br>*Preparing transaction: done*
<br>*Verifying transaction: done*
<br>*Executing transaction: done*
<br><br>*To activate this environment, use*
<br><br>*$ conda activate qiime2-amplicon-2024.10*
<br><br>*To deactivate an active environment, use*
<br><br> *$ conda deactivate*
<br><br><ins>If nothing appears, run the code below and re-run the installation code</ins>
```
conda config --set channel_priority flexible
```
Once the above output is shown in the terminal, Qiime2 can be activated in terminal command line by running:
```
conda activate /path/to/qiime2/on/local/drive/qiime2-amplicon-2024.10
```
Once you have activated Qiime2, set your working directory along the same path to the directory containing the gzipped sequence files (.fastq.gz). For example, if your sequences are found in /home/project/sequences, set your working directory to /home/project.
```
cd /path/to/working/directory
```

### Installing Qiime2 on the HPC
To access KOA on the command line run the code below, then enter your UH password & designate two factor authentication preference. NOTE: password is invisible and does not show key strokes!
```
ssh user@koa.its.hawaii.edu
```
In order to download and activate packages you must first start an interactive job, here is an example of how to do that
```
srun -p shared --mem=60G -c 4 -t 06:00:00 --pty /bin/bash
```
<br><ins>Stay on your home directory for Qiime2 installation</ins>
<br>Load anaconda module (this is already installed on KOA for all users)
```
module load lang/Anaconda3/2024.02-1
```
Now you can install Qiime2.<br>
NOTE: the line after -n is what the environment will be named, in the code below it is "qiime2" but could be anything. You will use this name to activate Qiime2 on the HPC, so **take note of whatever you name it!**
``` 
conda env create -n qiime2 --file https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.10-py310-linux-conda.yml
```
<br>
Check the conda environment to make sure it installed correctly by looking for what you named your Qiime2 package during installation.
```
conda info - e
```
Here is an example of what is printed on my command line when I run the code above:<br>
conda environments:<br>
qiime2    /home/alliej/.conda/envs/qiime2<br>
base      /opt/apps/software/lang/Anaconda3/2024.02-1<br><br>
<ins>Activating Qiime2 on HPC</ins>
<br>NOTE: this is different than how conda is activated on a local drive or personal computer!
<br>For each new session, make sure you perform the following prior to activating Qiime2:
  1. Start an interactive job
  2. Load anaconda module
```
source activate qiime2
```
Set working directory along the path to where the directory where the sequences are stored.<br>
For example, if your sequences are found in /home/project/sequences, set your working directory to /home/project
```
cd /path/to/working/directory
```

## STEP 2: Importing Sequences Into Qiime2
Instructions on importing sequences into a qiime2 artifact can be found [HERE](https://docs.qiime2.org/2024.10/tutorials/importing/).
<br>Qiime2 visualization files (.qzv) can be viewed [HERE](https://view.qiime2.org/?src=e96f979f-4cc6-46fc-800f-abe58740e4ea).
<br><br>WARNING: This can take upwards of 2-10 hours depending on how large your data set is!
<br><br>
### Import paired-end sequences using Casava 1.8 paired-end demultiplexed fastq method
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
<br><br>
### Import Forward sequences using Casava 1.8 single-end demultiplexed fastq method
<br><ins>Copy all foward sequences into new directory</ins>
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
<br><br>
## STEP 3: Trim Primers From Sequences
This section uses cutadapt, the handbook can be found [HERE](https://docs.qiime2.org/2024.10/plugins/available/cutadapt/index.html).
<br>For background on trimming Golay barcodes see [THIS](https://forum.qiime2.org/t/cutadapt-adapter-vs-front/15450) forum page.

**Primers commonly used in our studies:**
<br><ins>Bacterial small ribosomal subunit (16S) V4 amplicon using 515F/806R primers</ins>. Amplified region is ~
  <br>515F (Forward primer): 5′-GTGYCAGCMGCCGCGGTAA-3′
  <br>806R (Reverse primer): 5′-GGACTACNVGGGTWTCTAAT-3′
<br><br><ins>Arbuscular Mycorrhizal Fungi (AMF) small ribosomal subunit (18s) V4 amplicon using WANDA/AML2 primers</ins>. See Kacie Kajihara's paper [HERE](https://nph.onlinelibrary.wiley.com/doi/10.1111/nph.18058).
  <br>WANDA (Forward Primer): 5′-GAAACTGCGAATGGCTC-3′
  <br>AML2 (Reverse Primer): 5′-GAACCCAAACACTTTGGTTTCC-3′
<br><br><ins>Fungi small ribosomal subunit (18s) V3-V4 amplicon using 18S-82F/Euk-516r primers</ins>. See Jason Baer's paper [HERE](https://academic.oup.com/ismej/article/19/1/wraf228/8284954#supplementary-data).
  <br>18S-82F (Forward Primer): 5′-GAAACTGCGAATGGCTC-3′
  <br>Euk-516R (Reverse Primer): 5′-ACCAGACTTGCCCTCC-3′
<br><br><ins>Orchid Mycorrhizal Fungi (OMF) Internal Transcribed Spacer 2 (ITS2) amplicon using fITS7 paired with either Tul1F or Tul2F</ins>. See Wang et al. 2023 [HERE](https://nph.onlinelibrary.wiley.com/doi/full/10.1111/nph.19385).
  <br>fITS7 (Forward Primer): 5′-GTGARTCATCGAATCTTTG-3′
  <br>Tul1F (Reverse Primer): 5′-CGTYGGATCCCTYGGC-3′
  <br>Tul2F (Reverse Primer): 5′-TGGATCCCTTGGCACGTC-3′
<br><br>
### Trim Primers From Paired-end Sequences
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
<br><br>
### Trim Primers From Forward Sequences
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
<br><br>If running multiple tests on the same set of sequences, single end and paired-end outputs should have identical parameters.
<br><br>
## STEP 4: DADA2: Trimming, Merging, Denoising, and Feature Calling of Sequences
To assure that the most features will be detected, multiple tests will be run at this step based on post primer trimming quality plots.
<br><br>This process will produce three Qiime2 artifacts:
1. <ins>Feature Table</ins> (feature-table.qza)
2. <ins>Representative Sequences</ins> of detected features (feature-rep-seqs.qza)
3. <ins>Denoising Statistics</ins> that shows the breakdown of the reads which passed each cleaning step (filtering, denoising, merging, and chimeras) for every sample (denoising-stats.qza)
<br><br>
### Cleaning and Merging Paired-end Sequences
<br><ins>Test 1: Maintain Entire Sequence length</ins>
Setting truncation length to 300 should maintain the entire sequence for both foward and reverse reads.
```
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs path/to/primer/trimmed/file-name.qza \
  --p-trunc-len-f 300 \
  --p-trunc-len-r 300 \
  --o-table path/to/results/directory/feature-table.qza \
  --o-representative-sequences path/to/results/directory/feature-rep-seqs.qza \
  --o-denoising-stats path/to/results/directory/denoising-stats.qza \
  --verbose
```

<br><br>Convert feature table Qiime2 artifact (feature-table.qza) into visualization file (feature-table.qzv)
```
qiime feature-table summarize \
  --i-table path/to/results/directory/feature-table.qza \
  --o-visualization path/to/results/directory/feature-table.qzv
```
Record the number of samples, features, and total frequency (reads)
<br><br>Convert representative sequences Qiime2 artifcat (feature-rep-seqs.qza) into a visualization file (feature-rep-seqs.qzv)
```
qiime feature-table tabulate-seqs \
  --i-data path/to/results/directory/feature-rep-seqs.qza \
  --o-visualization path/to/results/directory/feature-rep-seqs.qzv
```
Record sequence count (features), and sequence length statistics.
<br>Additionally, you can download a .fasta file containing the sequence associated with each feature detected.
<br><br>Convert denoising statistics Qiime2 artifact (denoising-stats.qza) into visualization file (denoising-stats.qzv)
```
qiime metadata tabulate \
  --m-input-file path/to/results/directory/denoising-stats.qza \
  --o-visualization path/to/results/directory/denoising-stats.qzv
```
This table summarizes each samples sequencing reads that were able to pass the cleaning steps performed in DADA2. 
<br>Record how many samples were able to retain an acceptable amount (atleast 25%) of their reads. Additionally, you can download the entire table for referencing later.

<br><ins>Test 2: Trim sequences based on read quality</ins>
Based on the quality plots produced in STEP 3 (post primer trimming paired-end sequences) set truncation length based on where the quality score begins to abrubtly drop below 20. 
```
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs path/to/primer/trimmed/file-name.qza \
  --p-trunc-len-f 280 \
  --p-trunc-len-r 230 \
  --o-table path/to/results/directory/feature-table.qza \
  --o-representative-sequences path/to/results/directory/feature-rep-seqs.qza \
  --o-denoising-stats path/to/results/directory/denoising-stats.qza \
  --verbose
```
<br><br>Convert feature table Qiime2 artifact (feature-table.qza) into visualization file (feature-table.qzv)
```
qiime feature-table summarize \
  --i-table path/to/results/directory/feature-table.qza \
  --o-visualization path/to/results/directory/feature-table.qzv
```
Record the number of samples, features, and total frequency (reads)
<br><br>Convert representative sequences Qiime2 artifcat (feature-rep-seqs.qza) into a visualization file (feature-rep-seqs.qzv)
```
qiime feature-table tabulate-seqs \
  --i-data path/to/results/directory/feature-rep-seqs.qza \
  --o-visualization path/to/results/directory/feature-rep-seqs.qzv
```
Record sequence count (features), sequence length statistics, and download .fasta file.
<br><br>Convert denoising statistics Qiime2 artifact (denoising-stats.qza) into visualization file (denoising-stats.qzv)
```
qiime metadata tabulate \
  --m-input-file path/to/results/directory/denoising-stats.qza \
  --o-visualization path/to/results/directory/denoising-stats.qzv
```
<br>Record the number of samples that retained at least 25% of their reads, and download entire metadata table.

<br><br>
### Cleaning Forward Sequences
<br><ins>Test 3: Trim forward sequences conservatively</ins>
<br>Choose to trim sequences shorter in order to avoid poor quality reads 
```
qiime dada2 denoise-single \
  --i-demultiplexed-seqs path/to/primer/trimmed/file-name.qza \
  --p-trunc-len 270 \
  --o-table path/to/results/directory/feature-table.qza \
  --o-representative-sequences path/to/results/directory/feature-rep-seqs.qza \
  --o-denoising-stats path/to/results/directory/denoising-stats.qza \
  --verbose
```
<br><br>Convert feature table Qiime2 artifact (feature-table.qza) into visualization file (feature-table.qzv)
```
qiime feature-table summarize \
  --i-table path/to/results/directory/feature-table.qza \
  --o-visualization path/to/results/directory/feature-table.qzv
```
Record the number of samples, features, and total frequency (reads)
<br><br>Convert representative sequences Qiime2 artifcat (feature-rep-seqs.qza) into a visualization file (feature-rep-seqs.qzv)
```
qiime feature-table tabulate-seqs \
  --i-data path/to/results/directory/feature-rep-seqs.qza \
  --o-visualization path/to/results/directory/feature-rep-seqs.qzv
```
Record sequence count (features), sequence length statistics, and download .fasta file.
<br><br>Convert denoising statistics Qiime2 artifact (denoising-stats.qza) into visualization file (denoising-stats.qzv)
```
qiime metadata tabulate \
  --m-input-file path/to/results/directory/denoising-stats.qza \
  --o-visualization path/to/results/directory/denoising-stats.qzv
```
<br>Record the number of samples that retained at least 25% of their reads, and download entire metadata table.
<br><br><ins>Test 4: Trim forward sequences liberally</ins>
<br>Choose to keep sequences longer with the risk of maintaining some poor quality reads
```
qiime dada2 denoise-single \
  --i-demultiplexed-seqs path/to/primer/trimmed/file-name.qza \
  --p-trunc-len 280 \
  --o-table path/to/results/directory/feature-table.qza \
  --o-representative-sequences path/to/results/directory/feature-rep-seqs.qza \
  --o-denoising-stats path/to/results/directory/denoising-stats.qza \
  --verbose
```
<br><br>Convert feature table Qiime2 artifact (feature-table.qza) into visualization file (feature-table.qzv)
```
qiime feature-table summarize \
  --i-table path/to/results/directory/feature-table.qza \
  --o-visualization path/to/results/directory/feature-table.qzv
```
Record the number of samples, features, and total frequency (reads)
<br><br>Convert representative sequences Qiime2 artifcat (feature-rep-seqs.qza) into a visualization file (feature-rep-seqs.qzv)
```
qiime feature-table tabulate-seqs \
  --i-data path/to/results/directory/feature-rep-seqs.qza \
  --o-visualization path/to/results/directory/feature-rep-seqs.qzv
```
Record sequence count (features), sequence length statistics, and download .fasta file.
<br><br>Convert denoising statistics Qiime2 artifact (denoising-stats.qza) into visualization file (denoising-stats.qzv)
```
qiime metadata tabulate \
  --m-input-file path/to/results/directory/denoising-stats.qza \
  --o-visualization path/to/results/directory/denoising-stats.qzv
```
<br>Record the number of samples that retained at least 25% of their reads, and download entire metadata table.
<br><br><ins>Compare the number of features and number of samples with sufficient read retention among all tests. The test with the highest amount of features and samples with sufficient read retention should be used from this point forward.

<br><br>
## STEP 5 (OPTIONAL!): Cluster ASVs into OTUs
Use the feature table and representative sequences from the DADA2 cleanup with the best feature count and sample read retention outcome.
<br><br>Cluster sequences into 97% identity
```
qiime vsearch cluster-features-de-novo \
  --i-table path/to/results/directory/feature-table.qza \
  --i-sequences path/to/results/directory/feature-rep-seqs.qza \
  --p-perc-identity 0.97 \
  --o-clustered-table path/to/results/directory/otu-feature-table.qza \
  --o-clustered-sequences path/to/results/directory/otu-feature-rep-seqs.qza \
  --verbose
```
Convert feature table artifact (otu-feature-table.qza) into visualization file (otu-feature-table.qzv)
```
qiime feature-table summarize \
  --i-table path/to/results/directory/otu-feature-table.qza \
  --o-visualization path/to/results/directory/otu-feature-table.qzv
```
Record number of features and sequencing reads
<br><br>Convert artifact containing representative sequences (otu-feature-rep-seqs.qza) into visualization file (otu-feature-rep-seqs.qzv)
```
qiime feature-table tabulate-seqs \
  --i-data path/to/results/directory/otu-feature-rep-seqs.qza \
  --o-visualization path/to/results/directory/otu-feature-rep-seqs.qzv
```
Download .fasta file containing feature representative sequences

<br><br>
## STEP 6: Export Feature Table For Culling
The resulting files can be imported into R for initial culling assesment
<br><br>Export cleaned feature table to new directory
```
qiime tools export \
  --input-path path/to/results/directory/feature-table.qza \
  --output-path path/to/results/directory/feature-table-directory
```
Convert exported feature table into a .tsv
```
biom convert \
  --input-fp path/to/results/directory/feature-table-directory/feature-table.biom \
  --output-fp path/to/results/directory/feature-table-directory/feature-table.tsv \
  --to-tsv
```
Convert .tsv to .csv
```
sed 's/\t/,/g' path/to/results/directory/feature-table-directory/feature-table.tsv > path/to/results/directory/feature-table-directory/feature-table.csv
```
<br><br>Download final table in new command line window on local drive (not signed into KOA).
```
scp user@koa.its.hawaii.edu:/home/user/path/to/results/directory/feature-table-directory/feature-table.csv \
path/to/local/drive/directory
```
<br><br>
## STEP 7: Import Databases For Taxonomic Identification
The following databases are the most commonly used for our studies. Some require files to be downloaded from the database webpage, while other databases can be pulled directly through Qiime2.
<br><br><ins>SILVA</ins>
<br>This database is used for both small ribosomal subunits 16S & 18S
<br><br>Pull database files straight from silva database, see [THIS](https://forum.qiime2.org/t/sequence-and-taxonomy-files-for-silva-v138-2/33475) forum for more details. The code here is based on version 138.2, and reference sequences extracted are in rRNA format and need to be transcribed before using for taxonomic identification.
<br><br>Pull data from silva repository
```
qiime rescript get-silva-data \
  --p-version '138.2' \
  --p-target 'SSURef_NR99' \
  --o-silva-sequences desired/path/to/files/silva-138.2-rna-ref-seqs.qza \
  --o-silva-taxonomy desired/path/to/files/silva-138.2-ref-tax.qza
```
Reverse transcribe reference sequences (rRNA) for assignment compatibility (rDNA)
```
qiime rescript reverse-transcribe \
  --i-rna-sequences path/to/file/silva-138.2-rna-ref-seqs.qza \
  --o-dna-sequences desired/path/to/file/silva-138.2-ref-seqs.qza
```
### Fungal Databases
<br><ins>Eukaryome</ins>
<br>Files can be downloaded [HERE](https://eukaryome.org/qiime2/) for Qiime2 compatability. This code is based on QIIME2_EUK_SSU Version 2.0, however databases for LSU, ITS, and other packages are available.
<br><br>Transfer downloaded files into working directory
<br><br>Import reference sequences
```
qiime tools import --type 'FeatureData[Sequence]' \
  --input-path path/to/file/QIIME2_EUK_SSU_v2.0.fasta \
  --output-path desired/path/to/file/eukaryome-ref-seqs.qza
```
Import taxonomy
```
qiime tools import --type 'FeatureData[Taxonomy]' \
  --input-format HeaderlessTSVTaxonomyFormat \
  --input-path path/to/file/QIIME2_EUK_SSU_v2.0.tsv \
  --output-path desired/path/to/file/eukaryome-ref-tax.qza
```
<br><br>
<ins>National Center for Biotechnology Information (NCBI)</ins>
<br>Files can be pulled straight from NCBI using Qiime2 and are reliant on how often the database is updated remotely
```
qiime rescript get-ncbi-data \
  --p-query '18S[ALL] AND fungi[ORGN]' \
  --o-sequences desired/path/to/file/ncbi-fungi-ref-seqs.qza \
  --o-taxonomy desired/path/to/file/ncbi-fungi-ref-tax.qza \
  --p-n-jobs 5
```
You can alter the query to filter for specific taxa, such as Glomeromycotina
```
qiime rescript get-ncbi-data \
--p-query '18S[ALL] AND glomeromycotina [ORGN]' \
--o-sequences desired/path/to/file/ncbi-AMF-ref-seqs.qza \
--o-taxonomy desired/path/to/file/ncbi-AMF-ref-tax.qza \
--p-n-jobs 5

```
<br><br><ins>MaarjAM</ins>
<br>Qiime2 compatable files can be downloaded [HERE](https://maarjam.ut.ee/?action=bDownload). This code is based on MaarjAM VT sequences of 18S rDNA gene region QIIME release (2021) for identifying Arbuscular Mycorrhizal Fungi (AMF).
<br><br>Transfer downloaded files into working directory
<br>Import reference sequences
```
qiime tools import --type 'FeatureData[Sequence]' \
--input-path path/to/file/maarjam_database_SSU.qiime.fasta \
--output-path desired/path/to/file/maarjam-ref-seqs.qza
```
Import taxonomy
```
qiime tools import --type 'FeatureData[Taxonomy]' \
--input-format HeaderlessTSVTaxonomyFormat \
--input-path path/to/file/maarjam_database_SSU.qiime.txt \
--output-path desired/path/to/file/maarjam-ref-tax.qza
```

<br><br><ins>UNITE</ins>

### Bacterial Databases
<ins>Genome Taxonomy Database (GTDB)</ins>
<br>Database website can be found [HERE](https://gtdb.ecogenomic.org/). Sequences can be pulled using Qiime2 with the code below based on GTDP Version 220.0
```
qiime rescript get-gtdb-data \
  --p-version 220.0 \
  --p-domain Bacteria \
  --output-dir desired/directory/path/for/files
```
<br>

## STEP 8: Taxonomic Assignment To Features
Using the qiime artifact created by DADA2 (feature-rep-seqs.qza) containing the sequences associated with each identified feature (representative sequences) as the input file for BLAST searching each features taxonomic association.
<br><br>For each database, run unnassigned sequences with 95%, 90%, and 80% identity rates to allow for higher rates of taxonomic assignment by loosening the sequence consensus calling requiremnts. If multiple databases are available, start with BLAST searching the sequences through the databse believed to provide the most assignments at all three query coverage targets first, then running the remainder of the unassigned sequences through the next best databse.
<br><br><ins>Database ONE</ins>
<br>*95% Identity*
<br>First blast search will be run with all parameters at the default (NEED TO CHECK THIS), except setting the percent identity (--p-perc-identity) to 95% (0.95)
```
qiime feature-classifier classify-consensus-blast \
  --i-query path/to/dada2/output/file/feature-rep-seqs.qza \ #Your representative sequences post DADA2
  --i-reference-reads path/to/database/ref-seqs.qza \        #Database Reference Sequences
  --i-reference-taxonomy path/to/database/ref-tax.qza \      #Database Reference Taxa
  --p-maxaccepts 1 \                                         #Default value
  --p-perc-identity 0.95 \                                   #Percent identity is reduced in the next query
  --p-query-cov 0.90 \                                       #Default value
  --p-strand both \                                          #Default
  --p-evalue 1e-50 \                                         #Default value
  --p-min-consensus 0.51 \                                   #Default value
  --output-dir path/to/search/results/database/directory-95  #Makes new directory containing multiple files, will err if you pre-make it!
```
Filter out unassigned sequences into their own file
```
qiime taxa filter-seqs \
  --i-sequences path/to/dada2/output/file/feature-rep-seqs.qza \                              #Input is the file you want to be filtered (input file for BLAST search feature-rep-seqs.qza file)
  --i-taxonomy path/to/search/results/database/directory-95/classification.qza \              #The classification.qza file found in the classifier output directory
  --p-include unassigned \                                                                    #Key word for what you are filtering for (i.e, keeping)
  --o-filtered-sequences path/to/search/results/database/directory-95/unassigned-rep-seqs.qza #Remaining sequences that did not have taxa assigned under these BLAST search parameteres
```
<br>*90% Identity*
```
qiime feature-classifier classify-consensus-blast \
  --i-query path/to/search/results/database/directory-95/unassigned-rep-seqs.qza \ #Change to ouput file from the previous filtered sequences that remain unassigned
  --i-reference-reads path/to/database/ref-seqs.qza \                              #Stays the same
  --i-reference-taxonomy path/to/database/ref-tax.qza \                            #Stays the same
  --p-maxaccepts 1 \
  --p-perc-identity 0.90 \                                                         #Reduce percent identity to 90%
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir path/to/search/results/database/directory-90
```
Filter out remaining unassigned sequences from this query search into their own file
```
qiime taxa filter-seqs \
  --i-sequences path/to/search/results/database/directory-95/unassigned-rep-seqs.qza \        #Use the input file for the BLAST search (unassigned-rep-seqs.qza)
  --i-taxonomy path/to/search/results/database/directory-90/classification.qza \              #Input the resulting classification.qza file
  --p-include unassigned \
  --o-filtered-sequences path/to/search/results/database/directory-90/unassigned-rep-seqs.qza #Remaining sequences that did not have taxa assigned under these search parameteres
```
<br>*80% Identity*
```
qiime feature-classifier classify-consensus-blast \
  --i-query path/to/search/results/database/directory-90/unassigned-rep-seqs.qza \ #Change to ouput file from the previous filtered sequences that remain unassigned
  --i-reference-reads path/to/database/ref-seqs.qza \
  --i-reference-taxonomy path/to/database/ref-tax.qza \
  --p-maxaccepts 1 \
  --p-perc-identity 0.80 \                                                          #Reduce percent identity to 80%
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir path/to/search/results/database/directory-80                         #Change name of directory

qiime taxa filter-seqs \
  --i-sequences path/to/search/results/database/directory-90/unassigned-rep-seqs.qza \
  --i-taxonomy path/to/search/results/database/directory-80/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences path/to/search/results/database/directory-80/unassigned-rep-seqs.qza
```
Run remaining unassignd sequences through the next databse!
<br><br><ins>Database TWO</ins>
<br>*95% Identity*
At this step, you need to change the path for reference sequences and taxa to the files associated with the new databases reference sequences and taxa
The remaining unassigned representative sequences file from the previous database should be used for --i-query (unassigned-rep-seqs.qza)
```
qiime feature-classifier classify-consensus-blast \
  --i-query path/to/search/results/database/directory-80/unassigned-rep-seqs.qza \ #Remaining unnasigned representative sequences from first database
  --i-reference-reads path/to/NEW-database/ref-seqs.qza \                          #NEW database Reference Sequences
  --i-reference-taxonomy path/to/NEW-database/ref-tax.qza \                        #NEW database Reference Taxa
  --p-maxaccepts 1 \
  --p-perc-identity 0.95 \
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir path/to/search/results/NEW-database/directory-95

qiime taxa filter-seqs \
  --i-sequences path/to/search/results/database/directory-80/unassigned-rep-seqs.qza \
  --i-taxonomy path/to/search/results/NEW-database/directory-95/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences path/to/search/results/NEW-database/directory-95/unassigned-rep-seqs.qza
```
<br>Continue on to run the remaining unassigned representative sequences at 90% and 80% identities for this database, the same as you did for the first database.
```
qiime feature-classifier classify-consensus-blast \
  --i-query path/to/search/results/NEW-database/directory-95/unassigned-rep-seqs.qza \
  --i-reference-reads path/to/NEW-database/ref-seqs.qza \
  --i-reference-taxonomy path/to/NEW-database/ref-tax.qza \
  --p-maxaccepts 1 \
  --p-perc-identity 0.90 \
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir path/to/search/results/NEW-database/directory-90

qiime taxa filter-seqs \
  --i-sequences path/to/search/results/NEW-database/directory-95/unassigned-rep-seqs.qza \
  --i-taxonomy path/to/search/results/NEW-database/directory-90/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences path/to/search/results/NEW-database/directory-90/unassigned-rep-seqs.qza


qiime feature-classifier classify-consensus-blast \
  --i-query path/to/search/results/NEW-database/directory-90/unassigned-rep-seqs.qza \
  --i-reference-reads path/to/NEW-database/ref-seqs.qza \
  --i-reference-taxonomy path/to/NEW-database/ref-tax.qza \
  --p-maxaccepts 1 \
  --p-perc-identity 0.80 \
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir path/to/search/results/NEW-database/directory-80

qiime taxa filter-seqs \
  --i-sequences path/to/search/results/NEW-database/directory-90/unassigned-rep-seqs.qza \
  --i-taxonomy path/to/search/results/NEW-database/directory-80/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences path/to/databse/search/results/NEW-database/directory-80/unassigned-rep-seqs.qza
```
If you have more databases you are interested in running your representative sequences through for furhter taxonomic assignment, repeat these steps for each additional database. See (INSERT MY OWN PIPELINES HERE) for reference on using multiple databases for taxonomic assignment with a real dataset. 
<br>
## STEP 9: Filtering Feature Tables (OPTIONAL)
Once you are satisfied with your taxonomic assignments using your representative sequences, you can filter your actual feature table to make tables that contain only taxonomically assigned features, or conversely only the remaining unassigned features.
<br>
<ins>Database ONE</ins>
Begin with your original DADA2 feature table as your input table (feature-table.qza) and your classification file made from your first database at 95% identity. 
<br>
*Filtering for assigned features*
```
qiime taxa filter-table \
  --i-table path/to/results/directory/feature-table.qza \                            #Your feature table from DADA2
  --i-taxonomy path/to/search/results/database/directory-95/classification.qza \     #Classification file made from the first taxonomic query run
  --p-exclude unassigned \                                                           #This will keep all features EXCEPT those that are annotated as unassigned
  --o-filtered-table path/to/search/results/database/directory-95/assigned-table.qza #Desired output path of table
```
*Filtering for unassigned features*
```
qiime taxa filter-table \
  --i-table path/to/results/directory/feature-table.qza \                            #Your feature table from DADA2
  --i-taxonomy path/to/search/results/database/directory-95/classification.qza \     #Classification file made from the first taxonomic query you ran 
  --p-include unassigned \                                                           #This will keep only features annotated as unassigned
  --o-filtered-table path/to/search/results/database/directory-95/unassigned-table.qza #Output path of table
```
Using the resulting unassigned table as your input table, repeat these steps for all additional taxonomic query searches you performed in the same order in which they were run.
<br>
*Database ONE 90% Identity*
```
qiime taxa filter-table \
  --i-table path/to/search/results/database/directory-95/unassigned-table.qza \  #Your unassigned tabled made in the last step
  --i-taxonomy path/to/search/results/database/directory-90/classification.qza \ #Classification file made from the second taxonomic query run
  --p-exclude unassigned \
  --o-filtered-table path/to/search/results/database/directory-90/assigned-table.qza

qiime taxa filter-table \
  --i-table path/to/search/results/database/directory-95/unassigned-table.qza \  #Your unassigned tabled made in the last step
  --i-taxonomy path/to/search/results/database/directory-90/classification.qza \ #Classification file made from the second taxonomic query run
  --p-include unassigned \
  --o-filtered-table path/to/search/results/database/directory-90/unassigned-table.qza
```
*Database ONE 80% Identity*
```
qiime taxa filter-table \
  --i-table path/to/search/results/database/directory-90/unassigned-table.qza \
  --i-taxonomy path/to/search/results/database/directory-80/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table path/to/search/results/database/directory-80/assigned-table.qza

qiime taxa filter-table \
  --i-table path/to/search/results/database/directory-90/unassigned-table.qza \
  --i-taxonomy path/to/search/results/database/directory-80/classification.qza \
  --p-include unassigned \
  --o-filtered-table path/to/search/results/database/directory-80/unassigned-table.qza
```
Repeat these steps for the second database query results in sequence.
```
# Database TWO 95% identity
qiime taxa filter-table \
  --i-table path/to/search/results/database/directory-80/unassigned-table.qza \
  --i-taxonomy path/to/search/results/NEW-database/directory-95/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table path/to/search/results/NEW-database/directory-95/assigned-table.qza

qiime taxa filter-table \
  --i-table path/to/search/results/database/directory-80/unassigned-table.qza \
  --i-taxonomy path/to/search/results/NEW-database/directory-95/classification.qza \
  --p-include unassigned \
  --o-filtered-table path/to/search/results/NEW-database/directory-95/unassigned-table.qza

# Database TWO 90% identity
qiime taxa filter-table \
  --i-table path/to/search/results/NEW-database/directory-95/unassigned-table.qza \
  --i-taxonomy path/to/search/results/NEW-database/directory-90/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table path/to/search/results/NEW-database/directory-90/assigned-table.qza

qiime taxa filter-table \
  --i-table path/to/search/results/NEW-database/directory-95/unassigned-table.qza \
  --i-taxonomy path/to/search/results/NEW-database/directory-90/classification.qza \
  --p-include unassigned \
  --o-filtered-table path/to/search/results/NEW-database/directory-90/unassigned-table.qza

# Database TWO 80% identity
qiime taxa filter-table \
  --i-table path/to/search/results/NEW-database/directory-90/unassigned-table.qza \
  --i-taxonomy path/to/search/results/NEW-database/directory-80/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table path/to/search/results/NEW-database/directory-80/assigned-table.qza

qiime taxa filter-table \
  --i-table path/to/search/results/NEW-database/directory-90/unassigned-table.qza \
  --i-taxonomy path/to/search/results/NEW-database/directory-80/classification.qza \
  --p-include unassigned \
  --o-filtered-table path/to/search/results/NEW-database/directory-80/unassigned-table.qza
```
<br>Since the feature tables containing assigned taxa are made by subsetting the original table, they are found in multiple different tables. In order to retrieve a single feature table containing all of the assigned taxonomic features, all of the individual feature tables containing the assigned taxa will need to be merged. This does not need to be performed for the unassigned tables since you filter out the assigned taxa as you go rather than subset for them, making the final unassigned table your complete feature table containing all of the unassigned features. I like to make a copy of the final unassigned feature table, rename it along the lines of "final-unassigned-feature-table.qza", and move this copy to my final results folder for easier access, but this is strictly preference.
WHAT IS THE POINT OF THIS????
NOTE: Tables must be listed from largest to smallest in order for the merge to actually work. Generally, your first search query table will be the largest and your last will be the smallest.
```
qiime feature-table merge \
  --i-tables path/to/search/results/database/directory-95/assigned-table.qza \
  --i-tables path/to/search/results/database/directory-90/assigned-table.qza \
  --i-tables path/to/search/results/database/directory-80/assigned-table.qza \
  --i-tables path/to/search/results/NEW-database/directory-95/assigned-table.qza \
  --i-tables path/to/search/results/NEW-database/directory-90/assigned-table.qza \
  --i-tables path/to/search/results/NEW-database/directory-80/assigned-table.qza \
  --o-merged-table path/to/final/results/directory/all-assigned-features-table.qza \
  --p-overlap-method sum
```
Convert both the final assigned and unassigned feature tbale Qiime2 artifacts into a visualization files.
```
# Final assigned feature table
qiime feature-table summarize \
  --i-table path/to/final/results/directory/all-assigned-features-table.qza \
  --o-visualization path/to/final/results/directory/all-assigned-features-table.qzv

# Final unassigned feature table
qiime feature-table summarize \
  --i-table path/to/search/results/NEW-database/directory-80/unassigned-table.qza \
  --o-visualization path/to/final/results/directory/all-unassigned-features-table.qzv
```
Upload this onto the Qiime2 viewer and record the number of samples, features, and reads accounted for in both feature tables.
<br><br>
As a gut check, I like to merge both my assigned and unassigned feature tables to make sure that the sample, feature, and read counts match the original feature table from DADA2 (feature-table.qza). If the counts don't align, something went amiss when merging your assigned feature tables.
```
qiime feature-table merge \
  --i-tables path/to/final/results/directory/all-assigned-features-table.qza \
  --i-tables path/to/search/results/NEW-database/directory-80/unassigned-table.qza \
  --o-merged-table path/to/final/results/directory/final-all-features-table.qza \
  --p-overlap-method sum

qiime feature-table summarize \
  --i-table path/to/final/results/directory/final-all-features-table.qza \
  --o-visualization path/to/final/results/directory/final-all-features-table.qzv
```
Upload the visualization file and confirm that the sample, feature, and read counts match that of the feature-table.qza produced from DADA2.

## STEP 10: Merging Classification Files
To retrieve your final taxa table, you will need to merge all of your classification files which can be a bit tricky. Before doing this, I like to determine how many features were annotated within each classfication file to make sure that merging is occurring correctly. To do this, all of the clalssification.qza Qiime2 artifacts will need to be converted into a visualization file.
```
qiime metadata tabulate \
  --m-input-file path/to/search/results/database/directory-95/classification.qza \
  --o-visualization path/to/search/results/database/directory-95/classification.qzv

qiime metadata tabulate \
  --m-input-file path/to/search/results/database/directory-90/classification.qza \
  --o-visualization path/to/search/results/database/directory-90/classification.qzv

qiime metadata tabulate \
  --m-input-file path/to/search/results/database/directory-80/classification.qza \
  --o-visualization path/to/search/results/database/directory-80/classification.qzv

qiime metadata tabulate \
  --m-input-file path/to/search/results/NEW-database/directory-95/classification.qza \
  --o-visualization path/to/search/results/NEW-database/directory-95/classification.qzv

qiime metadata tabulate \
  --m-input-file path/to/search/results/NEW-database/directory-90/classification.qza \
  --o-visualization path/to/search/results/NEW-database/directory-90/classification.qzv

qiime metadata tabulate \
  --m-input-file path/to/search/results/NEW-database/directory-80/classification.qza \
  --o-visualization path/to/search/results/NEW-database/directory-80/classification.qzv
```
Upload each visualization file and record the number of features present
<br><br>
<ins>Merging Classification Tables</ins>
<br>When mergering classification tables, it is essential that the classification table with the least amount of classified taxa is listed as the first input table. Otherwise, the output table will come out only containing the taxa from the classification file with the most taxa present.
```
qiime feature-table merge-taxa \
  --i-data path/to/search/results/database/directory-90/classification.qza \
  --i-data path/to/search/results/database/directory-95/classification.qza \
  --o-merged-data path/to/search/results/database/classification_merged1.qza
```
Then check to make sure they merged by converting the output Qiime2 artifact into a visualization file, uploading it onto the Qiime2 viewer, and observing if the number of features that are present is equal to the sum of both of the classification files you merged.
```
qiime metadata tabulate \
  --m-input-file path/to/search/results/database/classification_merged1.qza \
  --o-visualization path/to/search/results/database/classification_merged1.qzv
```
Repeat these steps until all of the classification artifacts are merged. There is multiple ways to do this, I prefer to mkae multiple merge files that I then merge together at the end. The code below represents this approach but can be modified for preference
```
qiime feature-table merge-taxa \
  --i-data path/to/search/results/NEW-database/directory-95/classification.qza \
  --i-data path/to/search/results/database/directory-80/classification.qza \
  --o-merged-data path/to/search/results/database/classification_merged2.qza

# Convert and check feature counts make sense
qiime metadata tabulate \
  --m-input-file path/to/search/results/database/classification_merged2.qza \
  --o-visualization path/to/search/results/database/classification_merged2.qzv

qiime feature-table merge-taxa \
  --i-data path/to/search/results/NEW-database/directory-90/classification.qza \
  --i-data path/to/search/results/NEW-database/directory-80/classification.qza \
  --o-merged-data path/to/search/results/database/classification_merged3.qza

qiime metadata tabulate \
  --m-input-file path/to/search/results/database/classification_merged3.qza \
  --o-visualization path/to/search/results/database/classification_merged3.qzv

# Merge first two sets of merged classification artifacts
qiime feature-table merge-taxa \
  --i-data path/to/search/results/database/classification_merged2.qza \
  --i-data path/to/search/results/database/classification_merged1.qza \
  --o-merged-data path/to/search/results/database/classification_merged4.qza

qiime metadata tabulate \
  --m-input-file path/to/search/results/database/classification_merged4.qza \
  --o-visualization path/to/search/results/database/classification_merged4.qzv

# Merge final tables together to make one completely merged classification artifact
qiime feature-table merge-taxa \
  --i-data path/to/search/results/database/classification_merged3.qza \
  --i-data path/to/search/results/database/classification_merged4.qza \
  --o-merged-data path/to/search/results/database/classification_merged_final.qza

qiime metadata tabulate \
  --m-input-file path/to/search/results/database/classification_merged_final.qza \
  --o-visualization path/to/final/results/directory/classification_merged_final.qzv
```
NOTE: The directory for the final merged table is set to be where you want all of your final results to be for easy organization

## STEP 11: Export Final Tables And Representative Sequences
Now we have all of our final tables ready to export for furthing cleaning by asv & sample culling and decontam runs.
<br>
*Export Feature (ASV) Table (feature-table.qza)*<br>
Convert the Qiime2 artifact into a .biom file
```
qiime tools export \
  --input-path path/to/results/directory/feature-table.qza \
  --output-path path/to/results/directory/feature-table
```
This creates a directory named "feature-table" that contains a feature-table.biom file. We will now convert this into a .tsv file
```
biom convert \
  --input-fp path/to/results/directory/feature-table/feature-table.biom \
  --output-fp path/to/results/directory/feature-table/final-feature-table.tsv \
  --to-tsv
```
If you prefer to use a .csv file (which programs such as R Studio can play nicer with), convert the .tsv file into a .csv file
```
sed 's/\t/,/g' path/to/results/directory/feature-table/final-feature-table.tsv > path/to/results/directory/feature-table/final-feature-table.csv
```

*Export Representative Sequences (feature-rep-seqs.qza)*
<br>The code below converts the Qiime2 artifact containing the representative sequnces into a fasta file automatically names dna-sequences.fasta file inside of the directory named "rep-seqs"
```
qiime tools export \
  --input-path path/to/results/directory/feature-rep-seqs.qza \
  --output-path path/to/results/directory/rep-seqs
```
*Export Taxonomy Table (feature-rep-seqs.qza)*
<br>The code below converts the Qiime2 artifact containing the final taxa table (made in step 10: classification_merged_final.qza) into a .tsv file automatically named taxonomy.tsv inside of the directory named "taxonomy". Similarly as was seen for the feature table, if you prefer a .csv file you can convert the .tsv into a .csv file.
```
qiime tools export \
  --input-path path/to/final/results/directory/classification_merged_final.qza \
  --output-path path/to/results/directory/taxonomy

sed 's/\t/,/g' path/to/results/directory/taxonomy/taxonomy.tsv > path/to/results/directory/taxonomy/taxonomy.csv
```
<br><ins>Accessing final files</ins><br>
If you ran the pipeline on the HPC, you can log into your HPC accound and download the files. Alternatively, you can download them using the code belwo on the command line.
```
# Download an entire directory
scp -r user@koa.its.hawaii.edu:/home/user/path/to/results/directory/ \ ~/path/to/directory/on/local/drive/
```
```
# Download a single file
scp user@koa.its.hawaii.edu:/home/user/path/to/results/directory/file.ext \ ~/path/to/directory/on/local/drive/
```


