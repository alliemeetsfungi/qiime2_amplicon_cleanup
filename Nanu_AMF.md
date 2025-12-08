NOTE: The steps for the code below are directly correlated to the steps outlined in the Qiime2_SOP, thus some jumps will be observed between the numbered steps.<br><br>
## STEP 1: Prepare Files & Environment
Activate Qiime2 in terminal command line
```
conda activate /Users/yo/miniconda/envs/qiime2-amplicon-2024.10
```
## STEP 2: Importing Sequences Into Qiime2
This step uses the “Fastq manifest" approach to importing sequences into a Qiime artifact:
```
# Importing Sequences
qiime tools import \
  --type 'SampleData[SequencesWithQuality]' \
  --input-path FINAL/clean/fwd-manifest-Ph33 \
  --output-path FINAL/clean/demux.qza \
  --input-format SingleEndFastqManifestPhred33V2

# Convert for visualization
qiime demux summarize \
  --i-data FINAL/clean/demux.qza \
  --o-visualization FINAL/clean/vis/demux.qzv
```
NOTE: If you recieved sequences from the UH sequencing center (ASGPB) then you should be importing with the Cassava approach. These data were sequenced at UC Irvine and thus required a different approach.<br><br>
<ins>Demultiplexed sequence counts summary (forward reads)</ins><br>
Minimum	91<br>
Median	91590.0<br>
Mean	  86460.191083<br>
Maximum	303187<br>
Total	  13574250<br><br>
Downloaded per sample sequence counts .tsv file and saved as raw-per-sample-fastq-counts.tsv for reads per sample breakdown.<br>
## STEP 4: DADA2 - Trimming, Merging, Denoising, and Feature Calling of Sequences
In this version, I trimmed primers by length instead of sequence. See the Qiime2_SOP file for issues that can occur from this approach.<br><br>
Trim primers by size (20 bp) and truncate forward reads at 275 bp:
```
qiime dada2 denoise-single \
  --i-demultiplexed-seqs FINAL/clean/demux.qza \
  --p-trim-left 20 \
  --p-trunc-len 275 \
  --o-table FINAL/clean/nanu-table.qza \
  --o-representative-sequences FINAL/clean/nanu-rep-seqs.qza \
  --o-denoising-stats FINAL/clean/nanu-denoise-stats.qza \
  --verbose
```
Visualize Feature Table to be used for taxonomic assignment:
```
qiime feature-table summarize \
  --i-table FINAL/clean/nanu-table.qza \
  --o-visualization FINAL/clean/vis/nanu-table.qzv
```
*Observed details from Qiime2 Viewer:*
<br>Number of samples = 157<br>
Number of features = 6,229<br>
Total frequency = 10,132,044<br><br>
*Frequency per sample:*
<br>Minimum frequency = 1<br>
1st quartile = 26,101<br>
Median frequency = 70,282<br>
3rd quartile = 93,742<br>
Maximum frequency = 236,660<br>
Mean frequency = 64,535.3<br><br>
*Frequency per feature*
<br>Minimum frequency = 1<br>
1st quartile = 5<br>
Median frequency = 16<br>
3rd quartile = 110<br>
Maximum frequency = 682,915<br>
Mean frequency = 1,626.6<br><br>
Visualize Representative Sequences:
```
qiime feature-table tabulate-seqs \
  --i-data FINAL/clean/nanu-rep-seqs.qza \
  --o-visualization FINAL/clean/vis/nanu-reps-seqs.qzv
```
Observed details from Qiime2 Viewer:<br>
sequence count = 6229<br>
sequence length = 255<br><br>
Visualize overall denoising statistics:
```
qiime metadata tabulate \
  --m-input-file FINAL/clean/nanu-denoise-stats.qza \
  --o-visualization FINAL/clean/vis/nanu-denoise-stats.qzv
```
Downloaded denoising-stats.tsv for local access to the denoising overview

