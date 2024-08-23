# vds-filter
This repository describes the process of filtering the All of Us (AoU) VariantDataset ([VDS](https://hail.is/docs/0.2/vds/index.html)) and converting the resulting matrix tables to bgens (v1.2).

## AoU Disclaimer
The input data required for this process is not publicly availbaly and researchers who do not already have "Control Tier Access' will need to go through the proper registration and training before attempting to access the VDS on AoU.

## Environment & Dependency Set Up
1) Conda is not viable on AOU, due to the root access required to run conda. Luckily, most of the dependencies are already installed on AoU. If you intend to run these scripts on the command line you must install 'defopt' before proceesing. 

```bash
pip install defopt
```

2) [Hail](https://hail.is/docs/0.2/index.html), is a python tool developed by The Broad Institute, which handles large scale genomic data and runs exclusively on a [Spark](https://spark.apache.org/docs/latest/api/python/index.html) cluster. Spark has already been set-up on AoU, all that users need to do is select "Hail Genomics Analysis" under "Recommended environments" when launching their cloud analysis environment.

```bash
# To test correct environment & dependency install
python modules/vds_to_matrix_table.py --help
```
### Required Inputs
There are three general steps to successfully creating custom bgens from the VDS:
1) Create a variant list in the for of a tab seperated file with the following colums:
|Column Name    |Description     |
|---------------|----------------|
|CHR        |chromsome number preceeded by 'chr' ie. che1|
|POS    |GRCh38 position|
|REF |Reference allele(s) (capitalized)|
|ALT |Alternate allele(s) (capitalized)|

2) VDS data, for researchers with controll tier access please see this help [article](https://support.researchallofus.org/hc/en-us/articles/5439665241876-How-do-I-select-specific-variants-from-the-Hail-MatrixTables-or-Hail-VDS) on accessing the dataset.

## Running The Analysis
After having been approved for controlled tier access, generating a variant list, copying the python scripts in 'modules/' to your AoU disk, and launching a hail cloud compute environmet you are ready to being filtering!

### Step 1
Per chromosome, filter the VDS for variants in your variant list
* This step can be run on pre-emptible workers as this step is row independent
```bash
python modules/vds_to_matrix_table.py --variant-file <Google Bucket Path to varaint tsv>   --vds-storage-location <Google Bucket Path to VDS> --output-dir <Google Bucket Path to output directory *must be AoU associated Google Bucket*>
```

### Step 2
Converting matrix tables to bgens
* This step must only be run on persistent workers as this is NOT row independent
```bash
python modules/matrix_table_to_bgen.py --input-mt-dir <Google Bucket Path to directory with matrix tables produced in step 1> --output-dir <Google Bucket Path to output directory *must be AoU associated Google Bucket*>
```

**It is highly recommended to test your processes with chr22 before launching a production run**
**If you prefer to run code though a notebook simply copy the function in modules/vds_to_matrix_table.py & modules/matrix_table_to_bgen.py to a jupyter notebook**


## Running Notebooks In The Background
This process takes several hours (>24) and is very expensive. However, this may be the only method of accessing your variants of interest. That being said, we recommend setting up a juptyer notebook which either calls the provided scripts in a bash cell OR runs the code directly. Either way you will want to run said notebook in the 'background' in order to avoid the automatic idle enforced on AoU. In order to do this follow the instructions provided [here](https://workbench.researchallofus.org/workspaces/aou-rw-7172272a/howtorunnotebooksinthebackground/analysis), and make sure to chack in on your run at lesst once every 24hrs, otherwise your environment wil be paused and your progress will be lost. 
