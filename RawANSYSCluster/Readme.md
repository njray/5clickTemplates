# Simple deployment of a VM Scale Set of Linux VMs with a jumpbox

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ftanewill%2F5clickTemplates%2Fmaster%2FRawANSYSCluster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<b>Quickstart</b>

<b>Architecture</b>

<img src="https://github.com/tanewill/5clickTemplates/blob/master/images/hpc_vmss_architecture.png" alt="hpc_vmss_architecture" border="1"/> 
This template is designed to assist in the assessment of the ANSYS Fluent CFD package in the Microsoft Azure environment. It automatically downloads and configures Fluent. In addition it authenticates all of the nodes on the cluster and creates a common share directory to be used for each of the nodes. A Virtual Machine Jumpbox is created and a Virtual Machine Scale Set (VMSS) of the same type of machine is created. The VMSS enables easy scaling and quick deployment of the cluster. The Jumpbox serves as the head node. A network card is attached to the Jumpbox and placed in a Virtual Network. The Jumpbox and VMSS reside in the same virtual network. A public IP is assigned to the network with port 22 open. The Jumpbox can be accessed with the following command:

<code>ssh {username}@{vm-private-ip-address}</code>

Four Storage Accounts are created for the VMSS and one for the Jumpbox. An NFS file share is created from the head node's OS disk and shared with all of the VMSS nodes. This NFS share is located at /mnt/nfsshare/ No other file sharing or server is used. The Jumpbox and each of the nodes in the VMSS also have a data disk mounted at /mnt/resource/ that provides local storage space.


<b>Software Configuration</b>

A number of packages are installed during deployment in order to support the NFS share and the tools that are used to create the authentication. During the authentication phase of the deployment, files named <u>nodenames.txt</u> and <u>nodeips.txt</u> are placed in ~/bin. These are the names and ip addresses of all of the nodes in the VMSS. Each of these nodes should be accesible with the following command:

<code>ssh {username}@{vm-private-ip-address}</code>

In addition ANSYS Fluent version 17.2 is installed into the /mnt/nfsshare/ directory and the path to the Fluent binary is added to ~.bashrc. The benchmark model that was selected at deploy time is downloaded and unpacked as <u>benchmark.cas.tgz</u> and <u>benchmark.dat.tgz</u> these are the 'Case' and 'Data' files for Fluent. They are placed in /mnt/resource on the Jumpbox. A file named runme.jou contains the scripting commands for Fluent. With these three files a benchmark can be run by issuing the following command.

<code>time(fluent 3d -g -mpi=intel -pib.dapl -mpiopt="-genv I_MPI_DAPL_PROVIDER=ofa-v2-ib0" -ssh -t48 -cnf=/home/azureuser/bin/nodenames.txt -i runme.jou)</code>


To ssh into one of the VMs in the scale set, go to resources.azure.com to find the private IP address of the VM, make sure you are ssh'ed into the jumpbox, then execute the following command:

ssh {username}@{vm-private-ip-address}

PARAMETER RESTRICTIONS
======================

vmssName must be 3-61 characters in length. It should also be globally unique across all of Azure. If it isn't globally unique, it is possible that this template will still deploy properly, but we don't recommend relying on this pseudo-probabilistic behavior.
instanceCount must be 100 or less.