## STEP 7: Import Databases For Taxonomic Identification
<ins>Maarjam 2021</ins><br>
From MaarjAM webpage, the MaarjAM VT sequences of 18S rDNA gene region QIIME release (2021) were downloaded and then imported:
```
# Import Reference Sequences
qiime tools import --type 'FeatureData[Sequence]' \
--input-path Database_files/maarjam_database_SSU.qiime.2021/maarjam_database_SSU.qiime.fasta \
--output-path Database_files/maarjam-ref-seqs.qza

# Import Taxonomy
qiime tools import --type 'FeatureData[Taxonomy]' \
--input-format HeaderlessTSVTaxonomyFormat \
--input-path Database_files/maarjam_database_SSU.qiime.2021/maarjam_database_SSU.qiime.txt \
--output-path Database_files/maarjam-ref-tax.qza
```
<ins>Eukaryome</ins><br>
From the Eukaryome webpage QIIME2_EUK_SSU Version 1.9.4 was downloaded
```
qiime tools import --type 'FeatureData[Sequence]' \
--input-path Database_files/QIIME2_EUK_SSU_v1.9.4/QIIME2_EUK_SSU_v1.9.4.fasta \
--output-path Database_files/eukaryome-ref-seqs.qza
  
qiime tools import --type 'FeatureData[Taxonomy]' \
--input-format HeaderlessTSVTaxonomyFormat \
--input-path Database_files/QIIME2_EUK_SSU_v1.9.4/QIIME2_EUK_SSU_v1.9.4.tsv \
--output-path Database_files/eukaryome-ref-tax.qza
```
<br>
<ins>NCBI filtered for Glomeromycotina</ins><br>
NCBI information is directly pulled into Qiime2 without downloading files from the webpage:
```
qiime rescript get-ncbi-data \
--p-query '18S[ALL] AND glomeromycotina [ORGN]' \
--o-sequences Database_files/ncbi-glom-refseqs-unfiltered.qza \
--o-taxonomy Database_files/ncbi-glom-taxonomy-unfiltered.qza \
--p-n-jobs 5
```
<br>
<ins>SILVA</ins><br>
From the SILVA webpage Version 132 was downloaded:
```
qiime tools import --type 'FeatureData[Sequence]' \
--input-path Database_files/SILVA_132_QIIME_release/rep_set/rep_set_all/99/silva132_99.fna \
--output-path FINAL/Database_files/silva-ref-seqs.qza

qiime tools import --type 'FeatureData[Taxonomy]' \
--input-format HeaderlessTSVTaxonomyFormat \
--input-path Database_files/SILVA_132_QIIME_release/taxonomy/taxonomy_all/99/taxonomy_all_levels.txt \
--output-path Database_files/silva-ref-tax.qza
```


## STEP 8: Taxonomic Assignment To Features

## 4.1 Maarjam 2021 ##
######################

## 4.1.1 95% ID ##
##################

# 95% identity = --p-perc-identity 0.95
# 90% query coverage = --p-query-cov 0.90 (ALWAYS)
qiime feature-classifier classify-consensus-blast \
--i-query FINAL/clean/nanu-rep-seqs.qza \
--i-reference-reads FINAL/Database_files/maarjam-ref-seqs.qza \
--i-reference-taxonomy FINAL/Database_files/maarjam-ref-tax.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.95 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/maarjam-95

# filter out unassigned sequences into their own file
qiime taxa filter-seqs \
--i-sequences FINAL/clean/nanu-rep-seqs.qza \
--i-taxonomy FINAL/taxa/maarjam-95/classification.qza \
--p-include unassigned \
--o-filtered-sequences FINAL/taxa/maarjam-95/unassigned-rep-seqs-95.qza

## 4.1.2 90% ID ##
##################

# 90% identity = --p-perc-identity 0.90
qiime feature-classifier classify-consensus-blast \
--i-query FINAL/taxa/maarjam-95/unassigned-rep-seqs-95.qza \
--i-reference-reads FINAL/Database_files/maarjam-ref-seqs.qza \
--i-reference-taxonomy FINAL/Database_files/maarjam-ref-tax.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.90 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/maarjam-90

