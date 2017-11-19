## Framework to Deploy MATLAB at Scale with Azure Batch

** under construction **

These recipes provide a framework to deploy MATLAB at huge scale (up to 100's of 1000's of cores) on Azure.

Four recipes are provided as examples. All four recipes execute the exact same procedure on different platforms & configurations: 

* Linux with Task Factory Parametric Sweep using Azure CLI Extensions
* Windows with Task Factory Parametric Sweep using Azure CLI Extensions
* Batch Shipyard with Linux + Docker Containers
* Batch Container platform with Linux

The MATLAB example provided is compiled and deployed along with a MATLAB runtime library. Depending on the scenario/recipe, the code, data and runtime library will be packaged up into either a container, or an Azure Batch Application package (basically a zip file). 

The Azure CLI provides a very simple and convenient method to run Azure Batch without bothering with GUI's or API code. JSON templates make the process even easier, and you can simply define your pools and jobs in templates and run everything from the CLI, or wrap CLI commands into a simple bash script.

When you create your pools, the MATLAB package will be deployed onto all the nodes in the pool. Input data is automatically uploaded, and output data is automatically downloaded to a storage account at the end of the job. JSON templates define your pools and your jobs, and determine which tasks run on which nodes with which parameters. 
