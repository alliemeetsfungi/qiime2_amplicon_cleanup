# Qiime2 Amplicon Sequencing Cleanup
All of the code used here is compatible with the package <ins>Qiime2 amplicon version 2024.10.</ins>, documents and other tutorials for this version can be found [HERE](https://docs.qiime2.org/2024.10/).<br><br>
The instructions and code for installing on a local drive are for macOS (Apple Silicon), and linux for installing on an HPC, but can be applied to other machines with modification.

## Purpose:
This SOP is designed to assist those with using the Qiime2 package for (i) performing an initial quality control (cleaning) using DADA2 of all sequences by removing low quality sequences (quality filtering) and chimeric sequences, correcting sequencing errors that are present (denoising), grouping duplicate sequences, and merging paired end sequences (if maintaining both the forward and reverse sequences for all samples), and (ii) assigning taxonomy to reamining sequences using various databases. This procedure focuses specifically on fungal 18S and bacterial 16S bacterial sequences, but can be modified for other amplicons as needed.

### Inisghts Into Bioinformatics Terminology:
For the purposes of this SOP, a local drive referes to your personal computer and the High Perfomance Computer (HPC) that is used is the University of Hawaiʻi at Mānoa's cluster named KOA which can be accessed through the online interface [HERE](https://koa.its.hawaii.edu/) or through the command line (described below). In order to get access to the cluster (KOA) you need to be associated with the University of Hawaiʻi at Mānoa, register for an account [HERE](https://datascience.hawaii.edu/eligibility-sign-up/), and take the required onboarding. Once those steps are completed you can access the cluser with your UH username (described below).
<br><br>
You will often see the word "path" which tells you the location of a folder (called a directory) or a file. For example, if you see something like this "path/to/folder/file.ext" this would indicate that the file "file.ext" is inside of the folder "folder" which is inisde of the folder "to" which is inside of the folder "path". The extension of the file (.ext) indicates what kind of file it is, you've seen this before for common files such as PDF (.pdf) and Microsoft Office (.docx) files that contain different extensions.<br>
## STEP 1: Prepare Files & Environment
<ins>Preparing Files For Qiime2</ins><br>
To begin, make sure sequences are downloaded in an accessible location as fastq.gz files on the local drive.<br><br>
If you're sequences are in the .fastq format, you can compress each .fastq file in a directory into .fastq.gz files:
```
gzip path/to/fastq/sequence/files/*.fastq
```
If you plan on using the HPC, theses .fast.qz files will need to be uploaded onto the cluster from your local terminal:
```
scp -r "path/to/folder" \ 
user@koa.its.hawaii.edu:/home/user/path/to/directory/for/files
```
NOTE: This will only work if you alrady have a KOA (or other HPC interface) account, see below for further details if you do not.<br>
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
<br> *- `https://packages.qiime2.org/qiime2/2024.10/amplicon/released`*
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
NOTE: KOA has a much older version of Qiime2 already installed, however the code written in this SOP may not be compatible since it is written for a newer version of Qiime2. Installing the version associated with this SOP gaurantees code compatability.<br><br>
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
Check the conda environment to make sure it installed correctly by looking for what you named your Qiime2 package during installation.
```
conda info - e
```
Here is an example of what is printed on my command line when I run the code above:<br>
*conda environments:*<br>
*qiime2    /home/alliej/.conda/envs/qiime2*<br>
*base      /opt/apps/software/lang/Anaconda3/2024.02-1*<br><br>
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
Now all of the code you run will be set to start at this directory and all paths can be defined based on the directories found within this location.<bR>
## STEP 2: Importing Sequences Into Qiime2
Qiime2 uses "Qiime Artifact" files, with a .qza extension, to store large amounts of information in a compressed format. In order to use Qiime2, the information recorded in all of your sequence files will need to be imported into a Qiime artifact. Additionally, all output files made within Qiime2 will be initially made into Qiime artifacts and will be converted into other formats for visualization and export (explained in detail later on).<br>
**WARNING:** Importing sequencing files into a Qiime artifact can take anywhere from **2-10 hours** depending on how much processing power your computer has (if running Qiime2 on your local drive) and how large your data set is.<br><br>
The two approaches outlined below are for importing either (i) paired-end sequences (where you have a foward and reverse read for each sample) or (ii) single-end sequenes (generally only the foward read sequences for each sample) both of which use the Casava 1.8 method. This method requires sequences to be demultiplexed and in a specific format that is standard for sequences recieved from the Advanced Studies in Genomics, Proteomics, and Bioinformatics (ASGPB) center at University of Hawaiʻi at Mānoa. If your libraries were run outside of the University of Hawaiʻi at Mānoa, you may have to use an alternative importing approach. Further details on the formatting specifics required for importing sequences into a Qiime artifact, as well as other methods available can be found [HERE](https://docs.qiime2.org/2024.10/tutorials/importing/).<br><br>
Regardless of the approach you choose to take to import all of your .fastq.gz sequencing files into a single Qiime artifact there are <ins>two lines of code that should be modified</ins>. The first is the <ins>"--input-path"</ins> which should show the path to the folder where your sequencing files are stored. The second is the <ins>"--output-path"</ins> line which should contain the path where your want your Qiime2 artifcat to be stored, and what you want it to be named.
<br>
### Import paired-end sequences using Casava 1.8 paired-end demultiplexed fastq method
Modify the --input-path and --output-path lines to indicate where your sequences are stored, and where you want your Qiime artifact to be stored and named (described above) respectively.
```
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path path/to/directory/containing/gzipped/fastq/files \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path path/to/where/qiime2/artifact/will/be/saved/file-name.qza
```
Convert the Qiime2 artifcat (.qza) into a Qiime2 visualization file (.qzv):
```
qiime demux summarize \
  --i-data path/to/where/qiime2/artifact/was/saved/file-name.qza \
  --o-visualization path/to/where/qiime2/visual/file/will/be/saved/file-name.qzv
```
Now that the Qiime artifact (.qza) has been converted into a Qiime visualization file (.qzv) we can actually visualize what the Qiime artifact contained. To do this, open your browser and navigate to the Qiime2 Viewer found [HERE](https://view.qiime2.org/?src=e96f979f-4cc6-46fc-800f-abe58740e4ea). You can now upload the .qzv file and observe if the Forward and Reverse read counts are the same (this is our first gut check)! [THIS FORUM](https://forum.qiime2.org/t/demultiplexed-sequence-length-summary-identical-forward-and-reverse-for-emp-paired-end-reads/20692) has more information on why this should be the outcome for paired-end sequences.<br>
<ins>Using the Qiime2 Viewer, perform the following</ins><br>
1. Record the total reads present for all your sequences.<br>
2. Observe the quality of forward and reverse reads.<br>
*To Observe the read quality, go to the "Interactive Quality Plot" tab*<br>
Here, you can observe the quality of reads (y-axis) at a specific sequence length (x-axis) and assess (i) wether sequences should be trimmed, and (ii) how much they should be trimmed. The quality score tells us how confidently the nucleotide was called at that area such that higher quality scores indicate higher confidence that the detected nucleotide is correct and not an error. Generally, quality scores below 20 are considered poor and unreliable reads which can lead to inacurate taxonomic identification later on. You can zoom in and out of the quality plots by highlighting regions of interest. To zoom back out to see the entire plot, just double click the plot.
<br><ins>Approach to deciding how much to trim:</ins><br>
NOTE: Reverse reads are generally lower in quality and may require more trimming than Forward reads. However, if you want to be able to merge your paired end sequencese they must be long enough to have a minimum of 20 bp overlap (although I have found that it takes closer to 50 bp overlap for sequences to merge succesfully). Make sure you know how long you amplicon is to assess what the minimum length required for your forward and reverse reads needs sto be in order for them to be merged before trimming.
1. Observe how consistent the quality is by assessing the sequence length at which the quality begins to drop abrubptly, as well as how steep the drop in quality is.
2. Determine at what sequence length the quality score begins to drop under 20.
3. Take a screen shot of the quality plots for your records.
4. If you want to record or look over reads per sample, scroll to the very bottom of the *"Overview"* tab and click "Download as TSV" to download per-sample-fastq-counts.tsv.
<br><ins>If total read counts are different for Forward and Reverse sequences, check the following:</ins>
<br>1. All files were uploaded correctly (should not be 0 bytes).
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
<ins>Copy all foward sequences into new directory before importing sequences!</ins>
<br>All forward sequence files share a common suffix in their file names such as READ1.fastq.gz or R1_001.fastq.gz. Use this shared suffix, unique from the reverse reads (READ2.fastq.gz or R2_001.fastq.gz), to differentiate forward from reverse reads within the code.
```
cp /path/to/directory/*R1_001.fastq.gz \
/path/to/new/directory
```
Import forward sequences into a Qiime artifact (.qza)
```
qiime tools import \
  --type 'SampleData[SequencesWithQuality]' \
  --input-path path/to/directory/containing/gzipped/forward/fastq/files \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path path/to/where/qiime2/artifact/will/be/saved/file-name.qza
```
Convert Qiime2 artifcat into visualization file.
```
qiime demux summarize \
  --i-data path/to/where/qiime2/artifact/was/saved/file-name.qza \
  --o-visualization path/to/where/qiime2/artifact/will/be/saved/file-name.qzv
```
See paired-end section for next steps using the Qiime2 visualization file (.qzv)
<br><br>If running multiple tests on the same set of sequences, single end and paired-end outputs should have the same output parameters for the forward sequences.
<br>
## STEP 3: Trim Primers From Sequences
To trim primers from all sequences, Qiime2 uses uses cutadapt (handbook found [HERE](https://docs.qiime2.org/2024.10/plugins/available/cutadapt/index.html)).
<br>For background on trimming Golay barcodes see [THIS](https://forum.qiime2.org/t/cutadapt-adapter-vs-front/15450) forum page.

**Primers commonly used in our studies:**
<br><ins>Bacterial small ribosomal subunit (16S) V4 amplicon using 515F/806R primers</ins>. 
  <br>Amplicon ~ 250-300 bp
  <br>515F (Forward primer): 5′-GTGYCAGCMGCCGCGGTAA-3′
  <br>806R (Reverse primer): 5′-GGACTACNVGGGTWTCTAAT-3′
<br><br><ins>Arbuscular Mycorrhizal Fungi (AMF) small ribosomal subunit (18s) V4 amplicon using WANDA/AML2 primers</ins>. See Kacie Kajihara's paper [HERE](https://nph.onlinelibrary.wiley.com/doi/10.1111/nph.18058).
  <br>Amplicon ~ 500 bp
  <br>WANDA (Forward Primer): 5′-GAAACTGCGAATGGCTC-3′
  <br>AML2 (Reverse Primer): 5′-GAACCCAAACACTTTGGTTTCC-3′
<br><br><ins>Fungi small ribosomal subunit (18s) V3-V4 amplicon using 18S-82F/Euk-516r primers</ins>. See Jason Baer's paper [HERE](https://academic.oup.com/ismej/article/19/1/wraf228/8284954#supplementary-data).
  <br>Amplicon ~
  <br>18S-82F (Forward Primer): 5′-GAAACTGCGAATGGCTC-3′
  <br>Euk-516R (Reverse Primer): 5′-ACCAGACTTGCCCTCC-3′
<br><br><ins>Orchid Mycorrhizal Fungi (OMF) Internal Transcribed Spacer 2 (ITS2) amplicon using fITS7 paired with either Tul1F or Tul2F</ins>
  <br>Amplicon ~
  <br>fITS7 (Forward Primer): 5′-GTGARTCATCGAATCTTTG-3′
  <br>ITS4 (Reverse Primer): 5′-TCCTCCGCTTATTGATATGC-3′
  <br>Amplicon ~
  <br>Tul1F (Forward Primer): 5′-CGTYGGATCCCTYGGC-3′
  <br>ITS4-Tul2 (Reverse Primer): 5′-TTCTTTTCCTCCGCTGAWTA-3′
  <br>Amplicon ~
  <br>Tul2F (Forward Primer): 5′-TGGATCCCTTGGCACGTC-3′
  <br>ITS4-Tul2 (Reverse Primer): 5′-TTCTTTTCCTCCGCTGAWTA-3′
<br>
### Trim Primers From Paired-end Sequences
This method trims primers based on primer sequence rather than length, assuring that the correct region is trimmed off rather than only trimming off the initial ambiguous regions commonly seen in sequencing results (i.e., a region with NNNNNNN).
```
qiime cutadapt trim-paired \
   --i-demultiplexed-sequences path/to/where/qiime2/import/artifact/was/saved/file-name.qza \
   --p-front-f FWDPRIMERSEQUENCE \
   --p-front-r REVPRIMERSEQUENCE \
   --o-trimmed-sequences path/to/where/file/will/be/saved/file-name.qza \
   --verbose

# Convert .qza into .qzv
qiime demux summarize \
  --i-data path/to/where/file/was/saved/file-name.qza \
  --o-visualization path/to/where/file/will/be/saved/file-name.qzv
```
--verbose tells Qiime2 to print out the logging output to the terminal while it runs the command. This can help trouble shoot any errors that may occur during the run, but is not necessary for trimming process. When running codes with a bash scripts on the HPC, these will be printed in the .out files produced from that job.<br>
<br>
Go to the Qiime2 Viewer on the browser and upload the .qzv file, <ins>record total reads and any other pertinent information</ins>. Compare these outputs to your imported outputs from STEP 2. There shouldn't be any differences if everything worked correctly!
<br><br>Scroll to the very bottom of the "Overview" page and click "Download as TSV" to download per-sample-fastq-counts.tsv post primer trimming if desired.
<br>
### Trim Primers From Forward Sequences
This method is similar to trimming primers off of paired end sequences, with minor differences.
```
qiime cutadapt trim-single \
   --i-demultiplexed-sequences path/to/where/qiime2/import/artifact/was/saved/file-name.qza \
   --p-front FWDPRIMERSEQUENCE \
   --o-trimmed-sequences path/to/where/file/will/be/saved/file-name.qza \
   --verbose

qiime demux summarize \
  --i-data path/to/where/file/was/saved/file-name.qza \
  --o-visualization path/to/where/file/will/be/saved/file-name.qzv
```
See paired-end section for next steps using the Qiime2 visualization file (.qzv)
<br><br>If running multiple tests on the same set of sequences, single end and paired-end outputs should have identical parameters.
<br>
## STEP 4: DADA2 - Trimming, Merging, Denoising, and Feature Calling of Sequences
To assure that the most features will be detected, multiple tests will be run at this step based on post primer trimming quality plots.
<br><br>This process will produce three Qiime artifacts:
1. <ins>Feature Table</ins> (feature-table.qza)
2. <ins>Representative Sequences</ins> of detected features (feature-rep-seqs.qza)
3. <ins>Denoising Statistics</ins> that shows the breakdown of the reads which passed each cleaning step (filtering, denoising, merging, and chimeras) for every sample (denoising-stats.qza)

### Cleaning and Merging Paired-end Sequences<br>
<ins>Test 1: Maintain Entire Sequence length</ins><br>
Setting the truncation length (--p-trunc-len-f & --p-trunc-len-r) to 300 should maintain the entire sequence for both foward and reverse reads.
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
Convert feature table Qiime artifact (feature-table.qza) into visualization file (feature-table.qzv).
```
qiime feature-table summarize \
  --i-table path/to/results/directory/feature-table.qza \
  --o-visualization path/to/results/directory/feature-table.qzv
```
Upload to the Qiime2 Viewer and record the number of samples, features, and total frequency (reads) present for this test.
<br><br>Convert representative sequences Qiime2 artifcat (feature-rep-seqs.qza) into a visualization file (feature-rep-seqs.qzv).
```
qiime feature-table tabulate-seqs \
  --i-data path/to/results/directory/feature-rep-seqs.qza \
  --o-visualization path/to/results/directory/feature-rep-seqs.qzv
```
Now upload this .qzv file onto the Qiime2 Viewer and record sequence count (features), and sequence length statistics. Here, you can also download a .fasta file containing the sequence associated with each feature detected if desired, but this is also exported later on.
<br><br>Convert denoising statistics Qiime artifact (denoising-stats.qza) into visualization file (denoising-stats.qzv)
```
qiime metadata tabulate \
  --m-input-file path/to/results/directory/denoising-stats.qza \
  --o-visualization path/to/results/directory/denoising-stats.qzv
```
This table summarizes the sequencing reads for each sample that were able to pass the cleaning steps performed in DADA2. Record how many samples were able to retain an acceptable amount (atleast 25%) of their reads. Additionally, you can download the entire table for referencing later.<br>
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
Convert the feature table, representative sequences, and denoising statistics Qiime artifact files into visualization files as described above.
```
# Feature Table
qiime feature-table summarize \
  --i-table path/to/results/directory/feature-table.qza \
  --o-visualization path/to/results/directory/feature-table.qzv

# Representative Sequences
qiime feature-table tabulate-seqs \
  --i-data path/to/results/directory/feature-rep-seqs.qza \
  --o-visualization path/to/results/directory/feature-rep-seqs.qzv

# Denoising Statistics
qiime metadata tabulate \
  --m-input-file path/to/results/directory/denoising-stats.qza \
  --o-visualization path/to/results/directory/denoising-stats.qzv
```
As described above, make sure that the number of samples, features, reads, sequence length statistics, and denoising statistics are recorded when provided, and download any file of interest.<br>
### Cleaning Forward Sequences
<ins>Test 3: Trim forward sequences conservatively</ins>
<br>Choose to trim sequences shorter in order to avoid poor quality reads 
```
qiime dada2 denoise-single \
  --i-demultiplexed-seqs path/to/primer/trimmed/file-name.qza \
  --p-trunc-len 270 \
  --o-table path/to/results/directory/feature-table.qza \
  --o-representative-sequences path/to/results/directory/feature-rep-seqs.qza \
  --o-denoising-stats path/to/results/directory/denoising-stats.qza \
  --verbose

# Visualize Feature Table
qiime feature-table summarize \
  --i-table path/to/results/directory/feature-table.qza \
  --o-visualization path/to/results/directory/feature-table.qzv

# Visualize Representative Sequences
qiime feature-table tabulate-seqs \
  --i-data path/to/results/directory/feature-rep-seqs.qza \
  --o-visualization path/to/results/directory/feature-rep-seqs.qzv

# Visualize Denoising Statistics
qiime metadata tabulate \
  --m-input-file path/to/results/directory/denoising-stats.qza \
  --o-visualization path/to/results/directory/denoising-stats.qzv
```
As described above, make sure that the number of samples, features, reads, sequence length statistics, and denoising statistics are recorded when provided, and download any file of interest.
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

qiime feature-table summarize \
  --i-table path/to/results/directory/feature-table.qza \
  --o-visualization path/to/results/directory/feature-table.qzv

qiime feature-table tabulate-seqs \
  --i-data path/to/results/directory/feature-rep-seqs.qza \
  --o-visualization path/to/results/directory/feature-rep-seqs.qzv

qiime metadata tabulate \
  --m-input-file path/to/results/directory/denoising-stats.qza \
  --o-visualization path/to/results/directory/denoising-stats.qzv
```
As described above, make sure that the number of samples, features, reads, sequence length statistics, and denoising statistics are recorded when provided, and download any file of interest (described above).
<br><br><ins>Once all tests have been run, compare the number of features and number of samples with sufficient read retention among all tests.</ins> The test with the highest amount of features and samples with sufficient read retention should be used from this point forward.
<br>
## STEP 5 (OPTIONAL!): Cluster ASVs into OTUs
Using the feature table and representative sequences from the DADA2 cleanup with the best feature count and sample read retention outcome, ASVs can be clustered into OTUs if desired with the code below. Use these outputs for the subsequent steps in this SOP if you plan on assessing OTUs instead of ASVs for downstream analyses.
<br><br>Cluster sequences into 97% identity, and convert all output .qza artifact files into .qzv visualization files for assessment.
```
qiime vsearch cluster-features-de-novo \
  --i-table path/to/results/directory/feature-table.qza \
  --i-sequences path/to/results/directory/feature-rep-seqs.qza \
  --p-perc-identity 0.97 \
  --o-clustered-table path/to/results/directory/otu-feature-table.qza \
  --o-clustered-sequences path/to/results/directory/otu-feature-rep-seqs.qza \
  --verbose

# Feature Table
qiime feature-table summarize \
  --i-table path/to/results/directory/otu-feature-table.qza \
  --o-visualization path/to/results/directory/otu-feature-table.qzv

# Representative Sequences
qiime feature-table tabulate-seqs \
  --i-data path/to/results/directory/otu-feature-rep-seqs.qza \
  --o-visualization path/to/results/directory/otu-feature-rep-seqs.qzv
```
Download .fasta file containing feature representative sequences if desired.<br>
NOTE: Denoising statistics from the original output in STEP 4 will be the same and does not have an OTU specific output.
<br>
## STEP 6: Export Feature Table For Culling (OPTIONAL)
The resulting files can be imported into R for initial culling assesment, or you can skip this for now and export at the end after taxonomic assignment. Do so now will allow you to set your culling thresholds for sample and features  while your taxonomic assignments run. However, if you are using Phyloseq for your culling steps (as is standard for our lab), it is easier to wait until the taxonomic table can be included in the phyloseq object when removing samples and features. 
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
If you've been performing all of these steps on the HPC, you can download the final table in new command line window connected to your local drive (not signed into KOA).
```
scp user@koa.its.hawaii.edu:/home/user/path/to/results/directory/feature-table-directory/feature-table.csv \
path/to/local/drive/directory
```
The files should now be in your chosen directory and ready for import into R for culling.
<br>
## STEP 7: Import Databases For Taxonomic Identification
The following databases are the most commonly used for our studies. Some require files to be downloaded from the database webpage, while other databases can be pulled directly through Qiime2. This step only needs to be performed once even if the taxonomic identification has to be re-done or if you're wanting to perform taxonomic assignment on multiple tests (for example on both your ASV and OTU results).
<br><br>**SILVA**
<br>This database is used for both 16S (prokaryotes) & 18S (eukaryotes) small ribosomal subunits. You can pull the database files needed for Qiime2 straight from the SILVA database using the code below. See [THIS](https://forum.qiime2.org/t/sequence-and-taxonomy-files-for-silva-v138-2/33475) forum for more details. The code provided in this SOP is based on SILVA version 138.2. <ins>NOTE: Reference sequences extracted from SILVA are in rRNA format and need to be "transcribed" before using for taxonomic identification.</ins>
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

### Fungi (and other micro-eukaryotes) Databases
**Eukaryome**
<br>Files can be downloaded [HERE](https://eukaryome.org/qiime2/) for Qiime2 compatability. This code provided in this SOP is based on QIIME2_EUK_SSU Version 2.0, however databases for LSU, ITS, and other microbial amplicons are available.
<br><br>Once database files are downloaded transfer them into your working directory, then import the reference sequences and taxonomy files into Qiime2 using the following code:
```
# Import Reference Sequences
qiime tools import --type 'FeatureData[Sequence]' \
  --input-path path/to/file/QIIME2_EUK_SSU_v2.0.fasta \
  --output-path desired/path/to/file/eukaryome-ref-seqs.qza

# Import Taxonomy
qiime tools import --type 'FeatureData[Taxonomy]' \
  --input-format HeaderlessTSVTaxonomyFormat \
  --input-path path/to/file/QIIME2_EUK_SSU_v2.0.tsv \
  --output-path desired/path/to/file/eukaryome-ref-tax.qza
```
<br>

**National Center for Biotechnology Information (NCBI)**
<br>The reference sequences and taxonomic files from the most up to date version of the NCBI database can be pulled straight from the NCBI server and imported into Qiime2:
```
qiime rescript get-ncbi-data \
  --p-query '18S[ALL] AND fungi[ORGN]' \
  --o-sequences desired/path/to/file/ncbi-fungi-ref-seqs.qza \
  --o-taxonomy desired/path/to/file/ncbi-fungi-ref-tax.qza \
  --p-n-jobs 5
```
You can alter the query to filter for specific taxa, such as Glomeromycotina, if desired:
```
qiime rescript get-ncbi-data \
--p-query '18S[ALL] AND glomeromycotina [ORGN]' \
--o-sequences desired/path/to/file/ncbi-AMF-ref-seqs.qza \
--o-taxonomy desired/path/to/file/ncbi-AMF-ref-tax.qza \
--p-n-jobs 5

```
<br>

**MaarjAM**
<br>Qiime2 compatable files can be downloaded [HERE](https://maarjam.ut.ee/?action=bDownload). This code is based on MaarjAM VT sequences of 18S rDNA gene region QIIME release (2021) for identifying Arbuscular Mycorrhizal Fungi (AMF).
<br><br>Once database files are downloaded transfer them into your working directory, then import the reference sequences and taxonomy files into Qiime2 using the following code:
```
# Import Reference Sequences
qiime tools import --type 'FeatureData[Sequence]' \
--input-path path/to/file/maarjam_database_SSU.qiime.fasta \
--output-path desired/path/to/file/maarjam-ref-seqs.qza

# Import Taxonomy
qiime tools import --type 'FeatureData[Taxonomy]' \
--input-format HeaderlessTSVTaxonomyFormat \
--input-path path/to/file/maarjam_database_SSU.qiime.txt \
--output-path desired/path/to/file/maarjam-ref-tax.qza
```
<br>

**UNITE**<br>
UNDER CONSTRUCTION
<br>
### Bacteria (and other prokaryotes) Databases
**Genome Taxonomy Database (GTDB)**
<brThe database website can be found [HERE](https://gtdb.ecogenomic.org/). Sequences can be pulled using Qiime2 with the code below based on GTDP Version 220.0.
```
qiime rescript get-gtdb-data \
  --p-version 220.0 \
  --p-domain Bacteria \
  --output-dir desired/directory/path/for/files
```
<br>

**GREENGENES2**<br>
UNDER CONSTRUCTION
<br>

## STEP 8: Taxonomic Assignment To Features
Using the Qiime artifact created by DADA2 (feature-rep-seqs.qza) containing the sequences associated with each identified feature (representative sequences) as the input file, these sequences can be used for BLAST searching each features taxonomic association.
<br><br>For each database, run unnassigned sequences with 95%, 90%, and 80% identities. This allows for higher rates of taxonomic assignment by loosening the sequence consensus calling requirements for each database. If you want to use multiple databases to assign as many taxa as possible, start with BLAST searching the sequences through the database believed to provide the most assignments at all three query coverage targets first, then run the remainder of the unassigned sequences through the next best database. The code below shows an example of how to do this with two databases, however this can performed with additional databases (i.e., 3 or more) if needed!
<br><br>**Database ONE**
<br><ins>95% Identity</ins>
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
The final output made here (path/to/search/results/database/directory-95/unassigned-rep-seqs.qza) will be used as the input file for the next search query!<br><br>
<ins>90% Identity</ins><br>
The same code will be used for this search query, with the exception of the input file (which should be changed to your final filtered output file), the percent identity (change from 0.95 to 0.90), and the output directory.
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
Filter out remaining unassigned sequences from this query search into their own file in the new directory.
```
qiime taxa filter-seqs \
  --i-sequences path/to/search/results/database/directory-95/unassigned-rep-seqs.qza \        #Use the input file for the BLAST search (unassigned-rep-seqs.qza)
  --i-taxonomy path/to/search/results/database/directory-90/classification.qza \              #Input the resulting classification.qza file
  --p-include unassigned \
  --o-filtered-sequences path/to/search/results/database/directory-90/unassigned-rep-seqs.qza #Remaining sequences that did not have taxa assigned under these search parameteres
```
Using this final output, repeat the steps performed for 95% and 90% identities at 80% identity.
<br><br>
<ins>80% Identity</ins>
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
Now the remaining unassignd sequences can be run through the next database!
<br><br>**Database TWO**<br>
<ins>95% Identity</ins><br>
At this step, you need to change the path for the reference sequences (--i-reference-reads) and taxa (--i-reference-taxonomy) to the files associated with the new databases reference sequences and taxa. The remaining unassigned representative sequences (unassigned-rep-seqs.qza) from the previous database should be used for --i-query.
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
Continue on to run the remaining unassigned representative sequences at 90% and 80% identities for this database as you did for the first database.
```
# 90% Identity BLAST Search
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

# Filter out remaining unassigned sequences
qiime taxa filter-seqs \
  --i-sequences path/to/search/results/NEW-database/directory-95/unassigned-rep-seqs.qza \
  --i-taxonomy path/to/search/results/NEW-database/directory-90/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences path/to/search/results/NEW-database/directory-90/unassigned-rep-seqs.qza


# 80% Identity BLAST Search
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

# Filter out remaining unassigned sequences
qiime taxa filter-seqs \
  --i-sequences path/to/search/results/NEW-database/directory-90/unassigned-rep-seqs.qza \
  --i-taxonomy path/to/search/results/NEW-database/directory-80/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences path/to/database/search/results/NEW-database/directory-80/unassigned-rep-seqs.qza
```
If you have more databases you are interested in running your representative sequences through for furhter taxonomic assignment, repeat these steps for each additional database. See (INSERT MY OWN PIPELINES HERE) for reference on using multiple databases for taxonomic assignment with a real dataset.
<br>
## STEP 9: Filtering Feature Tables (OPTIONAL)
Once you are satisfied with your taxonomic assignments using your representative sequences, you can filter your actual feature table to contain only taxonomically assigned features, or conversely only the remaining unassigned features.<br><br>
**Database ONE**<br>
Begin with your original DADA2 feature table as your input table (feature-table.qza) and your classification file made from your first database at 95% identity.<br><br>
Filtering for <ins>assigned</ins> features:
```
qiime taxa filter-table \
  --i-table path/to/results/directory/feature-table.qza \                            #Your feature table from DADA2
  --i-taxonomy path/to/search/results/database/directory-95/classification.qza \     #Classification file made from the first taxonomic query run
  --p-exclude unassigned \                                                           #This will keep all features EXCEPT those that are annotated as unassigned
  --o-filtered-table path/to/search/results/database/directory-95/assigned-table.qza #Desired output path of table
```
Filtering for <ins>UNassigned</ins> features:
```
qiime taxa filter-table \
  --i-table path/to/results/directory/feature-table.qza \                            #Your feature table from DADA2
  --i-taxonomy path/to/search/results/database/directory-95/classification.qza \     #Classification file made from the first taxonomic query you ran 
  --p-include unassigned \                                                           #This will keep only features annotated as unassigned
  --o-filtered-table path/to/search/results/database/directory-95/unassigned-table.qza #Output path of table
```
Using the resulting unassigned table as your input table, repeat these steps for all additional taxonomic query searches you performed in the same order in which they were run.
<br><br>
<ins>90% Identity</ins>
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
<ins>80% Identity</ins>
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
Repeat these steps for the second database query results in sequence.<br><br>
**Database TWO**
```
# 95% identity
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


# 90% identity
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


# 80% identity
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
Since the feature tables containing assigned taxa are made by subsetting the original table, they are saved as multiple different tables. In order to retrieve a single feature table containing all of the assigned taxonomic features, all of the individual feature tables containing the assigned taxa will need to be merged. This does not need to be performed for the unassigned tables since you filter OUT the assigned taxa as you go rather than subset for them, making the final unassigned table your complete feature table containing all of the unassigned features. I like to make a copy of the final unassigned feature table, rename it along the lines of "final-unassigned-feature-table.qza", and move this copy to my final results folder for easier access, but this is strictly preference.<br><br>
Merge all assigned tables:
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
Convert both the final assigned and unassigned feature table Qiime artifacts (.qza) into Visualization files (.qzv).
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
Upload these onto the Qiime2 viewer and record the number of samples, features, and reads accounted for in both feature tables.
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
Upload the visualization file and confirm that the sample, feature, and read counts match that of the feature-table.qza produced from DADA2.<br>

## STEP 10: Merging Classification Files
To retrieve your final taxa table, you will need to merge all of your classification files which can be a bit tricky. Prior to merging, I find it helpful to determine how many features were annotated within each classfication file in order to make sure that merging is occurring correctly. To do this, all of the clalssification.qza Qiime artifacts will need to be converted into a visualization file.
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
Upload each visualization file and record the number of features present.
<br><br>
**Merging Classification Tables**
<br>When mergering classification tables, it is essential that the classification table with the least amount of classified taxa is listed as the first input table. Otherwise, the output table will come out only containing the taxa from the classification file with the most taxa present.
```
qiime feature-table merge-taxa \
  --i-data path/to/search/results/database/directory-90/classification.qza \
  --i-data path/to/search/results/database/directory-95/classification.qza \
  --o-merged-data path/to/search/results/database/classification_merged1.qza
```
Now check to make sure they merged by converting the output Qiime artifact into a visualization file, uploading it onto the Qiime2 viewer, and observing if the number of features that are present is equal to the sum of both of the classification files you merged.
```
qiime metadata tabulate \
  --m-input-file path/to/search/results/database/classification_merged1.qza \
  --o-visualization path/to/search/results/database/classification_merged1.qzv
```
Repeat these steps until all of the classification artifacts are merged. There are multiple combinations that can be made to do this, however I prefer to make multiple merge files that I then merge together at the end. The code below represents this approach but can be modified for preference.
```
# Merge next two classification artifacts
qiime feature-table merge-taxa \
  --i-data path/to/search/results/NEW-database/directory-95/classification.qza \
  --i-data path/to/search/results/database/directory-80/classification.qza \
  --o-merged-data path/to/search/results/database/classification_merged2.qza

# Convert and check feature counts make sense
qiime metadata tabulate \
  --m-input-file path/to/search/results/database/classification_merged2.qza \
  --o-visualization path/to/search/results/database/classification_merged2.qzv


# Merge next two classification artifacts, & check feature counts
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
NOTE: The directory for the final merged table is set to be where you want all of your final results to be for easy organization.<br>

## STEP 11: Export Final Tables And Representative Sequences
Now we have all of our final tables ready to export for normalization via asv & sample culling and decontam runs.<br><br>
**Export Feature (ASV) Table (feature-table.qza)**
<br>Convert the Qiime artifact containing the file feature table (from STEP 4 or 5) into a .biom file:
```
qiime tools export \
  --input-path path/to/results/directory/feature-table.qza \
  --output-path path/to/results/directory/feature-table
```
This creates a directory named "feature-table" that containing the feature table file (feature-table.biom).<br><br>
Convert the .biom file into a .tsv file for compatability with the downstream software you plan to use:
```
biom convert \
  --input-fp path/to/results/directory/feature-table/feature-table.biom \
  --output-fp path/to/results/directory/feature-table/final-feature-table.tsv \
  --to-tsv
```
If you prefer a .csv file (which programs such as R Studio can play nicer with), convert the .tsv file into a .csv file:
```
sed 's/\t/,/g' path/to/results/directory/feature-table/final-feature-table.tsv > path/to/results/directory/feature-table/final-feature-table.csv
```
<br>**Export Representative Sequences (feature-rep-seqs.qza)**
<br>The code below converts the Qiime artifact containing the representative sequnces (from STEP 4 or 5) into a .fasta file automatically named "dna-sequences.fasta" inside of a directory named "rep-seqs":
```
qiime tools export \
  --input-path path/to/results/directory/feature-rep-seqs.qza \
  --output-path path/to/results/directory/rep-seqs
```
<br>**Export Taxonomy Table (classification_merged_final.qza)**
<br>The code below converts the Qiime artifact containing the final taxa table (from Step 10) into a .tsv file automatically named "taxonomy.tsv" inside of the directory "taxonomy". Similarly as was seen for the feature table, if you prefer a .csv file you can convert the .tsv into a .csv file.
```
qiime tools export \
  --input-path path/to/final/results/directory/classification_merged_final.qza \
  --output-path path/to/results/directory/taxonomy

sed 's/\t/,/g' path/to/results/directory/taxonomy/taxonomy.tsv > path/to/results/directory/taxonomy/taxonomy.csv
```
<br>**Accessing final files**<br>
If you ran the pipeline on the HPC, you can log into your HPC account and download the files from within the directories created. Alternatively, you can download them using the code below on the command line.
```
# Download an entire directory
scp -r user@koa.its.hawaii.edu:/home/user/path/to/results/directory/ \ ~/path/to/directory/on/local/drive/
```
```
# Download a single file
scp user@koa.its.hawaii.edu:/home/user/path/to/results/directory/file.ext \ ~/path/to/directory/on/local/drive/
```
<br><br><br>
### References
1. Qiime2
2. Eukaryome
3. Maarjam
4. Silva
5. greengenes2
6. ncbi
7. gtdb
8. unite
9. others?