# filter out unassigned sequences into their own file
qiime taxa filter-seqs \
--i-sequences FINAL/taxa/maarjam-95/unassigned-rep-seqs-95.qza \
--i-taxonomy FINAL/taxa/maarjam-90/classification.qza \
--p-include unassigned \
--o-filtered-sequences FINAL/taxa/maarjam-90/unassigned-rep-seqs-90.qza

## 4.1.3 80% ID ##
##################

# 80% identity = --p-perc-identity 0.80
qiime feature-classifier classify-consensus-blast \
--i-query FINAL/taxa/maarjam-90/unassigned-rep-seqs-90.qza \
--i-reference-reads FINAL/Database_files/maarjam-ref-seqs.qza \
--i-reference-taxonomy FINAL/Database_files/maarjam-ref-tax.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.80 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/maarjam-80

# filter out unassigned sequences into their own file
qiime taxa filter-seqs \
--i-sequences FINAL/taxa/maarjam-90/unassigned-rep-seqs-90.qza \
--i-taxonomy FINAL/taxa/maarjam-80/classification.qza \
--p-include unassigned \
--o-filtered-sequences FINAL/taxa/maarjam-80/unassigned-rep-seqs-80.qza

#run remaining unassignd sequences through the next databse!


## 4.2 Eukaryome ##
###################

## 4.2.1 95% ID ##
##################

qiime feature-classifier classify-consensus-blast \
--i-query FINAL/taxa/maarjam-80/unassigned-rep-seqs-80.qza \
--i-reference-reads FINAL/Database_files/eukaryome-ref-seqs.qza \
--i-reference-taxonomy FINAL/Database_files/eukaryome-ref-tax.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.95 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/euk-95

# filter out unassigned sequences into their own file
qiime taxa filter-seqs \
--i-sequences FINAL/taxa/maarjam-80/unassigned-rep-seqs-80.qza \
--i-taxonomy FINAL/taxa/euk-95/classification.qza \
--p-include unassigned \
--o-filtered-sequences FINAL/taxa/euk-95/unassigned-rep-seqs-95.qza


## 4.2.2 90% ID ##
##################

qiime feature-classifier classify-consensus-blast \
--i-query FINAL/taxa/euk-95/unassigned-rep-seqs-95.qza \
--i-reference-reads FINAL/Database_files/eukaryome-ref-seqs.qza \
--i-reference-taxonomy FINAL/Database_files/eukaryome-ref-tax.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.90 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/euk-90

# filter out unassigned sequences into their own file.
qiime taxa filter-seqs \
--i-sequences FINAL/taxa/euk-95/unassigned-rep-seqs-95.qza \
--i-taxonomy FINAL/taxa/euk-90/classification.qza \
--p-include unassigned \
--o-filtered-sequences FINAL/taxa/euk-90/unassigned-rep-seqs-90.qza

## 4.2.3 80% ID ##
##################

qiime feature-classifier classify-consensus-blast \
--i-query FINAL/taxa/euk-90/unassigned-rep-seqs-90.qza \
--i-reference-reads FINAL/Database_files/eukaryome-ref-seqs.qza \
--i-reference-taxonomy FINAL/Database_files/eukaryome-ref-tax.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.80 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/euk-80

# filter out unassigned sequences into their own file
qiime taxa filter-seqs \
--i-sequences FINAL/taxa/euk-90/unassigned-rep-seqs-90.qza \
--i-taxonomy FINAL/taxa/euk-80/classification.qza \
--p-include unassigned \
--o-filtered-sequences FINAL/taxa/euk-80/unassigned-rep-seqs-80.qza

#run remaining unassignd sequences through the next databse!


## 4.3 NCBI ##
##############

## 4.3.1 95% ID ##
##################

qiime feature-classifier classify-consensus-blast \
--i-query FINAL/taxa/euk-80/unassigned-rep-seqs-80.qza \
--i-reference-reads FINAL/Database_files/ncbi-glom-refseqs-unfiltered.qza \
--i-reference-taxonomy FINAL/Database_files/ncbi-glom-taxonomy-unfiltered.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.80 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/ncbi-95

