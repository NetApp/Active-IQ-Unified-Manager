# Automated AIQUM, adding an ONTAP cluster to AIQUM and claiming AIQUM into Cisco Intersight using Ansible

This repository contains Ansible roles and playbooks for end-to-end AIQUM deployment, adding ONTAP to AIQUM and claiming AIQUM into Cisco Intersight

The deployment automation is based on the following roles:

``` yaml
aiqumsetup
intersightclaim
```

# Environment Validated
``` yaml
ONTAP 9.7p1+
AIQUM 9.8P1+
ESXi and vCenter 6.7+
UCSM 4.1.2+
CentOS 8
Ubuntu 20.04
```

# Prerequisites
1. It is assumed that the physical rack and stack and power-on, initialization of ONTAP OS, setup of node management IPs and initial ONTAP cluster with IP is completed.
2. It is assumed a Cisco Intersight account is setup and configured to allow devices to be added to the account. See below section for how to obtain a Cisco Intersight key and code.
3. It is assumed vCenter is in working order to deploy an OVA.
4. The user should have an Ansible Control sever that has network reachability to the ONTAP cluster and internet access to pull this repository from GitHub. Refer to https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html for guidance on setting up and Ansible Control server.
5. The Ansible control server should have the following collections, libraries and modules installed:

``` yaml
ansible-galaxy collection install cisco.intersight

ansible-galaxy collection install community.vmware

ansible-galaxy collection install netapp.ontap

```

6. Create a .sh file to install any additional libraries and modules or run the commands individually
``` yaml
vi setup.sh
```

7a. Paste the below content into the file ( For CentOS 8 )
``` yaml
#!/bin/bash
echo "Installing Python ------>"
sudo yum -y install python3 >/dev/null

echo "Installing Python Pip ------>"
sudo yum -y install python3-pip >/dev/null

echo "Installing git ------>"
sudo yum -y install git >/dev/null

echo "Installing PyVmomi ------>"
pip3 install --user  PyVmomi

echo "Installing pexpect ------>"
pip3 install --user pexpect

echo "change owner and group of /etc/ansible ------>"
sudo chown -R username.usergroup /etc/ansible/

echo "creating /usr/share/ansible_modules directory for AIQUM modules ------>"
sudo mkdir /usr/share/ansible_modules

echo "Downloading AIQUM module ------>"
wget https://raw.githubusercontent.com/NetApp/Ansible-with-Active-IQ-Unified-Manager/master/aiqum_modules/aiqum_clusters.py

echo "copying aiqum_clusters.py file to /usr/share/ansible_modules/ ------>"
sudo cp aiqum_clusters.py /usr/share/ansible_modules/
```
7b. Paste the below content into the file ( For Ubuntu 20.04 )
``` yaml
#!/bin/bash
echo "Installing Python3-pip ------>"
sudo apt-get -y install python3-pip  >/dev/null

echo "Installing PyVmomi ------>"
pip3 install --user  PyVmomi

echo "Installing expect ------>"
pip3 install --user pexpect

echo "echo Changing owner and group of /etc/ansible ------>"
sudo chown -R username.usergroup /etc/ansible/

echo "echo creating /usr/share/ansible_modules directory for AIQUM modules ------>"
sudo mkdir /usr/share/ansible_modules/

echo "Downloading AIQUM module ------>"
wget https://raw.githubusercontent.com/NetApp/Ansible-with-Active-IQ-Unified-Manager/master/aiqum_modules/aiqum_clusters.py

echo "copying aiqum_clusters.py file to /usr/share/ansible_modules/ ------>"
sudo cp aiqum_clusters.py /usr/share/ansible_modules/
```
Notes: \
Change /etc/ansible to the directory your AIQUM playbook will be kept \
Change username.usergroup to the user that will be setting up the ansible playbook 

8. Make the file executable
``` yaml
chmod +x setup.sh
```
9. Run the script
``` yaml
sudo ./setup.sh
```

10. Add the following line into your site ansible.cgf file
``` yaml
library        = /usr/share/ansible_modules/
```

# Getting Started

1. From the Ansible control server change directory to /etc/ansible and download a ZIP version of this repository or clone it using the below command.

``` yaml
git clone https://github.com/NetApp-Automation/NetApp-AIQUM.git
```

2. There is one variable file under the vars folder 'aiqum_main.yml ' that needs to be filled out with environment specific parameters prior to executing the playbook. 

NOTE: The format of the variable file needs to be maintained as it is, any changes to the structure of the file may lead to failure in execution of the playbook. 

NOTE: Sample values are pre-populated against some variables in order to provide the user additional clarity on how the variable needs to be filled out. Please replace the sample values with your environment specific information

3. Update the credentials for the ONTAP cluster, the AIQUM VM, Cisco Intersight and the vCenter. Appropriate files are located under group_vars folder. \
Sample:
``` yaml
# Credentials for connecting to the ONTAP Cluster
ontap_cluster_username: admin
ontap_cluster_password: password

# Management IP Addresses of the ONTAP cluster
ontap_cluster_address: 192.168.10.10
```

4. Executing the Playbook A playbook by name 'aiqum.yml' is available at the root of this repository. It calls all the required roles to complete the installation of AIQUM, adding a cluster to AIQUM and claiming AIQUM into Cisco Intersight. Execute the playbook from the Ansible Control server using the following command:

NOTE: Add -vvv to the below command to get highest verbose output to assist in troubleshooting. 

``` y
ansible-playbook aiqum.yml
```

If you would like to run a part of the deployment, you may use the appropriate tag that accompanies each task in the role and run the playbook in the below fashion.

To install AIQUM and add an ONTAP cluster use -t aiqum_setup
``` yaml
ansible-playbook aiqum.yml -t aiqum_setup
```

To claim an existing AIQUM instance into Intersight use -t intersight_claim

``` yaml
ansible-playbook aiqum.yml -t intersight_claim
```


# How to obtain Cisco Intersight Key and Code
Log into Cisco Intersight.\
Click on Gear then Settings.\
Click on API Keys.\
Click on Generate API KEY.\
Enter a Description.\
Click Generate.\
Copy API Key ID and use as value for “api_key_id” in group_vars/intersight file.\
Copy Secret Key and paste into a text file and use as path for “cisco_key” in group_vars/intersight file.


# Author Information
* Jason Walton (jason.walton@netapp.com) 
