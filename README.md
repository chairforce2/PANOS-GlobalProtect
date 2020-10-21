# PANOS-GlobalProtect
Configuration for quickly configuring (particularly cloud hosted) VM-series to act as a GlobalProtect Portal and Gateway

This process assumes you have already been able to stand up a VM-Series firewall with a minimum of 3 NICs (1x mgmt, 2 x dataplane) with the Untrust address as eth1/1 and it being reachable from the internet (either by public IP allocated or via DNS). If you have not yet dont this I suggest you check out the reference archicture guides from Palo Alto Networks at https://www.paloaltonetworks.com/resources/reference-architectures

If you would like to automate the deployment of these reference archtictures you can visit https://github.com/PaloAltoNetworks/reference_architecture_automation
Or search through one of the Palo Alto Networks affiliated Github pages for a repo which matches your needs:

https://github.com/PaloAltoNetworks
https://github.com/wwce

I recently tested using this one in particular, and deployed it in a new resource group, with the BYOL option:
https://github.com/wwce/azure-arm/tree/master/Azure-Common-Deployments/v1/1fw_3nic_avset

Since are focusing on a single VPC model in this case, you can look to the AWS single VPC deployment guide to understand what steps are being taken end to end.
https://www.paloaltonetworks.com/resources/guides/aws-deployment-guide-single-resource

The purpose of thie repo will be to:
1. Capture a config file that can be imported to do the bulk of configuration
2. Lay out what manual steps will need to be taken in order to wrap things up

*****Firewall config file: https://github.com/chairforce2/PANOS-Azure-Config-GP  *******
-this config is on PAN-OS 9.1.4
base image from Azure/AWS may not be the same, I don't know if that means this config will cause it to download and install or what, honestly. haven't tried yet

So after the deploy referenced above is complete, license the firewall (if BYOL), wait for UI restart, import the firewall config, make the changes below (ESPECIALLY FOR THE ADMIN INTERFACE SO YOU CAN LOGIN, then commit.

2. Configure the Management Interface for secure use in accordance with: https://docs.paloaltonetworks.com/pan-os/9-1/pan-os-admin/getting-started/best-practices-for-securing-administrative-access.html

a. now much of this can be handled in the setup process or embedded in the config file, but items you should consider carrying out yourself:

-limitting source IPs that can access the management interface (this can be done in the mgmt interface config on PAN-OS, or through the security groups of the cloud provider. however I will not include this in the configuration file as it is going to change from deployment to deployment and at the moment of writing this I don't have time to figure out how to automate that as part of the setup process with Terraform for example.

-setting up MFA with a SAML Provider or other resource, if desired. Ex: okta saml for admin login: https://saml-doc.okta.com/SAML_Docs/How-to-Configure-SAML-2.0-for-Palo-Alto-Networks-Admin-UI.html

-consider if you think it is worth adding certs as an additional layer of authentication for the admin interface

-configure the firewall admin interface for HTTPS by generating certs. If you already have a PKI strategy figured out, phenomenal. If not, I have created some notes here on the matter: https://github.com/chairforce2/panos-and-certificates/blob/main/README.md


3. Manipulate the routes on the virtual router
- you should still have the static 0.0.0.0/0 route our the untrust interface, but you'll have to create your own static route to the trust interface on your dfaut VR
-for the static route to the internet, make sure you have the right setting for next hop / it actually points to a gateway for the subnet/vpc/vnet
- this will vary depending on the cloud provider
- for example, the last time I was trying to deploy on azure I kept having a terrible time because it just seemed like the untrust interface and the internal interface never got a default gateway. I ended up just statically configuring a route in the VR pointing 0.0.0.0/0 to the first IP of my untrust interface's subnet and suddely I could reach the internet.

4. Depending on your Cloud provider you may have to either NAT things on the internal subnet going through the firewall, or just change the route table of the untrust subnet so every packet bound for an up in the rust subnet is sent to the untrust interface of the firewall, which then looks it up in its table and sends it on. For the purposes of this configuration, this is not in scope as all I care about is establishing GP for testing resources on the internet.

5. You will need to deploy certs for the GP portal/gateway as well. These can be signed by the same root CA you already created on your device, but now we are just going to have them be mapped to the IP/DNS Name for the untrust interface GP is deployed on, and they will instead serve the purpose of a VPN Server instead of for HTTPS. See https://github.com/chairforce2/panos-and-certificates/blob/main/README.md


6. you may need to check and/or edit the NAT policy on the firewall itself (Dynamic IP and port)




Ideas for future improvement:
1. using bootstrapping to set up BYOL licensing and more
2. converting this overall guide to an editable terraform/ansible template to take in variable such as home IP / office IP to whitelist, etc
3.

