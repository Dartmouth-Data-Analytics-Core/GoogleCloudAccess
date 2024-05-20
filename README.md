# GoogleCloudAccess
Instructions for creating a Dartmouth affiliated account on Google Cloud. 


## 1. Set up a Google Cloud account
To access cloud resources you will need to create a google cloud account with an *email addres*s and a *credit card*. 

Creating the account is free, compute resources and data storage you pay for what you use. With a grant one way to set this up is to link a p-card or corporate card to the google account and use the string to pay the monthly usage fees on the p-card. However, if you have an NIH grant you can receive up to a 20% discount on GCP storage or data processing costs. 

Cloud accounts created with Dartmouth College affiliated email address (i.e. Fred.Rogers[]()@dartmouth.edu) will be managed by [Research Computing](Research.Computing@dartmouth.edu). You should be able to create a project and billing will automatically be managed by Burwood. You should reach out to research computing with the details of the grant that supports the work you're doing so that they can provide those details to Burwood. You're grant will be charged on a monthly basis based on the storage and compute resources used in the project linked to your grant. 

You can follow the instructions on the [Google Cloud home page](cloud.google.com/free) to create an account. 

## 2. Set up Google Cloud project and link to a billing account

Currently we do not have the ability to make a new project in GDSC, to make a project send the title of the project to research computing and they will set up the project for you, even better you can also send them the grant number to charge.


## 4. Configure VM to use snakemake workflow 
  **All of the following steps will be captured in a single BASH script Tim is building**
```
# install conda on the VM
wget https://github.com/conda-forge/miniforge/releases/download/23.3.1-1/Mambaforge-23.3.1-1-Linux-x86_64.sh
chmod +x Mambaforge-23.3.1-1-Linux-x86_64.sh
./Mambaforge-23.3.1-1-Linux-x86_64.sh
exec bash

# create a conda environment that contains snakemake
mamba create -c conda-forge -c bioconda -n snakemake snakemake
mamba activate snakemake

# Install git to upload the pipeline to the VM
sudo apt-get install git
git clone https://github.com/Dartmouth-Data-Analytics-Core/DAC-RNAseq-pipeline.git
cd DAC-RNAseq-pipeline/

# update the versions of multiqc and snakemake
vim /home/tsully87/mambaforge/envs/snakemake/lib/python3.12/site-packages/snakemake/cli.py #770
vim /home/tsully87/mambaforge/envs/snakemake/lib/python3.12/site-packages/snakemake/scheduler.py #610
vim env_config/multiqc.yaml #python to 3.8
```

## 5. Build and index references used in the workflow
```
snakemake -s Snakefile  --use-conda -j 6  build_refs
# edit the config file to specify references - make the following line of code more clear
cat ref/pipeline_refs/hg38_chr567_100k.entries.yaml >> config.yaml
# Check that the files required for the specified references exist
snakemake -s Snakefile  --use-conda -j 6  check_refs
```

## 6.Run the workflow on the VM with the CLI

```
# run the pipeline on the VM using conda
snakemake -s Snakefile  --use-conda -j 6`
```
Please report any issues with creating an account [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-general/).
