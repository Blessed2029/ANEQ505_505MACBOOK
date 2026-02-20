**ANEQ Homework #1: Practice Importing Data, Demultiplexing Reads, and Denoising**

**Due Feb 19th at midnight**

**Name:**Esther Alorkpa

**For complete credit for this assignment, you must answer all questions and include all commands in your obsidian upload.**

------------------------------------------------------------------------
**Learning Objectives**

1. Practice independently running the first few steps of a Qiime2 workflow We will focus on importing data, demultiplexing, and denoising for homework 1.

2. Practice recording commands and editing code to match your analysis

3. Push to GitHub for credit

------------------------------------------------------------------------


**Cow Site Dataset**

20 adult dairy cattle were sampled (via swabbing) to determine the microbial community composition between 5 different body sites (nasal/skin/udder/fecal/oral).  We want you to determine if and how the microbial composition changes between sites over the course of multiple homework assignments.

We will first begin by copying raw sequencing data from a public folder on Alpine into a new directory (that you create) on your Alpine account.

**Steps:**

1. Log into Alpine using OnDemand and create a new directory for this new analysis in your _scratch_ directory. Hint: it should look something like: /scratch/alpine/$USER/cow/
2. Move into your new directory using OnDemand

 Using the OnDemand file browser, I navigated into my project folder:
`/scratch/alpine/c837277701@colostate.edu/cow/`

3. Create the following sub-directories using OnDemand: 
	1. slurm
	2. taxonomy
	3. tree
	4. taxaplots
	5. dada2
	6. demux
	7. metadata
	8. core_metrics
Inside `/scratch/alpine/c837277701@colostate.edu/cow/`, I created:
- slurm
- taxonomy
- tree
- taxaplots
- dada2
- demux
- metadata
- core_metrics

4. Download the cow_barcodes.txt and cow_metadata.txt files from Canvas and upload them both to your metadata folder within your new cow directory. So, your filepath should look something like this: /scratch/alpine/$USER/cow/metadata. 

ANSWER: I downloaded `cow_barcodes.txt` and `cow_metadata.txt` from Canvas and uploaded them to:
`/scratch/alpine/c837277701@colostate.edu/cow/metadata/`

5. On OnDemand, go to your cow directory and open a new terminal

6. Copy the raw sequencing files from this public folder to **your new folder** using the terminal. Do **not** change the names of these files and folders. Hint: make sure you are in your new cow folder before you run this code (this will copy over the whole folder):  I Opened an OnDemand terminal in my cow directory.

```
cp -r /pl/active/courses/2024_summer/maw_2024/raw_reads .
```

This created a `raw_reads/` folder in:  
`/scratch/alpine/c837277701@colostate.edu/cow/`

7.    Launch an interactive session and load qiime2 within your cow directory. 

```
#launch an interactive session: 
ainteractive --ntasks=6 --time=02:00:00

#insert your code here to activate qiime. Hint: there should be 2 things you add here

module purge
module load qiime2/2024.10_amplicon


```


8.    Import the sequences/reads into a Qiime2-readable format (.qza). Note this might take 10-20 mins

```
qiime tools import \
--type EMPPairedEndSequences \
--input-path raw_reads \
--output-path cow_reads.qza
```


9.    Demultiplex the reads by submitting a job. Note this may take ~30 mins

a.    Go into your slurm directory using OnDemand. Create a new file named **demux.sh** so you can submit a job that will demultiplex your sequences quicker. Fill in the lines that need editing (denoted by capital letters or hashes) to this demultiplexing command and add that to your new script. 


```
#!/bin/bash  
#SBATCH --job-name=demux  
#SBATCH --nodes=1  
#SBATCH --ntasks=12  
#SBATCH --partition=amilan  
#SBATCH --time=02:00:00  
#SBATCH --mail-type=ALL  
#SBATCH --output=slurm-%j.out  
#SBATCH --qos=normal  
#SBATCH --mail-user=c837277701@colostate.edu


#What needs to go here in order to “turn on” qiime2? Hint: we do these 2 commands every time we activate qiime2!                        

module purge
module load qiime2/2024.10_amplicon

#change the following line if your file path looks different
cd /scratch/alpine/$USER/cow/demux

#Below is the command you will run to demultiplex the samples.

qiime demux emp-paired \
--m-barcodes-file ../metadata/cow_barcodes.txt \
--m-barcodes-column barcode \
--p-rev-comp-mapping-barcodes \
--p-rev-comp-barcodes \
--i-seqs ../cow_reads.qza \
--o-per-sample-sequences demux_cow.qza \
--o-error-correction-details cow_demux_error.qza

#visualize the read quality
qiime demux summarize \
--i-data demux_cow.qza \
--o-visualization demux_cow.qzv
```


 Run the script in your slurm directory as a job using: 
 ```
 sbatch name of your script.sh
 ```


