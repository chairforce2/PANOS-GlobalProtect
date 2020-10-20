# PANOS-GlobalProtect
Configuration for quickly configuring (particularly cloud hosted) VM-series to act as a GlobalProtect Portal and Gateway

This process assumes you have already been able to stand up a VM-Series firewall with a minimum of 3 NICs (1x mgmt, 2 x dataplane) with the Untrust address as eth1/1 and it being reachable from the internet (either by public IP allocated or via DNS). If you have not yet dont this I suggest you check out the reference archicture guides from Palo Alto Networks at https://www.paloaltonetworks.com/resources/reference-architectures

If you would like to automate the deployment of these reference archtictures you can visit https://github.com/PaloAltoNetworks/reference_architecture_automation
Or search through one of the Palo Alto Networks affiliated Github pages for a repo which matches your needs:

https://github.com/PaloAltoNetworks
https://github.com/wwce

Since are focusing on a single VPC model in this case, you can look to the AWS single VPC deployment guide to understand what steps are being taken end to end.
https://www.paloaltonetworks.com/resources/guides/aws-deployment-guide-single-resource

The purpose of thie repo will be to:
1. Capture a config file that can be imported to do the bulk of configuration
2. Lay out what manual steps will need to be taken in order to wrap things up

So after the deploy referenced above is complete, you will import the FW config in this repo, and then proceed with the steps below:

1. License the VM-Series firewall (if using a BYOL license)
2. Configure the Management Interface for secure use in accordance with: https://docs.paloaltonetworks.com/pan-os/9-1/pan-os-admin/getting-started/best-practices-for-securing-administrative-access.html

a. now much of this can be handled in the setup process or embedded in the config file, but items you should consider carrying out yourself:

-limitting source IPs that can access the management interface (this can be done in the mgmt interface config on PAN-OS, or through the security groups of the cloud provider. however I will not include this in the configuration file as it is going to change from deployment to deployment and at the moment of writing this I don't have time to figure out how to automate that as part of the setup process with Terraform for example.

-setting up MFA with a SAML Provider or other resource, if desired. Ex: okta saml for admin login: https://saml-doc.okta.com/SAML_Docs/How-to-Configure-SAML-2.0-for-Palo-Alto-Networks-Admin-UI.html

-consider if you think it is worth adding certs as an additional layer of authentication for the admin interface

-configure the firewall admin interface for HTTPS by generating certs. If you already have a PKI strategy figured out, phenomenal. If not, I have created some notes here on the matter: https://github.com/chairforce2/panos-and-certificates/blob/main/README.md







Ideas for future improvement:
1. using bootstrapping to set up BYOL licensing and more
2. converting this overall guide to an editable terraform/ansible template to take in variable such as home IP / office IP to whitelist, etc
3.
