# DVC (Data Version Control)

DVC is an open-source tool designed to handle large files, datasets, and machine learning models — things that Git alone can't manage efficiently.  
Think of it this way:  
Git tracks your code changes (versions)  
DVC tracks your data changes (versions)  
Together, they give you complete version control over both code and data.  

S3 (Amazon Simple Storage Service)  
S3 is Amazon's cloud storage service. It acts as a remote storage location where your actual large data files are stored.  

### How DVC + S3 Work Together  
The problem:  
Git is not built to handle large files like datasets (e.g., 10GB CSV files). Pushing them to GitHub would be slow and impractical.  
The solution:  
DVC stores the actual data files in S3 (cloud) and keeps only a small pointer/reference file (.dvc file) in your Git repository.  

**Simple workflow:**
1. You add a large dataset using dvc add data.csv
2. DVC creates a small data.csv.dvc file (just a pointer)
3. You push the actual data to S3 using dvc push
4. The pointer file goes to Git, the real data goes to S3
5. When a teammate needs the data, they run dvc pull — it fetches the exact version from S3

**Data Pull** means downloading the correct version of data from S3 using DVC. Versioning means tracking every change to your dataset over time, so you can go back to any previous version.

Git is capable of doing auditing, versioning, RBAC for code repo’s
Same capabilities is required for Data also, But git not designed for  large files(GB, TB)

Why not Git for Data versioning
Wouldn’t handle large files    -  DVC handles  large files(GB, TB)
Slow while push & pull.        -   Fast
Cost in-effective                    -   Cost Effective(Cheap)
Local storage is limited         -   S3, Azure Blob, GCS (also supports versioning & Durability(99.99999))      
 
If you have datasets and python code in same folder.
Dvc add dataset -> .dvc file will generate which has dataset tracking info eg: wine.csv.dvc
Dvc push -> dvc/config file will generate which has datasets version info in remote storage s3.
Git push .dvc and dvc/config -> git (Git stores only a pointer file → wine.cvs.dvc and .dvc/config folder)

Dataset is stored in s3 bucket and metadata of dataset is stored in git for DS and other teams.

Push to Remote storage service
Dvc add winne.csv
aws sts get-caller-identity
Aws configure
Add S3 Permissions: AmazonS3FullAccess.
dvc remote list 
	If you need to change it: dvc remote modify myremote url s3://your-correct-bucket-name/path
python3 -m pip install dvc-s3
dvc remote add -d mlops-remote s3://mlops-ss3
Dvc push

Push wine.csv.dvc metadata of dataset file and .dvc folder to GitHub
Git add wine.csv
Git remote add origin https://github.com/Vinodh-Machireddy/MLOps-Project.git
Git push -u origin master
