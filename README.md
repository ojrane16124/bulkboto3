<!--# Bulk Boto: Python package for parallel and bulk operations on S3 based on boto3-->
<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/iamirmasoud/bulkboto">
    <img src="https://raw.githubusercontent.com/iamirmasoud/bulkboto/main/imgs/logo.jpg" alt="Logo" width="100" height="100">
  </a>
    
  <h3 align="center">Bulk Boto</h3>

  <p align="center">
    Python package for fast and parallel transfer of bulk of files to S3 based on boto3!
    <br />
    <!-- 
    <a href="https://github.com/iamirmasoud/bulkboto"><strong>Explore the docs »</strong></a>
    <br /> 
    -->
    <a href="https://github.com/iamirmasoud/bulkboto/blob/main/examples.py">View Examples</a>
    ·
    <a href="https://github.com/iamirmasoud/bulkboto/issues">Report Bug/Request Feature</a>
  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-bulk-boto">About Bulk Boto</a>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#contributors">Contributors</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#license">License</a></li>
  </ol>
</details>

## About Bulk Boto
[Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html) is the official Python SDK 
for accessing and managing all AWS resources such as Amazon Simple Storage Service (S3). 
Generally, it's pretty ok to transfer a small number of files using Boto3. However, transferring a large number of 
small files impede performance. Although it only takes a few milliseconds per file to transfer, 
it can take up to hours to transfer hundreds of thousands, or millions, of files if you do it sequentially. 
Moreover, because Amazon S3 does not have folders/directories, managing the hierarchy of directories and files 
manually can be a bit tedious especially if there are many files located in different folders.

The Bulk Boto package solves these issues. It speeds up transferring of many small files to Amazon AWS S3 by 
executing multiple download/upload operations in parallel by leveraging the Python multiprocessing module. 
Depending on the number of cores of your machine, Bulk Boto can make S3 transfers even 100X faster than sequential 
mode using traditional Boto3! Furthermore, Bulk Boto can keep the original folder structure of files and 
directories when transferring them. There are also some other features as follows.

### Main Functionalities
  - Multi thread uploading/downloading of a directory (keeping the directory structure) to/from S3 object storage
  - Deleting all objects of an S3 bucket
  - Checking the existence of an object on the S3 bucket
  - Listing all objects on an S3 bucket
  - Creating a new S3 bucket on the object storage

## Getting Started
### Prerequisites
* [Python 3.3+](https://www.python.org/)
* [pip](https://pip.pypa.io/en/stable/)
  
### Installation
Use the package manager [pip](https://pip.pypa.io/en/stable/) to install `bulkboto`.

```bash
pip install bulkboto
```

## Usage
You can find the following scripts in [examples.py](https://github.com/iamirmasoud/bulkboto/blob/main/examples.py).

#### Import and instantiate a `BulkBoto` object with your credentials
```python
from bulkboto import BulkBoto
TARGET_BUCKET = "test-bucket"
NUM_TRANSFER_THREADS = 50
TRANSFER_VERBOSITY = True

bulkboto_agent = BulkBoto(
    resource_type="s3",
    endpoint_url="<Your storage endpoint>",
    aws_access_key_id="<Your access key>",
    aws_secret_access_key="<Your secret key>",
    max_pool_connections=300,
    verbose=TRANSFER_VERBOSITY,
)
```

#### Create a new bucket
```python
bulkboto_agent.create_new_bucket(bucket_name=TARGET_BUCKET)
```

####  Upload a whole directory with its structure to an S3 bucket in multi thread mode
Suppose that there is a directory with the following structure on your local machine:
```console
test_dir
├── first_subdir
│   ├── f1
│   ├── f2
│   └── f3
└── second_subdir
    └── f4
```
To upload the directory (with its subdirectories) to the bucket 
under a new directory name called `my_storage_dir`:
```python
bulkboto_agent.upload_dir_to_storage(
     bucket_name=TARGET_BUCKET,
     local_dir="test_dir",
     storage_dir="my_storage_dir",
     n_threads=NUM_TRANSFER_THREADS,
)
# output:
# 2022-03-26 18:12:40 — INFO — Start uploading from local 'test_dir' to 'my_storage_dir' on the object storage with 50 threads.
# 100%|██████████| 4/4 [00:00<00:00,  4.00s/it]
# 2022-03-26 18:12:41 — INFO — Successfully uploaded 4 files to bucket 'test-bucket' in 0.07 seconds.
```

#### Download a whole directory with its structure to a local directory in multi thread mode
```python
bulkboto_agent.download_dir_from_storage(
    bucket_name=TARGET_BUCKET,
    storage_dir="my_storage_dir",
    local_dir="new_test_dir",
    n_threads=NUM_TRANSFER_THREADS,
)
# output: 
# 2022-03-26 18:14:08 — INFO — Start downloading from 'my_storage_dir' on storage to local 'new_test_dir' with 50 threads.
# 100%|██████████| 4/4 [00:00<00:00,  4.00it/s]
# 2022-03-26 18:14:09 — INFO — Successfully downloaded 4 files from bucket: 'test-bucket' in 0.04 seconds.
```

The structure of downloaded directory will be as follows on the local directory:
```console
new_test_dir
└── my_storage_dir
    ├── first_subdir
    │   ├── f1
    │   ├── f2
    │   └── f3
    └── second_subdir
        └── f4
```
You can set `local_dir=''` (it is the default value) to avoid the creation of the `new_test_dir` directory. 

#### Delete all objects on a bucket
```python
bulkboto_agent.empty_bucket(TARGET_BUCKET)
# output: 
# 2022-03-26 22:23:23 — INFO — Successfully deleted objects on: 'test-bucket'.
```

#### Check if a file exists in a bucket
```python
print(
    bulkboto_agent.check_object_exists(
        bucket_name=TARGET_BUCKET, object_path="my_storage_dir/first_subdir/test_file.txt"
    )
)
# output: False 

print(
    bulkboto_agent.check_object_exists(
        bucket_name=TARGET_BUCKET, object_path="my_storage_dir/first_subdir/f1"
    )
)
# output: True
```

#### Get list of objects in a bucket (with prefix)
```python
print(
    bulkboto_agent.list_objects(
        bucket_name=TARGET_BUCKET, storage_dir="my_storage_dir"
    )
)
# output: 
# ['my_storage_dir/first_subdir/f1', 'my_storage_dir/first_subdir/f2', 'my_storage_dir/first_subdir/f3', 'my_storage_dir/second_subdir/f4']

print(
    bulkboto_agent.list_objects(
        bucket_name=TARGET_BUCKET, storage_dir="my_storage_dir/second_subdir"
    )
)
# output: 
# ['my_storage_dir/second_subdir/f4']
```

### Benchmark
Uploaded 88800 small files (totally about 7GB) with 100 threads in 505 seconds that is about 
72X faster than the non-parallel mode.


## Contributing
Any contributions you make are **greatly appreciated**. If you have a suggestion that would make this better, please fork the repo and create a pull request. 
You can also simply open an issue with the tag "enhancement". To contribute to `bulkboto`, follow these steps:

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Make your changes and commit them (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a pull request

Alternatively see the GitHub documentation on [creating a pull request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request).

## Contributors
Thanks to the following people who have contributed to this project:

* [Amir Masoud Sefidian](https://sefidian.com/) 📖

## Contact
If you want to contact me you can reach me at [amir.masoud.sefidian@gmail.com](mailto:amir.masoud.sefidian@gmail.com).

## License
Distributed under the [MIT](https://choosealicense.com/licenses/mit/) License. See `LICENSE` for more information.


