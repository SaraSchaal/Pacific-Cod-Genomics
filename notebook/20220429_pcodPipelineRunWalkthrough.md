## lcWGS of Pacific Cod Walkthrough
I have run the Pacific cod data through the pipeline using the Atlantic cod reference genome and the new Pacific cod reference genome. The Atlantic cod run has all associated files named pcodGMAC, whereas the run using the pacific cod reference is called pcodREF.   

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

ii) Next we need to make the config file and organize everything needed for the run through Laura's pipeline.  

This pipeline will run with any file name so it is not necessary to rename files. However, we do need to get all file names into a .txt file for the configuration step. This can be done by moving to the appropriate directory where your files are and running this line of code:  

```
ls | while read file; do newfile=`echo /home/sschaal/pcod/20220429/novaseq/$file`; echo $newfile > pcodREF_fastqs.txt; done
```