# filter out unassigned sequences into their own file
qiime taxa filter-seqs \
--i-sequences FINAL/taxa/euk-80/unassigned-rep-seqs-80.qza \
--i-taxonomy FINAL/taxa/ncbi-95/classification.qza \
--p-include unassigned \
--o-filtered-sequences FINAL/taxa/ncbi-95/unassigned-rep-seqs-95.qza


## 4.3.2 90% ID ##
##################

### NO HITS MOVED TO NCBI 80% IDENTITY ###

qiime feature-classifier classify-consensus-blast \
--i-query FINAL/taxa/ncbi-95/unassigned-rep-seqs-95.qza \
--i-reference-reads FINAL/Database_files/ncbi-glom-refseqs-unfiltered.qza \
--i-reference-taxonomy FINAL/Database_files/ncbi-glom-taxonomy-unfiltered.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.90 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/ncbi-90

# filter out unassigned sequences into their own file.
qiime taxa filter-seqs \
--i-sequences FINAL/taxa/ncbi-95/unassigned-rep-seqs-95.qza \
--i-taxonomy FINAL/taxa/ncbi-90/classification.qza \
--p-include unassigned \
--o-filtered-sequences FINAL/taxa/ncbi-90/unassigned-rep-seqs-90.qza

## 4.3.3 80% ID ##
##################

### NO HITS MOVED TO SILVA 95% IDENTITY ###

qiime feature-classifier classify-consensus-blast \
--i-query FINAL/taxa/ncbi-95/unassigned-rep-seqs-95.qza \
--i-reference-reads FINAL/Database_files/ncbi-glom-refseqs-unfiltered.qza \
--i-reference-taxonomy FINAL/Database_files/ncbi-glom-taxonomy-unfiltered.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.80 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/ncbi-80

# filter out unassigned sequences into their own file
qiime taxa filter-seqs \
--i-sequences FINAL/taxa/ncbi-95/unassigned-rep-seqs-95.qza \
--i-taxonomy FINAL/taxa/ncbi-80/classification.qza \
--p-include unassigned \
--o-filtered-sequences FINAL/taxa/ncbi-80/unassigned-rep-seqs-80.qza


## 4.4 SILVA 132 ##
###################

## 4.4.1 95% ID ##
##################

### NO HITS MOVED TO SILVA 90% IDENTITY ###

qiime feature-classifier classify-consensus-blast \
--i-query FINAL/taxa/ncbi-95/unassigned-rep-seqs-95.qza \
--i-reference-reads FINAL/Database_files/silva-ref-seqs.qza \
--i-reference-taxonomy FINAL/Database_files/silva-ref-tax.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.95 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/silva-95

# filter out unassigned sequences into their own file.
qiime taxa filter-seqs \
--i-sequences nanu-rep-seqs.qza \
--i-taxonomy taxa/silva-95/classification.qza \
--p-include unassigned \
--o-filtered-sequences taxa/silva-95/unassigned-rep-seqs-95.qza

## 4.4.2 90% ID ##
##################

qiime feature-classifier classify-consensus-blast \
--i-query FINAL/taxa/ncbi-95/unassigned-rep-seqs-95.qza \
--i-reference-reads FINAL/Database_files/silva-ref-seqs.qza \
--i-reference-taxonomy FINAL/Database_files/silva-ref-tax.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.90 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/silva-90

# filter out unassigned sequences into their own file.
qiime taxa filter-seqs \
--i-sequences FINAL/taxa/ncbi-95/unassigned-rep-seqs-95.qza \
--i-taxonomy FINAL/taxa/silva-90/classification.qza \
--p-include unassigned \
--o-filtered-sequences FINAL/taxa/silva-90/unassigned-rep-seqs-90.qza

## 4.2.3 80% ID ##
##################

### NO HITS ###

