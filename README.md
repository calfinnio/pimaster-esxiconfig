# PiMaster-ESXiConfig  
Hopefully you are following this repo from the PiMaser bootstrap repo:  
    https://github.com/seroles/pimaster.git  

If you have then the following should be in place:  
  - ESXi booted and running the correct version (whatever you extracted)
  - Root user is associated with the homelab user ssh key
  - Golang and govc are installed on the Pi
  - DNS records are in place for pfsense, vcsa and the host
  - secrets_physical_esx.yml has been created in the role vars directory
  - Inventory file and settings have been configured by previous repo.  

## Aim of this repo
This repo should take your homelab to the next level.  It will perform the following:
 - Deploy standard switches as definied in main.yml
 - Deploy portgroups as definied in main.yml
 - Find a suitable physical disk and create a vmfs instance
 - Hook up NFS published form PiMaster
 - Deploy a pfsense vm (maual config needed for this - see pfsense_to_ovf.md)
 - Deploy VCSA
 - Create subscribed content library (William Lams Nested ESXi Library)
 - Deploy x number of virtual ESXi nodes from that library  

This should give you a homelab in a box to do with as you please.

## First Run
As mentioned above the pfsense ovf deploy that is integral to this lab requires some manual intervention.  This is because there is no automated install script for pfsense so it requires a manual install and then configuration. It can then be exported as an OVF to the PiMasterNFS and subsequent runs will install from this baseline.  The steps needed to get it running are detailed in pfsense_to_ovf.md.

## Variables Needed
The following variables are required for the lab to run:  
### content_library.yml
|Variable Name|Purpose| My Value|
|---|---|---|  
| subscription_url | URL of William Lams Nested ESXi Library | https://download3.vmware.com/software/vmw-tools/lib.json |  
| content_library_name | Name of the content library in vCenter - whatever you want | vGhettoLib |  
  
### main.yml
This var file has a different structure than most as the vswitches and ports are created as a dictionary to be ingested later.  The structure is as follows:  
```  
network_vars:  
    <vswitch_name>:  
      portgroup_name: <Name of portgroup to create>
      default_portgroup: <boolean>
```
This will create the required named vswitch and a portgroup on it.  If the boolean is set to true this will set that as the efault portgroup for subsequent VM builds.  

### nfs-vars.yml  
|Variable Name|Purpose| My Value|
|---|---|---|  
| nfs_mount_name | Name of the NFS share within vCenter | PiMasterNFS |  
| nfs_share_path | Path to NFS share on remote machine | /mnt/usb/nfs |  
  
### pfsense_vars.yml  
|Variable Name|Purpose| My Value|
|---|---|---|  
| pfsense_vm_name | Name of the VM | pfsense |
| pfsense_wan_pg | WAN network portgroup |  "VM Network" |
| pfsense_wan_ip | WAN IP address | 192.168.1.12 |
| pfsense_lan_pg | LAN network portgroup | pg_LabMgmt |
| pfsense_lan_ip | LAN IP address| 10.64.0.1 |
| pfsense_provision_mode | VM disk provision mode | thin |
| pfsense_ovftool_path | Path to ovftool | /vmfs/volumes/{{ default_nfs }}/vmware-ovftool/ovftool |
| pfsense_ovf_path | Path to previously exported pfSense OVF | /vmfs/volumes/{{ default_nfs }}/pfsense.ovf |
| pfsense_new_config | Boolean if you want to import the below config.xml | false
| pfsense_config_path | Path to where a previous config.xml is stored | /mnt/nfs/ |    
| pfsense_config_filename | NAme of backuped up pfsense config | config.xml |
  
### vcsa_vars.yml  
|Variable Name|Purpose| My Value|
|---|---|---|  
| vcsa_vm_name | Name of VM |  vcsa |
| vcsa_network | Network to connect VCSA to (should be sae as pfsense LAN portgroup) |  pg_LabMgmt |
| vcsa_ip_addr | IP address of VCSA |  10.64.0.20 |
| vcsa_ip_prefix | CIDR/Subnetmask |  24 |
| vcsa_gateway | Gateway (should be same as pfsense LAN IP |  10.64.0.1 |
| vcsa_dns | DNS for VCSA (should be pfsense VM which will forward to PiMaster) |  10.64.0.1 |
| vcsa_dns_name | What is the FQDN of the VCSA |  vcsa.houseofbears.co.uk |
| vcsa_dns_domain | Domain name |  houseofbears.co.uk |
| vcsa_username | VCSA admin name |  administrator |
| vcsa_sso_domain | SSO Domain name (best left as is) |  vsphere.local |
| vcsa_site_name | Site name |  Site01 |
| vcsa_dc_name | Name for daacenter in vSphere |  HomeLab |
| vcsa_ova_path | Path to VCSA OVA |  /vmfs/volumes/{{ default_nfs }}/vcsa/vcsa2.ova |  

### vesxi_vars.yml
|Variable Name|Purpose| My Value|
|---|---|---|  
| vesxi_ovf_name | NAme of the OVF to deploy from content library |  Nested_ESXi6.7u3_Appliance_Template_v1.0 |
| vesxi_host_count | How many hosts to deploy |  3 |
| vesxi_starting_ip | What IP address to start from |  10.64.0.21 |
| vesxi_last_octet | Last octet of aboe address |  21 |
| vesxi_basename | Base VM name prefix e.g. VM will be vesxi1, vesxi2 etc. |  vesxi |  

The OVF names all seem to follow the same convention:  
``` 
Nested_ESXi6.7u3_Appliance_Template_v1.0  
```  
So one could guess at what a 6.5 Update 2 host name might be...for example.

### vmfs_vars.yml  
|Variable Name|Purpose| My Value|
|---|---|---|  
| vmfs_guid | GUID used for VMFS partition creation.  Shouldn't really change |  AA31E02A400F11DB9590000C2911D1B8 |
| vmfs_start_sector | Default start sector for a VMFS volume |  2048 |
| vmfs_attribute | Default VMFS attribute |  0 |
| vmfs_device_path | Shortening for path to disks on ESXi |  /vmfs/devices/disks/ |
| vmfs_version | VMFS version to use |  6 |
| vmfs_name | Name for new datastore|  General1 |
| vmfs_extra_datastores | Create extra datastores?  These require device paths so best created after an initial config has completed and you can see the other disks. |  false |




