# Spoke deployment of an Ubuntu VM with Docker and Azure Data Lake Storage

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FDataSnowman%2Fspoke%2Fmaster%2FdeploySpoke%2Fazuredeploy.json" target="_blank">
    <img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png"/>
</a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2FDataSnowman%2Fspoke%2Fmaster%2FdeploySpoke%2Fazuredeploy.json" target="_blank">
    <img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/visualizebutton.png"/>
</a>

1. This ARM template allows you to deploy an Ubuntu VM with Docker (using the installed Docker Extension).  

    Following the deployment you can run `docker` commands by connecting to the virtual machine with SSH.  

2. The example used in this GitHub repo uses SQL Server 2019 running in a Docker container image on the Ubuntu Linux VM you deployed above. 

    `Persist SQL Server files on VM disk`
    
    Click [HERE](https://github.com/DataSnowman/spoke/tree/master/dockerSQL) for directions on how to deploy SQL Server to Docker on the deployed VM with persistant SQL Server files on the VM disks

    `Simpler with SQL Server files in the container`
    
    Click [HERE](https://github.com/DataSnowman/spoke/blob/master/dockerSQL/simpler.md) for directions on how to deploy SQL Server to Docker on the deployed VM with SQL Server files in the container.  Files will be lost if you remove the container.

    The SQL Server part is based on the [Quickstart: Run SQL Server container images with Docker](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-bash)