qiime feature-classifier classify-consensus-blast \
--i-query FINAL/taxa/silva-90/unassigned-rep-seqs-90.qza \
--i-reference-reads FINAL/Database_files/silva-ref-seqs.qza \
--i-reference-taxonomy FINAL/Database_files/silva-ref-tax.qza \
--p-maxaccepts 1 \
--p-perc-identity 0.80 \
--p-query-cov 0.90 \
--p-strand both \
--p-evalue 1e-50 \
--p-min-consensus 0.51 \
--output-dir FINAL/taxa/silva-80

# filter out unassigned sequences into their own file
qiime taxa filter-seqs \
--i-sequences FINAL/taxa/silva-90/unassigned-rep-seqs-90.qza \
--i-taxonomy FINAL/taxa/silva-80/classification.qza \
--p-include unassigned \
--o-filtered-sequences FINAL/taxa/silva-80/unassigned-rep-seqs-80.qza



####################################
	
### 5. FINALIZE TAXONOMY TABLES ###

####################################

## 5.1 Filter for Fungi ##
##########################

## 5.1.1 Maarjam 95% ID ##
##########################

# Filter for fungi only from maarjam 95% ID
qiime taxa filter-table \
--i-table FINAL/clean/nanu-table.qza \
--i-taxonomy FINAL/taxa/maarjam-95/classification.qza \
--p-include Fungi \
--o-filtered-table FINAL/taxa/maarjam-95/fungi-table.qza

# Filter for unnassigned features to be used for subsequent fungi filtering
#use output file for next table filtering step
qiime taxa filter-table \
--i-table FINAL/clean/nanu-table.qza \
--i-taxonomy FINAL/taxa/maarjam-95/classification.qza \
--p-include unassigned \
--o-filtered-table FINAL/taxa/maarjam-95/unassigned-table-95.qza


## 5.1.2 Maarjam 90% ID ##
##########################

# Filter out fungi
qiime taxa filter-table \
--i-table FINAL/taxa/maarjam-95/unassigned-table-95.qza \
--i-taxonomy FINAL/taxa/maarjam-90/classification.qza \
--p-include Fungi \
--o-filtered-table FINAL/taxa/maarjam-90/fungi-table.qza

# Filter out unnassigned 
qiime taxa filter-table \
--i-table FINAL/taxa/maarjam-95/unassigned-table-95.qza \
--i-taxonomy FINAL/taxa/maarjam-90/classification.qza \
--p-include unassigned \
--o-filtered-table FINAL/taxa/maarjam-90/unassigned-table-90.qza


## 5.1.3 Maarjam 80% ID ##
##########################

# Filter out fungi
qiime taxa filter-table \
--i-table FINAL/taxa/maarjam-90/unassigned-table-90.qza \
--i-taxonomy FINAL/taxa/maarjam-80/classification.qza \
--p-include Fungi \
--o-filtered-table FINAL/taxa/maarjam-80/fungi-table.qza

# Filter out unnassigned 
qiime taxa filter-table \
--i-table FINAL/taxa/maarjam-90/unassigned-table-90.qza \
--i-taxonomy FINAL/taxa/maarjam-80/classification.qza \
--p-include unassigned \
--o-filtered-table FINAL/taxa/maarjam-80/unassigned-table-80.qza


## 5.1.4 Eukaryome 95% ID ##
############################

# Filter out fungi
qiime taxa filter-table \
--i-table FINAL/taxa/maarjam-80/unassigned-table-80.qza \
--i-taxonomy FINAL/taxa/euk-95/classification.qza \
--p-include Fungi \
--o-filtered-table FINAL/taxa/euk-95/fungi-table.qza

# Filter out unnassigned 
qiime taxa filter-table \
--i-table FINAL/taxa/maarjam-80/unassigned-table-80.qza \
--i-taxonomy FINAL/taxa/euk-95/classification.qza \
--p-include unassigned \
--o-filtered-table FINAL/taxa/euk-95/unassigned-table-95.qza


## 5.1.5 Eukaryome 90% ID ##
############################

# Filter out fungi
qiime taxa filter-table \
--i-table FINAL/taxa/euk-95/unassigned-table-95.qza \
--i-taxonomy FINAL/taxa/euk-90/classification.qza \
--p-include Fungi \
--o-filtered-table FINAL/taxa/euk-90/fungi-table.qza

