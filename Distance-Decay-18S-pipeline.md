**NOTE:** The steps for the code below are directly correlated to the steps outlined in the [QIIME 2_SOP.md](https://github.com/alliemeetsfungi/qiime2_amplicon_cleanup/blob/main/QIIME 2_SOP.md), thus some jumps will be observed between step numbers.<br><br>
## STEP 1: Prepare Files & Environment
<ins>Preparing Files For QIIME 2</ins><br>
On the local command line run the following code to upload all .fastq.gz files onto KOA:
```
scp -r "/Volumes/LaCie/Thesis/Chapter 1 - Bromeliads/Sequencing/sequences/ssu_gz" \
alliej@koa.its.hawaii.edu:/home/alliej/hynson_koastore/ajhall/bros/sequences/ssu_raw/ssu_gz
```

### Installing QIIME 2 on HPC
Access KOA on command line:
```
ssh alliej@koa.its.hawaii.edu
```
When prompted, input UH password & designate two factor authentication preference.<br><br>

Start interactive job (NOTE: the more memory (--mem) and cores (-c) you ask for, the longer it will take to be given resources):
```
srun -p shared --mem=100G -c 4 -t 06:00:00 --pty /bin/bash
```
Load anaconda module:
```
module load lang/Anaconda3/2024.02-1 
```
Install qiime2 (NOTE: line after -n is what the environment will be named! Here it is qiime2):
```
conda env create -n qiime2 --file https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2024.10-py310-linux-conda.yml
```
Check conda environments, what you named the environment should be present
```
conda info - e
	#conda environments:                                                                                                                                        
	#qiime2    /home/alliej/.conda/envs/qiime2                                                                                                     
	#base       /opt/apps/software/lang/Anaconda3/2024.02-1  
```
We can see that qiime2 exists within the conda environment, now it can be activated on the HPC at any time after starting an interactive job and loading the Anaconda module:
```
source activate qiime2
```
Go to working directory where raw sequences can be accessed from the folder they are stored in. In this case, they were uploaded into a directory that is within the directory "sequences", thus the working directory will be set to this location:
```
cd hynson_koastore/ajhall/bros/sequences
```
<br>

## STEP 2: Importing Sequences Into QIIME 2
Further details and instructions can be found [HERE](https://docs.qiime2.org/2024.10/tutorials/importing/).<br>
Visualization files can be viewed [HERE](https://view.qiime2.org/?src=e96f979f-4cc6-46fc-800f-abe58740e4ea).<br><br>
### Import paired-end sequences using Casava 1.8 paired-end demultiplexed fastq method
```
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path sequences/ssu_raw/ssu_gz \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path clean/ssu/import/ssu-paired-end-casava.qza

# Convert qiime artifcat into visualization file 
qiime demux summarize \
  --i-data clean/ssu/import/ssu-paired-end-casava.qza \
  --o-visualization clean/ssu/import/ssu-paired-end-casava.qzv
```
Scroll to the bottom of the "Overview" tab to download per sample count .tsv file.<br>
stored as "ssu-paired-end-casava-per-sample-fastq-counts.tsv" on local drive.<br><br>
SSU Sequence Counts (reads):<br>
Minimum: 101<br>
Median: 24842.0<br>
Mean: 27740.121212<br>
Maximum: 101084<br>
Total: 12815936<br>
Note: Forward and Reverse read counts should be the same! See [THIS](https://forum.qiime2.org/t/demultiplexed-sequence-length-summary-identical-forward-and-reverse-for-emp-paired-end-reads/20692) forum for more details.<br><br>
If total read counts are different, check to make sure files were uploaded correctly (should not be 0 bytes). For this project, the first time I imported my fungal sequences the forward and reverse reads showed different read counts. It turned out one of my reverse sequencing files did not upload correctly onto the HPC (file was present, but contained 0 bytes of information), thus holding no reads and making the read counts non-identical. After re-uploading this file the forward and reverse read counts matched after importing.<br><br>
<ins> Here are some trouble shooting steps if this occurs.</ins>
1. Open per-sample-fastq-counts.tsv and search for samples where forward reads and reverse reads aren't the same. In cell D, set the formula to "=B2=C2", drag down, and filter for FALSE values. The sample that doesn't have equal forward and reverese reads is more than likely causing the issue.
2. Go to the HPC, navigate to the directory containing all of the raw .fastq.gz files, and sort from largest to smallest (that way the problematic file will be on the bottom!) and see if any of the files contain 0 bytes:
```
# Go to raw sequencing directory
cd hynson_koastore/ajhall/bros/sequences/ssu_raw/ssu_gz

# Sort by file size
ls -lhS
```
3. Alternatively, you can check the specific file associated with the sample found in step 1:
```
zcat /home/alliej/hynson_koastore/ajhall/bros/sequences/16s_raw/16s_gz/DD-LY-P2-BR11-L_S87_L001_R1_001.fastq.gz | echo $((`wc -l`/4))
```
4. Check to see if your locally stored files have a different file size. If it does, then re-upload the file you found to be the issue:
```
scp "/Volumes/LaCie/Thesis/Chapter 1 - Bromeliads/Sequencing//sequences/16s_gz/DD-LY-P2-BR11-L_S87_L001_R1_001.fastq.gz" \
alliej@koa.its.hawaii.edu:/home/alliej/hynson_koastore/ajhall/bros/sequences/16s_raw/16s_gz
```
5. Check to make sure it was re-uploaded correctly:
```
zcat /home/alliej/hynson_koastore/ajhall/bros/sequences/16s_raw/16s_gz/DD-LY-P2-BR11-L_S87_L001_R1_001.fastq.gz | echo $((`wc -l`/4))
```
6. Re-run import code above to make your QIIME 2 artifact containing all of your raw sequences and check to make sure that the forward and reverse read counts are identical.<br><br>
### Import Forward sequences using Casava 1.8 single-end demultiplexed fastq method
Copy all foward sequences into new directory:
```
cp /mnt/lustre/koa/koastore/hynson_group/ajhall/bros/sequences/ssu_raw/ssu_gz/*R1_001.fastq.gz \
/mnt/lustre/koa/koastore/hynson_group/ajhall/bros/sequences/ssu_raw/ssu_fwd_gz/
```
Import all forward sequences from this new directory:
```
qiime tools import \
  --type 'SampleData[SequencesWithQuality]' \
  --input-path sequences/ssu_raw/ssu_fwd_gz \
  --input-format CasavaOneEightSingleLanePerSampleDirFmt \
  --output-path clean/ssu/import/ssu-fwd-casava.qza

# Convert qiime artifcat into visualization file 
qiime demux summarize \
  --i-data clean/ssu/import/ssu-fwd-casava.qza \
  --o-visualization clean/ssu/import/ssu-fwd-casava.qzv
```
SSU Forward only sequence count summaries should be the same as paired-end.<br>
Download per sample count .tsv file, saved as "ssu-fwd-casava-per-sample-fastq-counts.tsv".<br>
## STEP 3: Trim Primers From Sequences
18S-82F/Euk-516R primers with Nextera indices were used for amplification & sequencing of the V3-V4 region of the fungal small ribosomal subunit:<br>
18S-82F (Forward Primer): 5′-GAAACTGCGAATGGCTC-3′<br>
Euk-516R (Reverse Primer): 5′-ACCAGACTTGCCCTCC-3′<br>
These sequences will be used detect and trim primers, rather than trimming by length (which could only remove ambiguous base calls rather than the primers).<br><br>
**Trim primers for the paired-end sequences:**
```
qiime cutadapt trim-paired \
   --i-demultiplexed-sequences clean/ssu/import/ssu-paired-end-casava.qza \
   --p-front-f GAAACTGCGAATGGCTC \
   --p-front-r ACCAGACTTGCCCTCC \
   --o-trimmed-sequences clean/ssu/prim_trim/ssu-prim-trim.qza \
   --verbose

# Convert qiime artifcat into visualization file
qiime demux summarize \
  --i-data clean/ssu/prim_trim/ssu-prim-trim.qza \
  --o-visualization clean/ssu/prim_trim/ssu-prim-trim.qzv
```
Download per sample count .tsv file, saved as "ssu-prim-trim-per-sample-fastq-counts.tsv"<br><br>
Sequence Counts (reads):
Minimum: 101<br>
Median: 24842.0<br>
Mean: 27740.121212<br>
Maximum: 101084<br>
Total: 12815936<br>
These should still match the reads that were observed for the raw import file. If there are less reads, non-target or over trimming may have occured!<br><br>
**Trim primers for the forward sequences:**
```
qiime cutadapt trim-single \
   --i-demultiplexed-sequences clean/ssu/import/ssu-fwd-casava.qza \
   --p-front CAGCCGCGGTAATTCCAGCT \
   --o-trimmed-sequences clean/ssu/prim_trim/ssu-fwd-prim-trim.qza \
   --verbose

qiime demux summarize \
  --i-data clean/ssu/prim_trim/ssu-fwd-prim-trim.qza \
  --o-visualization clean/ssu/prim_trim/ssu-fwd-prim-trim.qzv
```
Download per sample count .tsv file, saved as "ssu-fwd-prim-trim-per-sample-fastq-counts.tsv"<br>
Forward only sequence count summaries should be the same as paired-end! <br>
## STEP 4: DADA2 - Trimming, Merging, Denoising, and Feature Calling of Sequences
This step performs multiple "tests" in order to determine what the best approach should be taken (i.e, maintain paired-end vs. keeping only forward sequences, longer vs. shorter sequence truncation). Three outputs are produced for each test, all of which should be converted into a visualization file and viwed on QIIME 2View. The sample, read, and feature count should all be recorded (see [18s_qiime2_output_summary.csv](https://github.com/alliemeetsfungi/qiime2_amplicon_cleanup/blob/main/18s_qiime2_output_summary.csv) for one approach to do this).
**Paired-end sequences**<br>
<ins>Test 1: De-noise & merge sequences without trimming:</ins>
```
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs clean/ssu/prim_trim/ssu-prim-trim.qza \
  --p-trunc-len-f 300 \
  --p-trunc-len-r 300 \
  --o-table clean/ssu/denoise/no-trim-table.qza \
  --o-representative-sequences clean/ssu/denoise/no-trim-rep-seqs.qza \
  --o-denoising-stats clean/ssu/denoise/no-trim-stats.qza \
  --verbose

## Convert qiime artifcats into visualization files
# Feature table to be used for taxonomic assignment & culling
qiime feature-table summarize \
  --i-table clean/ssu/denoise/no-trim-table.qza \
  --o-visualization clean/ssu/denoise/no-trim-table.qzv

# Sequences of all detected features
qiime feature-table tabulate-seqs \
  --i-data clean/ssu/denoise/no-trim-rep-seqs.qza \
  --o-visualization clean/ssu/denoise/no-trim-rep-seqs.qzv
  
  #downloaded ssu-no-trim-sequences.fasta

# Overall denoising statistics
qiime metadata tabulate \
  --m-input-file clean/ssu/denoise/no-trim-stats.qza \
  --o-visualization clean/ssu/denoise/no-trim-stats.qzv
  
  #downloaded ssu-no-trim-denoise-stats.tsv
```
Total Reads: 378<br>
Sample Count: 462<br>
Features: 172<br>
% samples w/ read retention >25%: 0%<br><br>
<ins>Test 2: De-noise & merge with trimming based on quality plots (liberally):</ins><br>
(Trimming length is chosen based on the length at which the sequences drop below a quality score of 20)
```
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs clean/ssu/prim_trim/ssu-prim-trim.qza \
  --p-trunc-len-f 280 \
  --p-trunc-len-r 230 \
  --o-table clean/ssu/denoise/trimmed-table.qza \
  --o-representative-sequences clean/ssu/denoise/trimmed-rep-seqs.qza \
  --o-denoising-stats clean/ssu/denoise/trimmed-stats.qza \
  --verbose

qiime feature-table summarize \
  --i-table clean/ssu/denoise/trimmed-table.qza \
  --o-visualization clean/ssu/denoise/trimmed-table.qzv

qiime feature-table tabulate-seqs \
  --i-data clean/ssu/denoise/trimmed-rep-seqs.qza \
  --o-visualization clean/ssu/denoise/trimmed-rep-seqs.qzv
  
  #downloaded ssu-trimmed-sequences.fasta

qiime metadata tabulate \
  --m-input-file clean/ssu/denoise/trimmed-stats.qza \
  --o-visualization clean/ssu/denoise/trimmed-stats.qzv
  
  #downloaded ssu-trimmed-denoise-stats.tsv
```
Total Reads: 4,730,916<br>
Sample Count: 462<br>
Features: 9,883<br>
% samples w/ read retention >25%: 87%<br><br>
<ins>Test 3: De-noise & merge with trimming based on quality plots (conservatively):</ins><br>
```
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs clean/ssu/prim_trim/ssu-prim-trim.qza \
  --p-trunc-len-f 275 \
  --p-trunc-len-r 230 \
  --o-table clean/ssu/denoise/short-trimmed-table.qza \
  --o-representative-sequences clean/ssu/denoise/short-trimmed-rep-seqs.qza \
  --o-denoising-stats clean/ssu/denoise/short-trimmed-stats.qza \
  --verbose

qiime feature-table summarize \
  --i-table clean/ssu/denoise/short-trimmed-table.qza \
  --o-visualization clean/ssu/denoise/short-trimmed-table.qzv

qiime feature-table tabulate-seqs \
  --i-data clean/ssu/denoise/short-trimmed-rep-seqs.qza \
  --o-visualization clean/ssu/denoise/short-trimmed-rep-seqs.qzv
  
  #downloaded ssu-short-trimmed-sequences.fasta

qiime metadata tabulate \
  --m-input-file clean/ssu/denoise/short-trimmed-stats.qza \
  --o-visualization clean/ssu/denoise/short-trimmed-stats.qzv
  
  #downloaded ssu-short-trimmed-denoise-stats.tsv
```
Total Reads: 4,808,719<br>
Sample Count: 462<br>
Features: 10,126<br>
% samples w/ read retention >25%: 87%<br><br>
**Forward sequences**
<ins>Test 4: De-noise forward sequences w/out trimming:</ins><br>
```
qiime dada2 denoise-single \
  --i-demultiplexed-seqs clean/ssu/prim_trim/ssu-fwd-prim-trim.qza \
  --p-trunc-len 300 \
  --o-table clean/ssu/denoise/fwd-notrim-table.qza \
  --o-representative-sequences clean/ssu/denoise/fwd-notrim-rep-seqs.qza \
  --o-denoising-stats clean/ssu/denoise/fwd-notrim-stats.qza \
  --verbose

qiime feature-table summarize \
  --i-table clean/ssu/denoise/fwd-notrim-table.qza \
  --o-visualization clean/ssu/denoise/fwd-notrim-table.qzv

qiime feature-table tabulate-seqs \
  --i-data clean/ssu/denoise/fwd-notrim-rep-seqs.qza \
  --o-visualization clean/ssu/denoise/fwd-notrim-rep-seqs.qzv
	
	#downloaded ssd-fwd-notrim-denoise-stats.tsv

qiime metadata tabulate \
  --m-input-file clean/ssu/denoise/fwd-notrim-stats.qza \
  --o-visualization clean/ssu/denoise/fwd-notrim-stats.qzv

	#downloaded ssd-fwd-notrim-sequences.fasta
```
Total Reads: 7,284<br>
Sample Count: 462<br>
Features: 866<br>
% samples w/ read retention >25%: 0%<br><br>
<ins>Test 5: De-noise forward sequences with conservative trimming based on quality plots:</ins><br>
```
qiime dada2 denoise-single \
  --i-demultiplexed-seqs clean/ssu/prim_trim/ssu-fwd-prim-trim.qza \
  --p-trunc-len 265 \
  --o-table clean/ssu/denoise/fwd-table.qza \
  --o-representative-sequences clean/ssu/denoise/fwd-rep-seqs.qza \
  --o-denoising-stats clean/ssu/denoise/fwd-stats.qza \
  --verbose

qiime feature-table summarize \
  --i-table clean/ssu/denoise/fwd-table.qza \
  --o-visualization clean/ssu/denoise/fwd-table.qzv

qiime feature-table tabulate-seqs \
  --i-data clean/ssu/denoise/fwd-rep-seqs.qza \
  --o-visualization clean/ssu/denoise/fwd-rep-seqs.qzv
  
    #downloaded ssu-fwd-sequences.fasta

qiime metadata tabulate \
  --m-input-file clean/ssu/denoise/fwd-stats.qza \
  --o-visualization clean/ssu/denoise/fwd-stats.qzv
  
    #downloaded ssu-fwd-denoise-stats.tsv
```
Sample Count: 462<br>
Total Reads: 7,745,468<br>
Features: 10,839<br>
% samples w/ read retention >25%: 95%<br><br>
<ins>Test 6: De-noise forward sequences with cmore liberal trimming based on quality plots:</ins><br>
```
qiime dada2 denoise-single \
  --i-demultiplexed-seqs clean/ssu/prim_trim/ssu-fwd-prim-trim.qza \
  --p-trunc-len 280 \
  --o-table clean/ssu/denoise/long-fwd-table.qza \
  --o-representative-sequences clean/ssu/denoise/long-fwd-rep-seqs.qza \
  --o-denoising-stats clean/ssu/denoise/long-fwd-stats.qza \
  --verbose

qiime feature-table summarize \
  --i-table clean/ssu/denoise/long-fwd-table.qza \
  --o-visualization clean/ssu/denoise/long-fwd-table.qzv

qiime feature-table tabulate-seqs \
  --i-data clean/ssu/denoise/long-fwd-rep-seqs.qza \
  --o-visualization clean/ssu/denoise/long-fwd-rep-seqs.qzv
  
    #downloaded ssu-long-fwd-sequences.fasta

qiime metadata tabulate \
  --m-input-file clean/ssu/denoise/long-fwd-stats.qza \
  --o-visualization clean/ssu/denoise/long-fwd-stats.qzv
  
    #downloaded ssu-long-fwd-denoise-stats.tsv
```
Total Reads: 6,700,55<br>
Sample Count: 462<br>
Features: 10,050<br>
% samples w/ read retention >25%: 94%<br><br>
### Compare Among Tests
Choose the approach that detected the highest number of features with the largest read retention to move forward with. For this data set this correlates to test 5 ((see [18s_qiime2_output_summary.csv](https://github.com/alliemeetsfungi/qiime2_amplicon_cleanup/blob/main/18s_qiime2_output_summary.csv)).<br>
Files moving foward with are as follows:<br>
1. Feature table: fwd-table.qza
2. Representative Sequences: fwd-rep-seqs.qza
All other files were moved to "tests" directory for efficiency.
## STEP 5 (OPTIONAL!): Cluster ASVs into OTUs
This was performed as a "just-in-case" to have stored so we know that everything was run with the same versions as the ASV table if needed.<br><br>
Cluster ASVs into OTUs:
```
qiime vsearch cluster-features-de-novo \
  --i-table clean/ssu/denoise/fwd-table.qza \
  --i-sequences clean/ssu/denoise/fwd-rep-seqs.qza \
  --p-perc-identity 0.97 \
  --o-clustered-table clean/ssu/otu_clustering/ssu-otu-table.qza \
  --o-clustered-sequences clean/ssu/otu_clustering/ssu-otu-rep-seqs.qza \
  --verbose
  
# Convert all artifact files into visual files
# Feature table to be used for taxonomic assignment
qiime feature-table summarize \
  --i-table clean/ssu/otu_clustering/ssu-otu-table.qza \
  --o-visualization clean/ssu/otu_clustering/ssu-otu-table.qzv
  
# Sequences for all detected features	
qiime feature-table tabulate-seqs \
  --i-data clean/ssu/otu_clustering/ssu-otu-rep-seqs.qza \
  --o-visualization clean/ssu/otu_clustering/ssu-otu-rep-seqs.qzv
  
    #downloaded ssu-otu-sequences.fasta
```
NOTE: The denoising statistics file (fwd-stats.qza) is still applicable to the OTU table :)<br>
## STEP 6 (OPTIONAL!): Export Feature Table For Culling
Preliminary culling thresholds were observed using the feature table while taxonomic assignment runs. The final asv and sample culling was performed once the final taxa table was created. See [QIIME2_SOP.md](https://github.com/alliemeetsfungi/qiime2_amplicon_cleanup/blob/main/QIIME 2_SOP.md) for more  details on this step.<br><br>
Export feature table (QIIME 2 table.qza = feature table):
```
qiime tools export \
  --input-path clean/ssu/denoise/trimmed-table.qza \
  --output-path clean/ssu/feature-table
```
Convert feature table to TSV format:
```
biom convert \
  --input-fp clean/ssu/feature-table/feature-table.biom \
  --output-fp clean/ssu/feature-table/feature-table.tsv \
  --to-tsv
```
Convert .tsv to .csv:
```
sed 's/\t/,/g' clean/ssu/feature-table/feature-table.tsv > clean/ssu/feature-table/feature-table.csv
```
In terminal NOT signed in to KOA (aka local drive):
```
scp alliej@koa.its.hawaii.edu:/home/alliej/hynson_koastore/ajhall/bros/clean/ssu/feature-table/feature-table.csv \
~/Downloads/
```
Feature table was then imported into R Studio for preliminary culling using Phyloseq.<br>
## STEP 7: Import Databases For Taxonomic Identification
This dataset will use tje Eukaryome Version 2.0, NCBI, and SILVA Version 138.2 databases for taxonomic assignment.<br><br>
Import Eukaryome V2.0
```
#import reference sequences
qiime tools import --type 'FeatureData[Sequence]' \
  --input-path database_files/QIIME2_EUK_SSU_v2.0/QIIME2_EUK_SSU_v2.0.fasta \
  --output-path database_files/eukaryome-ref-seqs.qza
  
#import taxonomy
qiime tools import --type 'FeatureData[Taxonomy]' \
  --input-format HeaderlessTSVTaxonomyFormat \
  --input-path database_files/QIIME2_EUK_SSU_v2.0/QIIME2_EUK_SSU_v2.0.tsv \
  --output-path database_files/eukaryome-ref-tax.qza
```
<br>

Import NCBI using RESCRIPt to pull both the reference sequences and taxonomy:
```
qiime rescript get-ncbi-data \
  --p-query '18S[ALL] AND fungi[ORGN]' \
  --o-sequences database_files/ncbi-fungi-ref-seqs.qza \
  --o-taxonomy database_files/ncbi-fungi-ref-tax.qza \
  --p-n-jobs 5
```
<br>

Import SILVA 138.2 using RESCRIPt to pull both the reference sequences and taxonomy (see [THIS](https://forum.qiime2.org/t/sequence-and-taxonomy-files-for-silva-v138-2/33475) forum for more details:
```
qiime rescript get-silva-data \
  --p-version '138.2' \
  --p-target 'SSURef_NR99' \
  --o-silva-sequences database_files/SILVA_138.2/silva-138.2-rna-ref-seqs.qza \
  --o-silva-taxonomy database_files/silva-138.2-ref-tax.qza
```
The reference sequences pulled are in RNA format (UGCA), need to reverse transcribed into DNA format (TGCA)!
```
# Reverse transcribe reference sequences (rRNA) for assignment compatibility (rDNA)
qiime rescript reverse-transcribe \
  --i-rna-sequences database_files/SILVA_138.2/silva-138.2-rna-ref-seqs.qza \
  --o-dna-sequences database_files/silva-138.2-ref-seqs.qza
```
<br>
Databases only need to be imported once to make the .qza files, afterwards this step can be skipped for additional datasets :)<br>

## STEP 8: Taxonomic Assignment To Features
The same taxonomic approach was used for OTU taxonomic assignment with all but SILVA 95% contributing towards the taxa table. Output directory and files were saved to the taxa_id/ssu/otu directory.

**STEP 8.1  Eukaryome**
<ins>95% ID</ins><br>
95% identity = --p-perc-identity 0.95<br>
90% query coverage = --p-query-cov 0.90 (ALWAYS)
```
qiime feature-classifier classify-consensus-blast \
  --i-query clean/ssu/denoise/fwd-rep-seqs.qza \ #use final rep-seqs.qza file from cleanup
  --i-reference-reads database_files/eukaryome-ref-seqs.qza \ #use associated databases ref-seq.qza file
  --i-reference-taxonomy database_files/eukaryome-ref-tax.qza \ #use associated databases ref-tax.qza file
  --p-maxaccepts 1 \
  --p-perc-identity 0.95 \ #will change in subsequent runs
  --p-query-cov 0.90 \ 
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir taxa_id/ssu/euk-95 #makes new directory, will err if you pre-make it!

# Filter out unassigned sequences into their own file
qiime taxa filter-seqs \
  --i-sequences clean/ssu/denoise/fwd-rep-seqs.qza \ #use STARTING rep-seqs.qza file
  --i-taxonomy taxa_id/ssu/euk-95/classification.qza \ #input resulting classification.qza file
  --p-include unassigned \
  --o-filtered-sequences taxa_id/ssu/euk-95/unassigned-rep-seqs.qza #remaining sequences that did not have taxa assigned
```
<br><ins>90% ID</ins>
```
qiime feature-classifier classify-consensus-blast \
  --i-query taxa_id/ssu/euk-95/unassigned-rep-seqs.qza \ #change to final unnassigned taxa rep-seqs.qza to fill in the unknowns!
  --i-reference-reads database_files/eukaryome-ref-seqs.qza \ #stays the same
  --i-reference-taxonomy database_files/eukaryome-ref-tax.qza \ #stays the same
  --p-maxaccepts 1 \
  --p-perc-identity 0.90 \ #change to new desired %ID
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir taxa_id/ssu/euk-90 #new final directory

qiime taxa filter-seqs \
  --i-sequences taxa_id/ssu/euk-95/unassigned-rep-seqs.qza \ #use rep-seqs.qza file from --i-query
  --i-taxonomy taxa_id/ssu/euk-90/classification.qza \ #input resulting classification.qza file
  --p-include unassigned \
  --o-filtered-sequences taxa_id/ssu/euk-90/unassigned-rep-seqs.qza #remaining sequences that did not have taxa assigned
```
<br><ins>80% ID</ins>
```
qiime feature-classifier classify-consensus-blast \
  --i-query taxa_id/ssu/euk-90/unassigned-rep-seqs.qza \
  --i-reference-reads database_files/eukaryome-ref-seqs.qza \
  --i-reference-taxonomy database_files/eukaryome-ref-tax.qza \
  --p-maxaccepts 1 \
  --p-perc-identity 0.80 \
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir taxa_id/ssu/euk-80

qiime taxa filter-seqs \
  --i-sequences taxa_id/ssu/euk-90/unassigned-rep-seqs.qza \
  --i-taxonomy taxa_id/ssu/euk-80/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences taxa_id/ssu/euk-80/unassigned-rep-seqs.qza
```
Run the remaining unassignd sequences through the next databse!<br><br>
**STEP 8.2 NCBI**
<br><ins>95% ID</ins>
```
qiime feature-classifier classify-consensus-blast \
  --i-query taxa_id/ssu/euk-80/unassigned-rep-seqs.qza \ #use final unnassigned rep-seqs.qza file from previous database search
  --i-reference-reads database_files/ncbi-fungi-ref-seqs.qza \ #change to new database ref-seqs.qza
  --i-reference-taxonomy database_files/ncbi-fungi-ref-tax.qza \ #change to new database ref-tax.qza
  --p-maxaccepts 1 \
  --p-perc-identity 0.95 \ #don't forget to change back to 95%
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir taxa_id/ssu/ncbi-95 #make new directory for new database

qiime taxa filter-seqs \
  --i-sequences taxa_id/ssu/euk-80/unassigned-rep-seqs.qza \
  --i-taxonomy taxa_id/ssu/ncbi-95/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences taxa_id/ssu/ncbi-95/unassigned-rep-seqs.qza
```
<br><ins>90% ID</ins>
```
qiime feature-classifier classify-consensus-blast \
  --i-query taxa_id/ssu/ncbi-95/unassigned-rep-seqs.qza \
  --i-reference-reads database_files/ncbi-fungi-ref-seqs.qza \
  --i-reference-taxonomy database_files/ncbi-fungi-ref-tax.qza \
  --p-maxaccepts 1 \
  --p-perc-identity 0.90 \
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir taxa_id/ssu/ncbi-90

qiime taxa filter-seqs \
  --i-sequences taxa_id/ssu/ncbi-95/unassigned-rep-seqs.qza \
  --i-taxonomy taxa_id/ssu/ncbi-90/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences taxa_id/ssu/ncbi-90/unassigned-rep-seqs.qza
```
<br><ins>80% ID</ins>
```
qiime feature-classifier classify-consensus-blast \
  --i-query taxa_id/ssu/ncbi-90/unassigned-rep-seqs.qza \
  --i-reference-reads database_files/ncbi-fungi-ref-seqs.qza \
  --i-reference-taxonomy database_files/ncbi-fungi-ref-tax.qza \
  --p-maxaccepts 1 \
  --p-perc-identity 0.80 \
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir taxa_id/ssu/ncbi-80

qiime taxa filter-seqs \
  --i-sequences taxa_id/ssu/ncbi-90/unassigned-rep-seqs.qza \
  --i-taxonomy taxa_id/ssu/ncbi-80/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences taxa_id/ssu/ncbi-80/unassigned-rep-seqs.qza
```
Run the remaining unassignd sequences through the next databse.<br><br>
**STEP 8.3 SILVA 138.2**
<br><ins>95% ID</ins>
```
qiime feature-classifier classify-consensus-blast \
  --i-query taxa_id/ssu/ncbi-80/unassigned-rep-seqs.qza \
  --i-reference-reads database_files/silva-138.2-ref-seqs.qza \
  --i-reference-taxonomy database_files/silva-138.2-ref-tax.qza \
  --p-maxaccepts 1 \
  --p-perc-identity 0.95 \
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir taxa_id/ssu/silva-95

qiime taxa filter-seqs \
  --i-sequences taxa_id/ssu/ncbi-80/unassigned-rep-seqs.qza \
  --i-taxonomy taxa_id/ssu/silva-95/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences taxa_id/ssu/silva-95/unassigned-rep-seqs.qza
```
<br><ins>90% ID</ins>
```
qiime feature-classifier classify-consensus-blast \
  --i-query taxa_id/ssu/silva-95/unassigned-rep-seqs.qza \
  --i-reference-reads database_files/silva-138.2-ref-seqs.qza \
  --i-reference-taxonomy database_files/silva-138.2-ref-tax.qza \
  --p-maxaccepts 1 \
  --p-perc-identity 0.90 \
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir taxa_id/ssu/silva-90

qiime taxa filter-seqs \
  --i-sequences taxa_id/ssu/silva-95/unassigned-rep-seqs.qza \
  --i-taxonomy taxa_id/ssu/silva-90/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences taxa_id/ssu/silva-90/unassigned-rep-seqs.qza
```
<br><ins>80% ID</ins>
```
qiime feature-classifier classify-consensus-blast \
  --i-query taxa_id/ssu/silva-90/unassigned-rep-seqs.qza \
  --i-reference-reads database_files/silva-138.2-ref-seqs.qza \
  --i-reference-taxonomy database_files/silva-138.2-ref-tax.qza \
  --p-maxaccepts 1 \
  --p-perc-identity 0.80 \
  --p-query-cov 0.90 \
  --p-strand both \
  --p-evalue 1e-50 \
  --p-min-consensus 0.51 \
  --output-dir taxa_id/ssu/silva-80

qiime taxa filter-seqs \
  --i-sequences taxa_id/ssu/silva-90/unassigned-rep-seqs.qza \
  --i-taxonomy taxa_id/ssu/silva-80/classification.qza \
  --p-include unassigned \
  --o-filtered-sequences taxa_id/ssu/silva-80/unassigned-rep-seqs.qza
```
<br>

## STEP 9 (OPTIONAL!): Filtering Feature Tables
Filter final ASV table to make tables containing all identified and not identified features.
<br>NOTE: Final ASV table should only be used for initial filtering, use the unassigned output file for each filtering run for subsequent filtering steps.<br><br>
**STEP 9.1 Eukaryome**
95% ID: filter for assigned vs unassigned from the original dada2 table (trimmed-table.qza)
```
qiime taxa filter-table \
  --i-table clean/ssu/denoise/fwd-table.qza \
  --i-taxonomy taxa_id/ssu/euk-95/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table taxa_id/ssu/euk-95/assigned-table.qza

qiime taxa filter-table \
  --i-table clean/ssu/denoise/fwd.qza \
  --i-taxonomy taxa_id/ssu/euk-95/classification.qza \
  --p-include unassigned \
  --o-filtered-table taxa_id/ssu/euk-95/unassigned-table.qza
```
90% ID: use resulting unassigned-table.qza as input table
```
qiime taxa filter-table \
  --i-table taxa_id/ssu/euk-95/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/euk-90/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table taxa_id/ssu/euk-90/assigned-table.qza

qiime taxa filter-table \
  --i-table taxa_id/ssu/euk-95/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/euk-90/classification.qza \
  --p-include unassigned \
  --o-filtered-table taxa_id/ssu/euk-90/unassigned-table.qza
```
80% ID
```
qiime taxa filter-table \
  --i-table taxa_id/ssu/euk-90/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/euk-80/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table taxa_id/ssu/euk-80/assigned-table.qza

qiime taxa filter-table \
  --i-table taxa_id/ssu/euk-90/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/euk-80/classification.qza \
  --p-include unassigned \
  --o-filtered-table taxa_id/ssu/euk-80/unassigned-table.qza
```
<br>

**STEP 9.2 NCBI**
Use the resulting unassigned-table.qza table that holds the remaining unassigned features after eukaryome 80%
```
# 95% ID
qiime taxa filter-table \
  --i-table taxa_id/ssu/euk-80/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/ncbi-95/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table taxa_id/ssu/ncbi-95/assigned-table.qza

qiime taxa filter-table \
  --i-table taxa_id/ssu/euk-80/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/ncbi-95/classification.qza \
  --p-include unassigned \
  --o-filtered-table taxa_id/ssu/ncbi-95/unassigned-table.qza


# 90% ID
qiime taxa filter-table \
  --i-table taxa_id/ssu/ncbi-95/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/ncbi-90/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table taxa_id/ssu/ncbi-90/assigned-table.qza

qiime taxa filter-table \
  --i-table taxa_id/ssu/ncbi-95/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/ncbi-90/classification.qza \
  --p-include unassigned \
  --o-filtered-table taxa_id/ssu/ncbi-90/unassigned-table.qza


# 80% ID
qiime taxa filter-table \
  --i-table taxa_id/ssu/ncbi-90/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/ncbi-80/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table taxa_id/ssu/ncbi-80/assigned-table.qza

qiime taxa filter-table \
  --i-table taxa_id/ssu/ncbi-90/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/ncbi-80/classification.qza \
  --p-include unassigned \
  --o-filtered-table taxa_id/ssu/ncbi-80/unassigned-table.qza
```
<br>

**STEP 9.3 SILVA**
Use the resulting unassigned-table.qza table that holds the remaining unassigned features after NCBI 80%
```
# 95% ID
qiime taxa filter-table \
  --i-table taxa_id/ssu/ncbi-80/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/silva-95/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table taxa_id/ssu/silva-95/assigned-table.qza

qiime taxa filter-table \
  --i-table taxa_id/ssu/ncbi-80/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/silva-95/classification.qza \
  --p-include unassigned \
  --o-filtered-table taxa_id/ssu/silva-95/unassigned-table.qza


# 90%
qiime taxa filter-table \
  --i-table taxa_id/ssu/ncbi-80/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/silva-90/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table taxa_id/ssu/silva-90/assigned-table.qza

qiime taxa filter-table \
  --i-table taxa_id/ssu/ncbi-80/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/silva-90/classification.qza \
  --p-include unassigned \
  --o-filtered-table taxa_id/ssu/silva-90/unassigned-table.qza


# 80%
qiime taxa filter-table \
  --i-table taxa_id/ssu/silva-90/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/silva-80/classification.qza \
  --p-exclude unassigned \
  --o-filtered-table taxa_id/ssu/silva-80/assigned-table.qza

qiime taxa filter-table \
  --i-table taxa_id/ssu/silva-90/unassigned-table.qza \
  --i-taxonomy taxa_id/ssu/silva-80/classification.qza \
  --p-include unassigned \
  --o-filtered-table taxa_id/ssu/silva-80/unassigned-table.qza
```
<br>

**STEP 9.4 Merge Tables**
Combine all assigned-table.qza files to make a single table containing all assigned features:
```
qiime feature-table merge \
  --i-tables taxa_id/ssu/euk-95/assigned-table.qza \
  --i-tables taxa_id/ssu/euk-90/assigned-table.qza \
  --i-tables taxa_id/ssu/euk-80/assigned-table.qza \
  --i-tables taxa_id/ssu/ncbi-95/assigned-table.qza \
  --i-tables taxa_id/ssu/ncbi-90/assigned-table.qza \
  --i-tables taxa_id/ssu/ncbi-80/assigned-table.qza \
  --i-tables taxa_id/ssu/silva-95/assigned-table.qza \
  --i-tables taxa_id/ssu/silva-90/assigned-table.qza \
  --i-tables taxa_id/ssu/silva-80/assigned-table.qza \
  --o-merged-table taxa_id/ssu/ssu-assigned-table.qza \
  --p-overlap-method sum

qiime feature-table summarize \
  --i-table taxa_id/ssu/ssu-assigned-table.qza \
  --o-visualization taxa_id/ssu/ssu-assigned-table.qzv
```
Samples: 456<br>
Features: 9,192<br>
Reads: 7,581,774<br><br>
The final unassigned-table.qza created from silva-80 holds all features that were not able to be assigned.<br>
Copy and rename final unassinged table into folder containing all final taxa documents (taxa_id/ssu)
```
cp taxa_id/ssu/silva-80/unassigned-table.qza taxa_id/ssu/ssu-unassigned-table.qza

qiime feature-table summarize \
  --i-table taxa_id/ssu/ssu-unassigned-table.qza \
  --o-visualization taxa_id/ssu/ssu-unassigned-table.qzv
```
Samples: 405<br>
Features: 1,647<br>
Reads: 163,694<br><br>
As as a gut check, merge assigned and unassigned table to see if the sample, feature, and read counts are the same as the table they were originally filtered from (fwd-table.qza)
```
qiime feature-table merge \
  --i-tables taxa_id/ssu/ssu-assigned-table.qza \
  --i-tables taxa_id/ssu/ssu-unassigned-table.qza \
  --o-merged-table taxa_id/ssu/ssu-all-table.qza \
  --p-overlap-method sum

qiime feature-table summarize \
  --i-table taxa_id/ssu/ssu-all-table.qza \
  --o-visualization taxa_id/ssu/ssu-all-table.qzv
```
Samples: 461<br>
Features: 10,839<br>
Frequency: 7,745,468<br><br>
The same code was run for the ssu OTU table (ssu-otu-table.qza) and counts are as follows:
1. ssu-otu-assigned-table.qza:
Samples: 451<br>
Features: 4,141<br>
Reads: 5,032,398<br>
2. ssu-otu-unassigned-table.qza
Samples: 339<br>
Features: 682<br>
Reads: 85,882<br>
3. #ssu-otu-all-table.qza
Samples: 455<br>
Features: 4,823<br>
Reads: 5,118,280<br>
## STEP 10: Merging Classification Files
Check counts for all classification files before merging !!!
```
# Eukaryome
qiime metadata tabulate \
  --m-input-file taxa_id/ssu/euk-95/classification.qza \
  --o-visualization taxa_id/ssu/euk-95/classification.qzv
#6,346

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/euk-90/classification.qza \
  --o-visualization taxa_id/ssu/euk-90/classification.qzv
#2,090

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/euk-80/classification.qza \
  --o-visualization taxa_id/ssu/euk-80/classification.qzv
#633

# Eukaryome total = 9,069


# NCBI
qiime metadata tabulate \
  --m-input-file taxa_id/ssu/ncbi-95/classification.qza \
  --o-visualization taxa_id/ssu/ncbi-95/classification.qzv
#16
 
qiime metadata tabulate \
  --m-input-file taxa_id/ssu/ncbi-90/classification.qza \
  --o-visualization taxa_id/ssu/ncbi-90/classification.qzv
#18

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/ncbi-80/classification.qza \
  --o-visualization taxa_id/ssu/ncbi-80/classification.qzv
#45

# NCBI Total = 79

# SILVA
qiime metadata tabulate \
  --m-input-file taxa_id/ssu/silva-95/classification.qza \
  --o-visualization taxa_id/ssu/silva-95/classification.qzv
#3

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/silva-90/classification.qza \
  --o-visualization taxa_id/ssu/silva-90/classification.qzv
#27

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/silva-80/classification.qza \
  --o-visualization taxa_id/ssu/silva-80/classification.qzv
#14

# SILVA Total = 44
```
There were 9,192 total assigned features found within the classification files.<br><br>
Merge classification files: <ins>NOTE: tables only merge IF the lower % ID file is listed first<br>
```
# Eukaryome
qiime feature-table merge-taxa \
  --i-data taxa_id/ssu/euk-90/classification.qza \
  --i-data taxa_id/ssu/euk-95/classification.qza \
  --o-merged-data taxa_id/ssu/euk_partial_merged_taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/euk_partial_merged_taxonomy.qza \
  --o-visualization taxa_id/ssu/euk_partial_merged_taxonomy.qzv
#8,436


qiime feature-table merge-taxa \
  --i-data taxa_id/ssu/euk-80/classification.qza \
  --i-data taxa_id/ssu/euk_partial_merged_taxonomy.qza \
  --o-merged-data taxa_id/ssu/euk_merged_taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/euk_merged_taxonomy.qza \
  --o-visualization taxa_id/ssu/euk_merged_taxonomy.qzv
#9,069


# NCBI
qiime feature-table merge-taxa \
  --i-data taxa_id/ssu/ncbi-90/classification.qza \
  --i-data taxa_id/ssu/ncbi-95/classification.qza \
  --o-merged-data taxa_id/ssu/ncbi_partial_merged_taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/ncbi_partial_merged_taxonomy.qza \
  --o-visualization taxa_id/ssu/ncbi_partial_merged_taxonomy.qzv
#34

  
qiime feature-table merge-taxa \
  --i-data taxa_id/ssu/ncbi-80/classification.qza \
  --i-data taxa_id/ssu/ncbi_partial_merged_taxonomy.qza \
  --o-merged-data taxa_id/ssu/ncbi_merged_taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/ncbi_merged_taxonomy.qza \
  --o-visualization taxa_id/ssu/ncbi_merged_taxonomy.qzv
#79


# Eukaryome + NCBI
qiime feature-table merge-taxa \
  --i-data taxa_id/ssu/ncbi_merged_taxonomy.qza \
  --i-data taxa_id/ssu/euk_merged_taxonomy.qza \
  --o-merged-data taxa_id/ssu/euk_ncbi_merged_taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/euk_ncbi_merged_taxonomy.qza \
  --o-visualization taxa_id/ssu/euk_ncbi_merged_taxonomy.qzv
#9,148


# SILVA 
qiime feature-table merge-taxa \
  --i-data taxa_id/ssu/silva-90/classification.qza \
  --i-data taxa_id/ssu/silva-95/classification.qza \
  --o-merged-data taxa_id/ssu/silva_partial_merged_taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/silva_partial_merged_taxonomy.qza \
  --o-visualization taxa_id/ssu/silva_partial_merged_taxonomy.qzv
#30


qiime feature-table merge-taxa \
  --i-data taxa_id/ssu/silva-80/classification.qza \
  --i-data taxa_id/ssu/silva_partial_merged_taxonomy.qza \
  --o-merged-data taxa_id/ssu/silva_merged_taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/silva_merged_taxonomy.qza \
  --o-visualization taxa_id/ssu/silva_merged_taxonomy.qzv
#44


# FINAL MERGE
qiime feature-table merge-taxa \
  --i-data taxa_id/ssu/silva_merged_taxonomy.qza \
  --i-data taxa_id/ssu/euk_ncbi_merged_taxonomy.qza \
  --o-merged-data taxa_id/ssu/ssu-taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxa_id/ssu/ssu-taxonomy.qza \
  --o-visualization taxa_id/ssu/ssu-taxonomy.qzv
# 10,839
```
Summary of final taxa table:<br>
Total features = 10,839<br>
Total Unassigned = 1,647<br>
Total assigned = 9,192<br><br>
The same code was run for the ssu OTU table (ssu-otu-table.qza) and counts are as follows:
1. Eukaryome: 4,028<br>
euk-95: 1,822<br>
euk-90: 1,758<br>
euk-80: 448<br>
2. NCBI: 100<br>
ncbi-95: 3<br>
ncbi-90: 5<br>
ncbi-80: 92<br>
3. SILVA: 13<br>
silva-90: 7<br>
silva-80: 6<br>
4. Total Assigned Features: 4,141
5. Final output file for SSU OTU taxonomic identification is ssu-otu-taxonomy.qza<br>
Total Features: 4,823<br>
Total Unassigned: 682<br>
Total Assigned: 4,141<br><br>
## STEP 11: Export Final Tables And Representative Sequences
The final files to be exported are as follows:
1. Feature/ASV table: fwd-table.qza
2. Representative Sequences: fwd-rep-seqs.qza
3. Taxonomy Table: ssu-taxonomy.qza
<br>
**Export Feature Table (only if step 5 was skipped):**
```
qiime tools export \
  --input-path clean/ssu/denoise/fwd-table.qza \
  --output-path clean/ssu/feature-table

# Convert feature table to TSV format
biom convert \
  --input-fp clean/ssu/feature-table/feature-table.biom \
  --output-fp clean/ssu/feature-table/ssu-feature-table.tsv \
  --to-tsv

# Convert .tsv to .csv
sed 's/\t/,/g' clean/ssu/feature-table/ssu-feature-table.tsv > clean/ssu/feature-table/ssu-feature-table.csv
```
<br>

**Export Representative Sequences:**
```
qiime tools export \
  --input-path clean/ssu/denoise/fwd-rep-seqs.qza \
  --output-path clean/ssu/rep-seqs

# Rename for clarity
mv clean/ssu/rep-seqs/dna-sequences.fasta clean/ssu/rep-seqs/ssu-rep-sequences.fasta
```
<br>

**Export Taxonomy Table:**
```
qiime tools export \
  --input-path taxa_id/ssu/ssu-taxonomy.qza \
  --output-path clean/ssu/taxonomy

# Rename for clarity
mv clean/ssu/taxonomy/taxonomy.tsv clean/ssu/taxonomy/ssu-taxonomy.tsv

# Convert .tsv to .csv
sed 's/\t/,/g' clean/ssu/taxonomy/ssu-taxonomy.tsv > clean/ssu/taxonomy/ssu-taxonomy.csv
```
<br>

**Download to local drive**
In local terminal NOT signed in to KOA (aka local drive):
```
# Download the entire directory
scp -r alliej@koa.its.hawaii.edu:/home/alliej/hynson_koastore/ajhall/bros/clean/ \
~/Downloads/

# Download a single file if need be
scp alliej@koa.its.hawaii.edu:/home/alliej/hynson_koastore/ajhall/bros/clean/ssu/taxonomy/ssu-taxonomy.csv \
~/Downloads/
```



