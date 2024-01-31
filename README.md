# GoogleCloudAccess
Instructions for running the DAC bulk RNAseq data reduction pipeline on Google Cloud. 


## 1. Set up a Google Account that enables you to access the cloud

## 2. Set up Google Cloud project and link to a billing account

## 3. Build VM on Google cloud
  - link to where a VM can be created?
  - VM configurations?
    - what OS do you specify?
    - RAM?
    - CPUs?
   
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
## 7. Create a google bucket to store results long term within your project. Any results not saved to this bucket will be lost when the VM is closed.

```
#create a Google Bucket -replace BUCKET_NAME with your preferred name
gcloud storage buckets create gs://BUCKET_NAME 
```

```
#create and configure a Google Bucket
gcloud storage buckets create gs://BUCKET_NAME \
  --project MY_PROJECT \
  --billing-project MY_BILLING_ACCOUNT \ 
  --default-storage-class STANDARD \ # could be "STANDARD" "NEARLINE" "COLDLINE" "ARCHIVE"
  --location US-EAST1 \
  --uniform-bucket-level-access \ # set IAM access for all files in the bucket to be the same
  --public-access-prevention \ # protects bucket from being made public accidently
  --retention-period=1Y1M1D5S \ # the bucket must be retained for this minimum period of time (1 year, 1month, 1day, and 5seconds)
  --soft-delete-duration=2W1D \ # the period of time to recover deleted files (2weeks and 1day)
  --enable-autoclass \ # enable autoclass to adjust storage tier based on the date of last access --no-enable-autoclass is also available
```

```
# write the results to a google bucket for access later
gcloud auth login
gsutil rsync -r -x '\..*|./[.].*$'  . gs://BUCKET_NAME
```

## 8. Manage permissions of the bucket to share data with collaborators

To set and manage permissions for a bucket you must have a Storage Admin role (roles/storage.admin). 

[Here](https://cloud.google.com/iam/docs/principal-identifiers) is a link to various principal identifiers that can be used to specify access.
[Here](https://cloud.google.com/storage/docs/access-control/iam-roles) is a link to various roles that control the level of access the user has to the data in the bucket.
```
# Check IAM permissions for a given bucket
gcloud storage buckets get-iam-policy gs://BUCKET_NAME

# Add IAM policy to a storage bucket for a new user
gcloud storage buckets add-iam-policy-binding gs://BUCKET_NAME \
  --member=PRINCIPAL_IDENTIFIER \ # ex. user:jane@gmail.com 
  --role=IAM_ROLE  # ex. roles/storage.objectViewer 

# Remove bucket access for a given user
gcloud storage buckets remove-iam-policy-binding  gs://BUCKET_NAME 
--member=PRINCIPAL_IDENTIFIER \ # ex. user:jane@gmail.com 
--role=IAM_ROLE # ex. roles/storage.objectViewer
```

# SURVEY QUESTIONS

1. Please report any issues with setting up the virtual machine [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-forms/).
2. Please report any issues setting up the following tools [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-forms/). 
  - Conda
  - Snakemake
  - Tools within the Snakemake pipeline
3. Please report any issues formatting references [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-forms/).
4. Please report any issues that do not fit into the categories above [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-forms/).