# Filter out unnassigned 
qiime taxa filter-table \
--i-table FINAL/taxa/euk-95/unassigned-table-95.qza \
--i-taxonomy FINAL/taxa/euk-90/classification.qza \
--p-include unassigned \
--o-filtered-table FINAL/taxa/euk-90/unassigned-table-90.qza


## 5.1.6 Eukaryome 80% ID ##
############################

# Filter out fungi
qiime taxa filter-table \
--i-table FINAL/taxa/euk-90/unassigned-table-90.qza \
--i-taxonomy FINAL/taxa/euk-80/classification.qza \
--p-include Fungi \
--o-filtered-table FINAL/taxa/euk-80/fungi-table.qza

# Filter out unnassigned 
qiime taxa filter-table \
--i-table FINAL/taxa/euk-90/unassigned-table-90.qza \
--i-taxonomy FINAL/taxa/euk-80/classification.qza \
--p-include unassigned \
--o-filtered-table FINAL/taxa/euk-80/unassigned-table-80.qza


## 5.1.7 NCBI 95% ID ##
#######################

# Filter out fungi
qiime taxa filter-table \
--i-table FINAL/taxa/euk-80/unassigned-table-80.qza \
--i-taxonomy FINAL/taxa/ncbi-95/classification.qza \
--p-include Fungi \
--o-filtered-table FINAL/taxa/ncbi-95/fungi-table.qza

# Filter out unnassigned 
qiime taxa filter-table \
--i-table FINAL/taxa/euk-80/unassigned-table-80.qza \
--i-taxonomy FINAL/taxa/ncbi-95/classification.qza \
--p-include unassigned \
--o-filtered-table FINAL/taxa/ncbi-95/unassigned-table-95.qza


## 5.1.8 SILVA 90% ID ##
########################

# Filter out fungi
qiime taxa filter-table \
--i-table FINAL/taxa/ncbi-95/unassigned-table-95.qza \
--i-taxonomy FINAL/taxa/silva-90/classification.qza \
--p-include Fungi \
--o-filtered-table FINAL/taxa/silva-90/fungi-table.qza

## Plugin error from taxa: All features were filtered, resulting in an empty table.

# Filter out unnassigned 
qiime taxa filter-table \
--i-table FINAL/taxa/ncbi-95/unassigned-table-95.qza \
--i-taxonomy FINAL/taxa/silva-90/classification.qza \
--p-include unassigned \
--o-filtered-table FINAL/taxa/silva-90/unassigned-table-90.qza



## 5.2 Merge Tables ##
############################

## 5.2.1 Fungi Only Tables ##
#############################

# merge the fungi only taxonomy classification files.
qiime feature-table merge \
--i-tables FINAL/taxa/maarjam-95/fungi-table.qza \
--i-tables FINAL/taxa/maarjam-90/fungi-table.qza \
--i-tables FINAL/taxa/maarjam-80/fungi-table.qza \
--i-tables FINAL/taxa/euk-95/fungi-table.qza \
--i-tables FINAL/taxa/euk-90/fungi-table.qza \
--i-tables FINAL/taxa/euk-80/fungi-table.qza \
--i-tables FINAL/taxa/ncbi-95/fungi-table.qza \
--o-merged-table FINAL/nanu-fungi-table.qza \
--p-overlap-method sum

qiime feature-table summarize \
--i-table FINAL/nanu-fungi-table.qza \
--o-visualization FINAL/nanu-fungi-table.qzv \
--m-sample-metadata-file FINAL/nanu-metadata.tsv

qiime metadata tabulate \
--m-input-file FINAL/taxa/nanu-fungi-table.qza \
--o-visualization FINAL/taxa/nanu-fungi-table_2.qzv

## 5.2.2 Unassigned & Fungi Tables ##
#####################################

qiime feature-table merge \
--i-tables FINAL/nanu-fungi-table.qza \
--i-tables FINAL/taxa/silva-90/unassigned-table-90.qza \
--o-merged-table FINAL/nanu-all-table.qza \
--p-overlap-method sum

