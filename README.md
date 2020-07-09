# More-cancer-mutation-analysis
Analysis of mutations in new cohorts after TCGA, such as ICGC, CPTAC and GENIE, using the tools developed in the NeST paper

TODO: once this document is getting long, move to wiki

# MutSigCV 1.4

With the modified output including the "probability of mutating gene X in patient Y"

Usage (here breaking lines to show some clarity, don't break when actually run them):   
```
/cellar/local/matlab_linux64_R2016b/bin/matlab
 -nodisplay -nojvm -r 
 "MutSigCV('blca.maf',
           'exome_full192_coverage.txt',
           'gene.covariates.txt',
           'output', 
           'mutation_type_dictionary_file.txt',
           'chr_files_hg38');
quit"
```
The input is a MAF file (e.g. `blca.maf`), which calls MutSigCV from MATLAB. `output` above specify an output directory. Other files are really big so there are not here but on the lab server. In the output directory there will be a file ending with "p1gp", which is the per-patient probability we are looking for.

### Original website
https://software.broadinstitute.org/cancer/cga/mutsig


### the links of other files here

`/cellar/users/f6zheng/work_2020/nest_confirmatory/n4wilson/mutsigcv_files` (contains coverage, covariates and dictionary)
`/cellar/users/f6zheng/Data/Public_data/chr_files_hg38`

However, needs to be careful to find out whether the new studies uses hg19, it is possible that they use hg38

confirmed that the CPTAC cohort use GRch38 (hg38);

Download from 
Updates: need to convert the FASTA file downloaded from `https://hgdownload.soe.ucsc.edu/goldenPath/hg38/chromosomes/` to txt (remove header and newlines) before they can be used by MutSigCV.


# Obtaining the MAF files

1. Go to [GDC commons](https://portal.gdc.cancer.gov/exploration)
2. Select CPTAC studies; select WXS (whole-exome sequencing); download MAF files (via `gdc-client`)

<p align="center">
  <img src="readme_figs/f1.png" width="600" align="center">
</p>

3. Concatenate MAF files. In theory, the MAF file should be able to be used as the input of the matlab code. 

# Update Jul. 2

## use p1gp file to calculate expectation
is an M x N table; M patients are ordered as in `$type.patients.txt`; N genes are ordered as in `all_genes.txt`. 
need to sum values over patients to get a value for each gene.


## use MAF file to calculate observe
use `Variant_Classification` column, keep the rows with these values `['Missense_Mutation', 'Nonsense_Mutation', 'Frame_Shift_Del', 'Frame_Shift_Ins', 'Splice_Site', 'Splice_Region', 'In_Frame_Del', 'In_Frame_Ins', 'Nonstop_Mutation']`

## goal is to create two files like the following

`/cellar/users/f6zheng/Data/human_cancer_hierarchy_submit/cancerdata/analyzed/mutsigcv_pancanatlas/mutsigcv1.4_pancanceratlas_observe.csv`

`/cellar/users/f6zheng/Data/human_cancer_hierarchy_submit/cancerdata/analyzed/mutsigcv_pancanatlas/mutsigcv1.4_pancanceratlas_expect.csv`

# Update Jul.9

## Run HiSig

See the repo of [HiSig](https://github.com/fanzheng10/HiSig) here. 

Next step is running HiSig. Now HiSig does not require DDOT installation. Also by default it fits a Poisson (log-linear) regression instead of linear regression, so no need to worry about logarithms. For each gene, calculate `observe-expect`, and used as the input of HiSig (the `--sig` parameter for `prepare_input.py`).

Also need a file from the NeST project, for the `--ont` parameter. Use this file:

`/cellar/users/f6zheng/work_2020/nest_confirmatory/n4wilson/clixo_ccmi_20190527-164246_min4.ont`

As a sanity check, in the meantime also use the `sample.ont` file provided in the HiSig repo. It is based on the cellular components annotations from the Gene Ontology.

Estimate time: 3-7 hrs (seems to be longer after using Poisson?)

Need special cares when submititng jobs to cluster. use `mem` and `cpu` parameters in the `Cluster.sbatch` function, set both to 8. Confirm the bash script to submit contains the following in the header:  

#SBATCH --mem-per-cpu=8G  
#SBATCH -c 8





