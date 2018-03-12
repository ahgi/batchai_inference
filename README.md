Distributed Batched Scoring Using Azure Batch AI
========================

This recipe shows how to run distributed Batched scoring Using Azure Batch AI, and run benchmark in term of the GPU cluster scale. 

## Image Data, Codebase and Models

The image dataset for scoring is provided by CaiCloud and VIP.com. The dataset is hosted in Azure Storage account `octopustext` and under Blob Container `batchaisample`. The dataset contains 366111 jpg format images with total size approximately 45 GB. 

The main codebase and pertained model files are also provided by CaiCloud, and are hosted in the same storage account under File Share `abc`. The codebase contains all required dependency to run batch scoring of image classification and cloth recognition. Pretained VGG16 and Inception V3 models have also been uploaded to the File Share under the `output` directory.   

The Storage account `octopustext` is in East US data center. Please use [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) or [Azure Portal](https://portal.azure.com/) to view the detail directory structures in Blob Container `batchaisample` and File Share `abc`.

## Batch Scoring Job Script

The main script used for the scoring job `dist_inference.py` locates in the root directory of File Share `abc`. To view it, please download it from Azure Storage. 

The input argument `'--inference'` is used to specify the scoring task: `'cloth_recognition'` or `'classification'`. The main idea is to shad the whole dataset into partitions based on the total number of workers, and each work process its assigned partition of images independently. There is no inter-communication between each work. 

When the script completes, it outputs the total number of images it processed and how long it took, such as:

 ```sh
 Worker 0 Processed 3124 images, took 0:10:43.983157
 ```

Please feel free to edit/optimize the logics of the script if needed

## Prerequisites

Please place the filled `configuration.json` in the same directory. It should includes the Azure Batch AI authentication information and credentials of the Storage Account `octopustext`.

Install Azure Batch AI management client using the following command:
 
 ```sh
 pip install azure-mgmt-batchai
 ```

We may need to utilize APIs from other Azure products (e.g, Azure storage, credentials), it is also required to install the full package of Azure Python SDK:

 ```sh
 pip install azure
 ```

Install Jupyter Notebook from https://jupyter.org/ or run

```sh
python -m pip install jupyter
```

## Run the Batch Scoring Recipe

#### [Distributed-Scoring-Tensorflow](./Distributed-Scoring-Tensorflow.ipynb)

This Jupyter Notebook file contains information on how to run Batch Scoring job on a GPU node with BatchAI. You will be able to tune variables including `node_count`, `vm_size` to obtain different benchmark results.

Note that, since there is no communication between each worker, Parameter Server is not required in this case. Therefore, we use the `customToolkitSettings` in the Batch AI job definition (instead of `TensorFlowSettings`) and use OpenMPI to launch and monitor all workers more efficiently. The OpenMPI binary is installed in container using `JobPreparation` task.

## Benchmark Results

The below table illustrates elapsed time to label 100k images for `'cloth_inference'`(VGG-16) task:

|  Number of GPUs |       1       |      8     |      16    |     32     |
| --------------- |:-------------:| ----------:| ----------:| ----------:|
| K80             |  741 mins     |   99 mins  |   49 mins  |  25 mins   |
| P100            |  255 mins     |   32 mins  |   19 mins  |  10 mins   |

Qusi-linear scaling-up can be observed in terms of number of GPUs. 

The benchmark for `'classification'`(Inception-V3) task has not been done yet. The test code needs to be optimized to achieve higher GPU efficiency.

## Reference

- To transfer large amount of data between local device and Azure Storage, please use [AzCopy](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azcopy) or [Blobxfer](https://github.com/Azure/blobxfer/blob/master/docs/10-cli-usage.md) instead of Portal/Storage Explorer

- A detail reference of Batch AI python SDK can be found [here](https://docs.microsoft.com/en-us/python/api/azure.mgmt.batchai?view=azure-python)

- If you prefer to use Azure CLI 2.0 instead of python SDK, please see the [article](https://github.com/Azure/BatchAI/blob/master/documentation/using-azure-cli-20.md) for instruction.