qiime feature-table summarize \
--i-table FINAL/nanu-all-table.qza \
--o-visualization FINAL/nanu-all-table.qzv \
--m-sample-metadata-file FINAL/nanu-metadata.tsv


## 5.2.3 Taxonomy Classification files ##
#########################################

## merge taxa table only merges files IF the lower % ID file is listed first! ##
qiime feature-table merge-taxa \
--i-data FINAL/taxa/maarjam-90/classification-m90.qza \
--i-data FINAL/taxa/maarjam-95/classification-m95.qza \
--o-merged-data FINAL/taxa/m95-m90-classification.qza

qiime metadata tabulate \
--m-input-file FINAL/taxa/maarjam-95/classification-m95.qza \
--o-visualization FINAL/taxa/maarjam-95/classification-m95.qzv
# 867

qiime metadata tabulate \
--m-input-file FINAL/taxa/maarjam-90/classification-m90.qza \
--o-visualization FINAL/taxa/maarjam-90/classification-m90.qzv
# 142

qiime metadata tabulate \
--m-input-file FINAL/taxa/m95-m90-classification.qza \
--o-visualization FINAL/taxa/m95-m90-classification.qzv
# 1,009


qiime feature-table merge-taxa \
--i-data FINAL/taxa/maarjam-80/classification.qza \
--i-data FINAL/taxa/m95-m90-classification.qza \
--o-merged-data FINAL/taxa/maarjam-merged-classification.qza

qiime metadata tabulate \
--m-input-file FINAL/taxa/maarjam-80/classification.qza \
--o-visualization FINAL/taxa/maarjam-80/classification-m80.qzv
# 294

qiime metadata tabulate \
--m-input-file FINAL/taxa/maarjam-merged-classification.qza \
--o-visualization FINAL/taxa/maarjam-merged-classification.qzv
# 1,303


qiime feature-table merge-taxa \
--i-data FINAL/taxa/euk-90/classification.qza \
--i-data FINAL/taxa/euk-95/classification.qza \
--o-merged-data FINAL/taxa/e95-e90-classification.qza

qiime metadata tabulate \
--m-input-file FINAL/taxa/euk-95/classification.qza \
--o-visualization FINAL/taxa/euk-95/classification-e95.qzv
# 150

qiime metadata tabulate \
--m-input-file FINAL/taxa/euk-90/classification.qza \
--o-visualization FINAL/taxa/euk-90/classification-e90.qzv
# 63

qiime metadata tabulate \
--m-input-file FINAL/taxa/e95-e90-classification.qza \
--o-visualization FINAL/taxa/e95-e90-classification.qzv
# 213


qiime feature-table merge-taxa \
--i-data FINAL/taxa/euk-80/classification.qza \
--i-data FINAL/taxa/e95-e90-classification.qza \
--o-merged-data FINAL/taxa/euk-merged-classification.qza

qiime metadata tabulate \
--m-input-file FINAL/taxa/euk-80/classification.qza \
--o-visualization FINAL/taxa/euk-80/classification-e80.qzv
# 23

qiime metadata tabulate \
--m-input-file FINAL/taxa/euk-merged-classification.qza \
--o-visualization FINAL/taxa/euk-merged-classification.qzv
# 


qiime feature-table merge-taxa \
--i-data FINAL/taxa/ncbi-95/classification.qza \
--i-data FINAL/taxa/silva-90/classification.qza \
--o-merged-data FINAL/taxa/remaining-merged-classification.qza

qiime metadata tabulate \
--m-input-file FINAL/taxa/ncbi-95/classification.qza \
--o-visualization FINAL/taxa/ncbi-95/classification.qzv
# 2

qiime metadata tabulate \
--m-input-file FINAL/taxa/silva-90/classification.qza \
--o-visualization FINAL/taxa/silva-90/classification.qzv
# 0

qiime metadata tabulate \
--m-input-file FINAL/taxa/remaining-merged-classification.qza \
--o-visualization FINAL/taxa/remaining-merged-classification.qzv
# 2


