# GoogleCloudAccess
Instructions for running the DAC bulk RNAseq data reduction pipeline on Google Cloud. 


## 1. Set up a Google Cloud account
To access cloud resources you will need to create a google cloud account with an *email addres*s and a *credit card*. 

Creating the account is free, compute resources and data storage you pay for what you use. With a grant one way to set this up is to link a p-card or corporate card to the google account and use the string to pay the monthly usage fees on the p-card. However, if you have an NIH grant you can receive up to a 20% discount on GCP storage or data processing costs. 

Cloud accounts created with Dartmouth College affiliated email address (i.e. Fred.Rogers[]()@dartmouth.edu) will be managed by [Research Computing](Research.Computing@dartmouth.edu). You should be able to create a project and billing will automatically be managed by Burwood. You should reach out to research computing with the details of the grant that supports the work you're doing so that they can provide those details to Burwood. You're grant will be charged on a monthly basis based on the storage and compute resources used in the project linked to your grant. 

You can follow the instructions on the [Google Cloud home page](cloud.google.com/free) to create an account. 

## 2. Set up Google Cloud project and link to a billing account

Currently we do not have the ability to make a new project in GDSC, to make a project send the title of the project to research computing and they will set up the project for you, even better you can also send them the grant number to charge.

## 3. Build VM on Google cloud

From the dashboard page (insert an image of this page) select the project that you would like to run the VM from in the drop down menu. 
Next go to the menu on the top left of the page and select **Compute Engine** and then select **VM instance**.
Click the blue button at the top of the page **Create instance**.
Select the region that is closest to you for faster data transfer, this will be negligible as long as you select a region in the united states. Here I'm selecting *us-east1*.
Next select the machine configuration that best meets the needs of your project, for the purposes of the RNAseq pipeline low cost day to day computing will be fine, so I will select *E2*. If you select a different configuration you can see how this would affect the projected monthly pricing on the right pane of the screen. 
Next select the machine type, again because we are using relatively few compute resources for our pipeline I've selected e2-micro, you can see the price drops from ~$25 to only $7 per month with the micro machine. You can see the size of the machine mostly affects the memory available though you do get more CPUs with a e2-medium machine.

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

1. Please report any issues with setting up the virtual machine [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-form-VM/).
2. Please report any issues setting up the following tools [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-tools/). 
  - Conda
  - Snakemake
  - Tools within the Snakemake pipeline
3. Please report any issues formatting references [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-refs/).
4. Please report any issues that do not fit into the categories above [here](https://sites.dartmouth.edu/cqb/google-cloud-analyst-feedback-general/).