10.    Denoise. 

Fill in the blank to denoise your samples based on what you think should be trimmed (from the front of the reads) or truncated (from the ends of the reads) based on the demux_cow.qzv file. You can run this in the terminal or as a job.

```
cd ADD PATH TO DADA2 DIRECTORY

qiime dada2 denoise-paired \  
--i-demultiplexed-seqs ../demux/demux_cow.qza \  
--p-trim-left-f 0 \  
--p-trim-left-r 0 \  
--p-trunc-len-f 150 \  
--p-trunc-len-r 150 \  
--p-n-threads 6 \  
--o-representative-sequences cow_seqs_dada2.qza \  
--o-denoising-stats cow_dada2_stats.qza \  
--o-table cow_table_dada2.qza

#Visualize the denoising results:
qiime metadata tabulate \  
--m-input-file cow_dada2_stats.qza \  
--o-visualization cow_dada2_stats.qzv

qiime feature-table summarize \  
--i-table cow_table_dada2.qza \  
--m-sample-metadata-file ../metadata/cow_metadata.txt \  
--o-visualization cow_table_dada2.qzv

qiime feature-table tabulate-seqs \  
--i-data cow_seqs_dada2.qza \  
--o-visualization cow_seqs_dada2.qzv
```

	
Briefly **describe** the key information from each denoising output file:
1. Representative Sequences
- This file contains the **unique ASV sequences** produced by DADA2 (the “representative” DNA sequence for each feature/ASV).
- **4,657 features (ASVs)**.
- Sequence length summary:
    
    - **Min length:** 240 nt
        
    - **Max length:** 427 nt
        
    - **Mean length:** 253.06 nt
        
    - Most sequences are tightly clustered around **~253 nt** (the percentile table shows ~252–254 for most).
2. Denoising Stats
- This file reports, **per sample**, how many reads survive each DADA2 step:
    - `input` → `filtered` → `denoised` → `merged` → `non-chimeric`
- The most important “final yield” column is **percentage of input non-chimeric** (how many reads remain after all steps).

3. Denoised Table
- This is the **feature table**: counts of each ASV in each sample after denoising + chimera removal.
- Overall table summary:
    
    - **Number of samples:** 147
        
    - **Number of features:** 4,657
        
    - **Total frequency:** 1,691,073 reads (total counts in the final table)
        
    - Per-sample read counts include:
        
        - **Mean frequency:** 11,503.9
            
        - **Median frequency:** 9,670
            
        - **Max frequency:** 34,453
            
        - **Min frequency:** 0

**Answer the following questions:**  
1. What is the mean reads per sample? **Mean frequency (mean reads/sample) = 11,503.9** (from the table summary).
2. How long are the reads? Reads are **~253 bp on average**.   **Mean length = 253.06 nt** . Most sequences are tightly centered around ~253 (percentile summary shows ~252–254 for most).
3. What is the maximum length of all your sequences? **Maximum sequence length = 427 bp**.
4. Which sample (not including extraction controls starting with EC) lost the highest % of reads? 
   **`2019.3.14.cow.oral.20`**
- **% input non-chimeric = 8.92%**
- So it **lost 91.08%** of reads (100 − 8.92).
(I sorted table and it shows several EC* controls with lower values, but the first **non-EC** row with the lowest retention is **cow.oral.20**)
5. Why did you chose to trim or truncate where you did?
I truncated both forward and reverse reads at **150 bp** (with **trim-left = 0**) because the per-base quality scores start to deteriorate toward the read tails, and truncating before the low-quality region improves denoising accuracy. Even with truncation, **150 + 150 = 300 bp** still leaves sufficient overlap to merge reads for an amplicon of ~**253 bp** (≈ **47 bp** overlap), so merging remains reliable while reducing error from poor-quality tails.

**To submit your homework from this document:**
write all of your commands here, then use command+P (for mac) or control+P (for windows) and search Git: commit. click it. then search for Git: Push and click it. go to your github online to check that it pushed correctly. we will check your github for homework credit. 