qiime feature-table merge-taxa \
--i-data FINAL/taxa/euk-merged-classification.qza \
--i-data FINAL/taxa/maarjam-merged-classification.qza \
--o-merged-data FINAL/taxa/euk-maarjam-merged-classification.qza

qiime metadata tabulate \
--m-input-file FINAL/taxa/euk-maarjam-merged-classification.qza \
--o-visualization FINAL/taxa/euk-maarjam-merged-classification.qzv
# 1,539


qiime feature-table merge-taxa \
--i-data FINAL/taxa/remaining-merged-classification.qza \
--i-data FINAL/taxa/euk-maarjam-merged-classification.qza \
--o-merged-data FINAL/taxa/final-merged-classification.qza

qiime metadata tabulate \
--m-input-file FINAL/taxa/final-merged-classification.qza \
--o-visualization FINAL/taxa/final-merged-classification.qzv
#


## 5.3 Remove Controls ##
#########################

# filter out negative samples
qiime feature-table filter-samples \
--i-table FINAL/nanu-fungi-table.qza \
--m-metadata-file FINAL/nanu-metadata.tsv \
--p-where "SampleType IN ('Field', 'Trap-Sorgum', 'Trap-Bahia', 'Slurry')" \
--o-filtered-table FINAL/no-negs-nanu-fungi-table.qza

# filter the representative sequences to have no negatives using the no-negs table.
qiime feature-table filter-seqs \
--i-data FINAL/clean/nanu-rep-seqs.qza \
--i-table FINAL/no-negs-nanu-fungi-table.qza \
--o-filtered-data FINAL/no-negs-nanu-rep-seqs.qza

qiime feature-table summarize \
--i-table FINAL/no-negs-nanu-fungi-table.qza \
--o-visualization FINAL/no-negs-nanu-table.qzv 



################################
	
### 6. PRELIMINARY ANALYSES ###

################################

## 6.1 Phylogeny ##
###################

# create a phylogeny tree
qiime phylogeny align-to-tree-mafft-fasttree \
--i-sequences FINAL/no_negs/no-negs-nanu-rep-seqs.qza \
--o-alignment FINAL/phylogeny/aligned-rep-seqs.qza \
--o-masked-alignment FINAL/phylogeny/masked-aligned-rep-seqs.qza \
--o-tree FINAL/phylogeny/unrooted-tree.qza \
--o-rooted-tree FINAL/phylogeny/rooted-tree.qza



############################
	
### 7. EXPORT FILES TO R ###

############################

## 7.1 Feature Table ##
#######################

# export feature table and rep seqs (Qiime2 table.qza = feature table)
qiime tools export \
  --input-path FINAL/no_negs/no-negs-nanu-fungi-table.qza \
  --output-path FINAL/no_negs/exported-feature-table

# convert feature table to TSV format
biom convert \
  --input-fp FINAL/no_negs/exported-feature-table/feature-table.biom \
  --output-fp FINAL/no_negs/exported-feature-table/feature-table.tsv \
  --to-tsv

# convert the tsv to a csv just in general command line 
sed 's/\t/,/g' FINAL/no_negs/exported-feature-table/feature-table.tsv > FINAL/no_negs/exported-feature-table/feature-table.csv


## 7.2 Rep Seqs ##
##################

# export the rep seqs
qiime tools export \
--input-path FINAL/no-negs-nanu-rep-seqs.qza \
--output-path FINAL/no_negs/exported-rep-seqs


## 7.3 Taxonomy Table ##
########################

# export taxonomy
qiime tools export \
  --input-path FINAL/taxa/final-merged-classification.qza \
  --output-path FINAL/exported-taxonomy
  
sed 's/\t/,/g' FINAL/exported-taxonomy/taxonomy.tsv > FINAL/exported-taxonomy/taxonomy.csv


 
## 7.4 Metadata Table ##
########################

sed 's/\t/,/g' FINAL/nanu_metadata_R.tsv > FINAL/nanu_metadata_R.csv













  


