# GoogleCloudAccess
Instructions for running the DAC bulk RNAseq data reduction pipeline on Google Cloud

# BUILD VM ON GOOGLE CLOUD
# how do you spin up VM what size, memory and processing power configurations??
# where is the link to where a VM can be created?
# what OS do you specify?

# INSTALL TOOLS ON GOOGLE CLOUD
# Install conda forge on a virtual machine
`wget https://github.com/conda-forge/miniforge/releases/download/23.3.1-1/Mambaforge-23.3.1-1-Linux-x86_64.sh`
`chmod +x Mambaforge-23.3.1-1-Linux-x86_64.sh`
`./Mambaforge-23.3.1-1-Linux-x86_64.sh`

`exec bash`

# create a conda environment that contains snakemake
`mamba create -c conda-forge -c bioconda -n snakemake snakemake`

`mamba activate snakemake`

# Install git to upload the pipeline to the VM
`sudo apt-get install git`
`git clone https://github.com/Dartmouth-Data-Analytics-Core/DAC-RNAseq-pipeline.git`
`cd DAC-RNAseq-pipeline/`

# Tim manually edited something about these files - what was changed are these modifications necessary?
`vim /home/tsully87/mambaforge/envs/snakemake/lib/python3.12/site-packages/snakemake/cli.py` #770
`vim /home/tsully87/mambaforge/envs/snakemake/lib/python3.12/site-packages/snakemake/scheduler.py` #610
`vim env_config/multiqc.yaml` #python to 3.8

# SET UP REFERENCES ON GOOGLE CLOUD
# Build and index reference files
`snakemake -s Snakefile  --use-conda -j 6  build_refs`
# adjust the settings in the config file to match the configuration of your experiment and include variables for the references you've just formatted
`cat ref/pipeline_refs/hg38_chr567_100k.entries.yaml >> config.yaml`

# Check that the files required for the specified references exist
`snakemake -s Snakefile  --use-conda -j 6  check_refs`

# RUN THE ANALYSIS ON GOOGLE CLOUD
# run the pipeline on the VM using conda
`snakemake -s Snakefile  --use-conda -j 6`

# WRITE RESULTS TO GOOGLE CLOUD BUCKET
# write the results to a google bucket for access later
`gcloud auth login`
`gsutil rsync -r -x '\..*|./[.].*$'  . gs://rna-seq-output-test`

# how do you share google bucket with user?



# SURVEY QUESTIONS

1. Please report any issues with setting up the virtual machine [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-forms/).
2. Please report any issues setting up the following tools [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-forms/). 
  - Conda
  - Snakemake
  - Tools within the Snakemake pipeline
3. Please report any issues formatting references [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-forms/).
4. Please report any issues that do not fit into the categories above [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-forms/).
