# Recipe 2: Deploying Azure Batch Custom Image with a Python Script

Sometimes you want complete control of batch via the API, or just want to give your users a single python script to run so they can stand up their own CPU or GPU nodes and run their Azure Batch jobs by themselves. 

This script takes the example Python script from the <a href="https://docs.microsoft.com/en-us/azure/batch/batch-python-tutorial">Azure Batch SDK Tutorial</a> and adapts it to use a Custom OS Image. 

## Pre-Requisites

In <a href="https://github.com/azurebigcompute/recipes/blob/master/Azure%20Batch/CustomImages/CustomImageCLI.md">recipe 1</a>, we showed you how to create all of the configuration and infrastructure objects needed to create a batch pool with a custom image. In this recipe, you will need to execute steps 1-3 in recipe 1 and create your resource group storage account, batch account and service principal before you begin. 

In addition to the pre-reqs from recipe 1, you also need to check you have the <a href="https://pypi.python.org/pypi/azure-batch">azure-batch==4.0</a> python API installed in order to use the new CustomImage feature: 
```
$ pip freeze | grep batch
azure-batch==4.0.0
azure-batch-extensions==1.0.1
azure-cli-batch==3.1.7
azure-cli-batchai==0.1.3
azure-mgmt-batch==4.1.0
azure-mgmt-batchai==0.2.0
```
If not, then do: 
```
$ pip install –upgrade azure-batch 
```
and check again. 

## Step 1: Gather the Keys and Configuration

Before we start, we need to first gather together all of the information necessary to have <a href="https://github.com/azurebigcompute/Recipes/blob/master/Azure%20Batch/CustomImages/custompoolexample.py">our example python script</a> authenticate and communicate with all of the batch features. This is a bit tedious, but generally you will need to do this the first time, and after that just cut & paste the values into your various scripts. 

You will notice a section at the top of the python script:
```
_BATCH_ACCOUNT_NAME = 'mikebatchwe'
_BATCH_ACCOUNT_KEY = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=='
_BATCH_ACCOUNT_URL = 'https://mkbatchwe.westeurope.batch.azure.com'

_STORAGE_ACCOUNT_NAME = 'mkbatchwestore'
_STORAGE_ACCOUNT_KEY = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=='

_BATCH_CLIENT_ID='XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX'         #-- Application ID ("appId")
_BATCH_SECRET='XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX'
_BATCH_TENANT_ID='XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX'
_BATCH_PRINCIPAL='mkbatchappwe'
_BATCH_PRINCIPAL_URL='http://mkbatchappwe'
```
We have to trawl these values from a few Azure CLI commands

Firstly, from the service principal creation you did earlier (see recipe 1) you have much of the information needed. 

```
$ az ad sp create-for-rbac --name mkbatchappwe --output json
{
  "appId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",        << This is your _BATCH_CLIENT_ID >>
  "displayName": "mkbatchappwe",                          << This is your _BATCH_PRINCIPAL>>
  "name": "http://mkbatchappwe",                          << This is your _BATCH_PRINCIPAL_URL>>
  "password": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",     << This is your _BATCH_SECRET>>
  "tenant": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"        << This is your _BATCH_TENANT_ID>>
}
```
You can also pull some of theses values if necessary from other commands such as:
```
$ az ad app list --display-name mkbatchappwe --output json
$ az ad sp list --display-name mkbatchappwe --output json
$ az account show --output json
$ az batch account show --name mkbatchwe --resource-group mkbatchwegrp --output json
```
If you did not record the password for the Service Principal, you will need to go to the portal and generate another one, or recreate the sp from the command line again. 

Get the Azure Batch Account Key (you only need the "primary" key): 
```
$ az batch account keys list --name mkbatchwe --resource-group mkbatchwegrp --output json
{ 
  "accountName": "mkbatchwe",   << This is your _BATCH_ACCOUNT_NAME >>
  "primary": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX==", 
  "secondary": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=="
}
```
And the "primary" key value is your _BATCH_ACCOUNT_KEY value. 
```
$ az batch account list --output json | grep accountEndpoint
    "accountEndpoint": "mkbatchwe.westeurope.batch.azure.com"
```
Gives you your _BATCH_ACCOUNT_URL value. 

Get the Storage Account Key (you only need to record the primary): 
```
$ az storage account keys list  --account-name mkbatchwestore --resource-group mkbatchwegrp --output json
[
  {
    "keyName": "key1",
    "permissions": "Full",
    "value": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=="
  },
  {
    "keyName": "key2",
    "permissions": "Full",
    "value": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=="
  }
]
```

Here the _STORAGE_ACCOUNT_NAME is the storage account name you created in recipe 1 ('mkbatchwestore' in this case), and the _STORAGE_ACCOUNT_KEY is the "value" field for "key1". 

Once you have all of the values together, simply edit <a href="https://github.com/azurebigcompute/Recipes/blob/master/Azure%20Batch/CustomImages/custompoolexample.py">the python script</a> and insert the values. 

## Step 2: Execute the script

Note that the jobs/tasks are a little meaningless here, but we have left them in to keep this as close to the tutorial examples as possible. You can replace them with some namd, gromacs & amber command lines + input data as necessary. 
```
$ python3 custompoolexample.py
Sample start: 2017-11-18 06:47:34

Uploading file /home/mk/python_tutorial_task.py to container [application]...
Uploading file /home/mk/data/taskdata1.txt to container [input]...
Uploading file /home/mk/data/taskdata2.txt to container [input]...
Uploading file /home/mk/data/taskdata3.txt to container [input]...
Creating pool [CHPCPool01]...
Creating job [PythonTutorialJob]...
Adding 3 tasks to job [PythonTutorialJob]...
Monitoring all tasks for 'Completed' state, timeout in 0:20:00......................................................................................................................................................................................................................
  Success! All tasks reached the 'Completed' state within the specified timeout period.
Downloading all files from container [output]...
  Downloaded blob [taskdata1_OUTPUT.txt] from container [output] to /home/mk/taskdata1_OUTPUT.txt
  Downloaded blob [taskdata2_OUTPUT.txt] from container [output] to /home/mk/taskdata2_OUTPUT.txt
  Downloaded blob [taskdata3_OUTPUT.txt] from container [output] to /home/mk/taskdata3_OUTPUT.txt
  Download complete!
Deleting containers...

Sample end: 2017-11-18 06:51:39
Elapsed time: 0:04:05

Delete job? [Y/n] Y
Delete pool? [Y/n] Y

Press ENTER to exit...
$
```
Monitor the pool creation and the jobs in your Batch Labs GUI, and it should line up with the output from the python script. 
