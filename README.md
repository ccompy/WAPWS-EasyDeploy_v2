WAPWS-EasyDeploy_v2
===================

Introduction
The purpose of the tool is to setup a WAP Web Sites environment on a single HyperV host, thus allowing user to experience the basics of WAP Web Sites.  This is suitable for performing POC's, demos and doing testing.  It is not intended to be used to create a production deployment.  Please look to the technet information here: http://technet.microsoft.com/en-us/library/dn457745.aspx for production installation instructions.
	
Prerequisite
	a. Host machine requirement, 8960MB memory required, 40GB hard disk needed, supported OS's are:  2008 Server, Windows 8 and Windows 2012 Server
	b. Internet access is needed for all VM's. The network adapter in every VM will connect to
		i. An existing virtual network named as "WAP Network"
		ii. If not there, the first available external virtual network
		iii. If there are no existing virtual networks, a new external virtual network with name "WAP Network" will be created
	c. A sys-preped OS virtual hard disk, which will be used by all VM's as the base. Both VHD and VHDX is supported (be noticed that 2008 Server does not support VHDX). 
		i. Get a VHD from the internet. Server 2012 R2 can be downloaded from http://technet.microsoft.com/en-us/evalcenter/hh670538.aspx
		ii. Or DIY by running "sysprep.exe /generalize /oobe /shutdown" in a VHD-making VM. It is recommended to add a read-only flag to that disk right after the VM shutdown and never turn on the VHD-making VM.
	d. DeployWAP.exe must be run with administrative level if UAC is enabled.
			 
Command Line Options
	a. [Required] -BaseVHD 
	A VHD file served as the base for all VM's. Should contains a basic OS installation and sys-preped.
	b. [Optional] -SiteDomain
	Default to "test.com", which is the domain suffix for web sites
	c. [Optional] -VMSuffix
	Default to host machine name, which will be appended at the end of VM name and corresponding machine name
		
Process Details
	a. 8 VMs will be created with the name as "<Prefix>-<VMSuffix>". Default VMSuffix is the machine name, and any VMSuffix will be truncated to 10 characters to make actual machine name valid. If there is a corresponding VM already using the name, it will got reimaged. Originally attached VHD will be deleted.
	The <Prefix> is
	CN	Controller
	DB	Database Server
	FS	File Server
	LB	Load Balancer
	MN	Management Server
	PB	Publisher
	PT	Portal
	W	Shared Worker
	b. VHD's will be created under the HyperV's "Virtual Hard Disk" path, which can be changed through HyperV management UI
	c. Inside each VM, user "antaresadmin" with password "Wap2013!" will be created and assigned to "Administrators" group. 
	d. Both Sql and MySql will be installed on the database server. The admin user for Sql server is "sa" with password "Wap2013!". The admin user for MySql server is "root" with password "Wap2013!"
	e. The management end point is on the management server, port 443 with basic authentication. The user is "cloudadmin" with password "Wap2013!"
	f. A plan "WAP Default Plan" will be created after successful deployment and promotion code is "Wap2013!"
	
Using WAP Web Sites
	a. This install does not supply it's own DNS.  In order to use the system it needs to be properly configured with DNS per the guidance here: http://technet.microsoft.com/en-us/library/dn469319.aspx#BKMK_DNS
	b. If performing a quick demo and configuring DNS is too much overhead then the you can create entries in the windows\system32\etc\hosts file to add the ftp and publish entries as well as the entries for each website being created.

Trouble Shooting
	a. DeployWAP failed on creating or modifying Vm
	Cause: most likely DeployWAP can't access the HyperV WMI interface
	Solution: run DeployWAP as administrator
	b. DeployWAP can't mount the Vhd
	Cause: Target Vhd is still mounted in the host machine
	Solution: On 2008 Server, use WMI Msvm_ImageManagementService to umount. On 2012 Server, VHD can be directly umounted from Disk Management. Bottom line, reboot the host machine.
	c. Host machine lost network connectivity during deployment
	Cause: DeployWAP is creating a new virtual network and disrupted the connectivity
	Solution: Wait till connectivity restored. To prevent it from happening, make sure there is virtual network named "WAP Network" or there is an existing external virtual network
	d. No user "antaresadmin" in the VM
	Cause: initial setup script was not properly executed during first boot, and this is because the base vhd is not properly syspreped 
	Solution: Get the official VHDor do nothing after "sysprep.exe"
	e. Machine names not resolved , as a result, Infinite waiting for controller and other waiting
	Cause: the virtual network all VMs connected to are not configed correctly
	Solution: Make sure the virtual network VMs connected to are healthy and can provide correct name resolution
	f. Infinite waiting for Azure pack to be installed
	Cause: WebPI failed
	Solution: The failure model is quite complex, the simplest solution is to rerun DeployWAP. To find out the real trouble, go inside the Vm and check the WebPI log.
