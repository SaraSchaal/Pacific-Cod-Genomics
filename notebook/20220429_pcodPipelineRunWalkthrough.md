## lcWGS of Pacific Cod Walkthrough
I have run the Pacific cod data through the pipeline using the Atlantic cod reference genome and the new Pacific cod reference genome. The Atlantic cod run has all associated files named pcodGMAC, whereas the run using the pacific cod reference is called pcodREF. pcodGMAC had 944 samples 

https://github.com/letimm/WGSfqs-to-genolikelihoods

# Step 0 - Configure  

i) We first need move data of interest from sednagold to sedna.  

NOTE TO SELF, remember to add the hmac when logging into sednagold:  
```
 ssh -m hmac-sha2-512 sara.schaal@161.55.97.202
```

To move files from sednagold to sedna, you must be logged into sedna and run scp from there. I put all files that I am interested in analyzing into a single folder and just move all .fq.gz files from that folder on sednagold to the destination folder on sedna using this line of code:  
```
 scp -P 22 -o MACs=hmac-sha2-512 sara.schaal@161.55.97.202:/sednagold/Gadus_macrocephalus/novaseq/*.fq.gz /home/sschaal/pcod/20220429/novaseq/
```

For my sanity, I check to make sure the file sizes on sedna match the ones on sednagold (overkill but I like to be sure in case something funky happened on the transfer).  

```
ls -la | awk '{ print $5 }' > pcodREF_filesizesSEDNAGOLD.txt
ls -la | awk '{ print $5 }' > pcodREF_filesizesSEDNA.txt
```

I check in R that they are the same vectors.  

ii) Next we need to make the config file and organize everything needed for the run through Laura's pipeline.  

This pipeline will run with any file name so it is not necessary to rename files. However, we do need to get all file names into a .txt file for the configuration step. This can be done by moving to the appropriate directory where your files are and running this line of code:  

```
ls *.fq.gz | while read file; do newfile=`echo /home/sschaal/pcod/20220125/novaseq/$file`; echo $newfile >> pcod_fastqs.txt; done
```

iii) run the first pipeline script

```
lcWGSpipeline_step0-configure_vX.Y.py -c {lcWGS_config.txt}
```

if the config step works you should see this message:
```
Step 0 has finished successfully! You will find two new scripts in ./scripts/: /home/sschaal/pcod/20220429/novaseq/scripts/PGA_assembly_bwa-indexSLURM.sh and /home/sschaal/pcod/20220429/novaseq/scripts/PGA_assembly_faiSLURM.sh.
Both scripts can run simultaneously with 'sbatch'.
 However, there is no need to wait for these jobs to finish to move to step 1.
To continue on, call step 1 to generate scripts for fastQC and multiQC.
Remember to pass the newly-made checkpoint (.ckpt) file with the '-p' flag.

```

# Step 1

Next we do the QC steps. 

i) run the second pipeline script:
```
 lcWGSpipeline_step1-qc_20220125.py -p pcod20220713.ckpt
```

```
Step 1 has finished successfully! You will find two new scripts in ./scripts/: lc/home/sschaal/pcod/20220429/novaseq/scripts/pcodREF-raw_fastqcARRAY.sh and /home/sschaal/pcod/20220429/novaseq/scripts/pcodREF-raw_multiqcSLURM.sh.
/home/sschaal/pcod/20220429/novaseq/scripts/pcodREF-raw_fastqcARRAY.sh must run prior to launching /home/sschaal/pcod/20220429/novaseq/scripts/pcodREF-raw_multiqcSLURM.sh.
 However, there is no need to wait for these jobs to finish to move to step 2.
To continue on, call step 2 to generate a script for TRIMMOMATIC.
Remember to pass the checkpoint (.ckpt) file with the '-p' flag.
```

For the multiqc step there is one error still in the production of the multiQC.sh file. The path to QC is still linked to lauras. Need to manually change to: ```source /home/sschaal/qc_env/bin/activate```.  

# Step 2

```
Step 2 has finished successfully! You will find three new scripts in ./scripts/: /home/sschaal/pcod/20220429/novaseq/scripts/pcodREF_trimARRAY.sh, /home/sschaal/pcod/20220429/novaseq/scripts/pcodREF-trim_fastqcARRAY.sh, and /home/sschaal/pcod/20220429/novaseq/scripts/pcodREF-trim_multiqcSLURM.sh.
Each script must run in the above order and each job must finish before submitting the next.
While there is no need to wait before running step 3 to generate the alignment script, it is probably wise to wait until the multiQC script has completed and the results have been viewed in a web browser prior to submitting the script written by step 3.
```


```
Step 3 has finished successfully! You will find two new scripts in ./scripts/: /home/sschaal/pcod/20220429/novaseq/scripts/pcodREF_alignARRAY.sh and /home/sschaal/pcod/20220429/novaseq/scripts/pcodREF_depthsARRAY.sh.
These scripts must be run in the order given above and both must finish before moving on to step 4.
After /home/sschaal/pcod/20220429/novaseq/scripts/pcodREF_depthsARRAY.sh has run, download the resulting file: pcodREF_depths.csv and generate a barchart of mean depths by individual. This will help you determine whether any individuals fall substantially below the average depth (usually, we use a cutoff of 1x).
If you identify samples with coverage that is 'too low', add the sample id(s) to a new file, referred to as the 'blacklist' of individuals to be excluded from genotype likelhiood calculations and the final data sets.
After generating this blacklist, you can continue to step 4 to write scripts for generating the final data sets.
Remember to pass the checkpoint (.ckpt) file with the '-p' flag AND the blacklsit file with the '-b' flag.

```

```
Step 4 has finished successfully! You will find two new scripts in ./scripts/:
/home/sschaal/pcod/20220429/novaseq/scripts/pcodREF_globalARRAY.sh calculates genotype likelihoods across all sites on each chromosome (separately).
/home/sschaal/pcod/20220429/novaseq/scripts/pcodREF_polymorphicARRAY.sh calculates genotype likelihoods across all polymorphic sites on each chromosome (separately).

Both scripts can run simultaneously.
After they have run, you will have genotype likelihoods (gls) and allele frequencies (maf) for all sites in the genome (global) and putatively variable sites (polymorphic).
Congratulations! This represents the end of data assembly.

```

### Needed updates

1) add the python script: mean_cov_ind.py
2) add multiqc instructions and instructions for updating the script that the pipeline generates 
    - Need to manually change your multiqc path in the multiQC.sh file to: ```source /home/sschaal/qc_env/bin/activate```. 
3) genomewide data for fst
4) add pcangsd notes for understanding the covariance matrix output is in the order of the bams list that went into the program 
