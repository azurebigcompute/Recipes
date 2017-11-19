
# Recipe: Deploying Azure Batch with Custom Images

Custom Images avoid the overhead of installing applications dynamically, which can slow down pool deployment times. Azure has supported Custom Images for a very long time, but until recently, when you needed to deploy a custom image on the Azure Batch platform it was necessary to deploy with "User Mode Batch", which came with several limitations. The good news is that Azure now supports custom images on the mainstream Azure Batch Platform offering. The primary documentation for custom images with Azure Batch is available <a href="https://docs.microsoft.com/en-us/azure/batch/batch-custom-images">here</a>, and covers deployment via the Azure Portal and PowerShell in some detail. 

These pages describe in more detail the steps necessary to deploy Azure Batch with Custom Images:

* <b>Recipe 1: </b><a href="https://github.com/azurebigcompute/recipes/blob/master/Azure%20Batch/CustomImages/CustomImageCLI.md">Using the Azure Batch CLI</a>

* <b>Recipe 2: </b><a href="https://github.com/azurebigcompute/recipes/blob/master/Azure%20Batch/CustomImages/CustomImagePython.md">Using a Python API script. </a>
</b>

In both recipes we are deploying the same CentOS 7.1 Image with some preinstalled chemistry applications (NAMD, Gromacs & Amber), that we want to deploy on Azure GPU instances. The recipes are adaptable to any type of VM or Operating System. 

These templates are intended purely as functional examples, and may require further work to make them production ready. 

Credit to David Macleod of CHPC South Africa for providing the custom image that was used to test these recipes. 
