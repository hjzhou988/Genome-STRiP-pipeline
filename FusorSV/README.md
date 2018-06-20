# FusorSV
## Overview
FusorSV [1] is algorithm that trains a "fusion" model with 8 SV detection algorithms' outputs and a truth set. The fusion model is then used for detecting SVs in new samples. Since we do not have a truth set, we are going to use their fusion model for SV detection. 
FusorSV is part of Structural Variants Engine, which installs all the algorithms and aligners and does the aligning and SV detection for you. Due to the fact that hoffman2 is not able to install Docker, which SVE highly depends on, I focused on FusorSV only.  We might need to install each algorithm by ourselves. 

I was testing FusorSV using the author's own git hub page https://github.com/timothyjamesbecker/FusorSV , which is different from the official FusorSV https://github.com/TheJacksonLaboratory/SVE . The official one seems harder to install, and the author already corrected some bugs and made it usable for many samples. However, he has not solved the memory issue if we want to run on hundreds of samples yet. 

## Install
* Go to https://github.com/timothyjamesbecker/FusorSV and install according to the "pip" install instructions. 
Follow their "pip" install instructions. Use "--user" option for each install so that it's installed in your local directory. The FusorSV.py file will be in the ~/.local/bin/ directory. bx-python was hard to install. Since it was only used for "liftover", it's optional, and the 0.7.3  (even though it's not the required <0.7.3) version was already installed in python 2.7.2 (hoffman2).

* I also installed python 2.7.10 in my home directory, as the required version of python (2.7.6 < python < 2.7.12) is not available on hoffman2, but 2.7.2 from hoffman2 is fine. 

* FusorSV requires the HG37 version of reference genome (not HG19), as using HG19 will cause this error:
```
Traceback (most recent call last):
  File "/u/home/h/hjzhou/.local/bin/FusorSV.py", line 529, in <module>
      overlap=cluster_overlap))
  File "/u/home/h/hjzhou/.local/lib/python2.7/site-packages/fusorsv/svu_utils.py", line 475, in fusorSV_vcf_multi_sample_merge
       with open(vcfs[0],'r') as f:
  IndexError: list index out of range
```
You also need to copy the HG37 genome into your own folder, as FusorSV will index it, if it was not indexed yet. 


## Prepare inputs from different algorithms
Currently, we have the outputs from CNVnator, Genome STRiP, and LUMPY. The author provided script for converting multi-sample vcf from Genome STRiP to single-sample vcf with the VCFs from both SVGenotyper and CNVDiscovery. I tweaked the script to accept the VCF from SVGenotyper only. 
```
python gs_split_merge.HZ.py -d GenomeStrip.vcf -o GSsplit/
```
CNVnator also has its own perl code to convert their format of output (.call.txt) to VCF format. 
```
# in the folder of CNVnator output
for file in `ls *.calls.txt`
do
name=`echo $file | sed 's/.calls.txt//g'`
perl ~/CNVnator/cnvnator2VCF.pl $file > $SCRATCH/mysamples/${name}/${name}_S10.vcf
done
```
Both scripts are included in the scripts folder

## Run FusorSV
* For the samples that was installed with the software, the peak memory is 46G (whether or not whole genome or just chromosome 22). 
* FusorSV appears to be able to resume failed job. Because if I specify the same output folder, the job uses less time and peak memory for finishing the job. 
* If bx-python was not installed, it will have this error:
```
Traceback (most recent call last):
  File "/u/home/h/hjzhou/.local/bin/FusorSV.py", line 532, in <module>
    su.fusorSV_vcf_liftover_samples(out_dir+'/vcf/all_samples_genotypes*.vcf*',ref_path,lift_over) #default is on
  File "/u/home/h/hjzhou/.local/lib/python2.7/site-packages/fusorsv/svu_utils.py", line 902, in fusorSV_vcf_liftover_samples
    fusorSV_vcf_liftover(vcf,ref_path,chain_path)
  File "/u/home/h/hjzhou/.local/lib/python2.7/site-packages/fusorsv/svu_utils.py", line 894, in fusorSV_vcf_liftover
    import crossmap as cs
  File "/u/home/h/hjzhou/.local/lib/python2.7/site-packages/fusorsv/crossmap.py", line 16, in <module>
    from bx.bbi.bigwig_file import BigWigFile
ImportError: No module named bx.bbi.bigwig_file

```
At this time point your can use hoffman2 python 2.7.2 to resume the job. 


[1] Becker, Timothy, et al. "FusorSV: an algorithm for optimally combining data from multiple structural variation detection methods." Genome biology 19.1 (2018): 38.
