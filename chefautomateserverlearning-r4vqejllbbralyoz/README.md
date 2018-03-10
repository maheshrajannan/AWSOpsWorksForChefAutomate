Overview
========

This zip file starts you out with a simple example of a Chef repository that is preconfigured to work with an AWS OpsWorks Chef Automate server.
In this repository, you store cookbooks, roles, configuration files, and other artifacts for managing systems with Chef.
It is recommended that you store this repository in a version control system such as Git, and treat it like source code.

Repository Directories
======================

This repository contains several directories. Each directory contains a README file that describes the directory's purpose,
and how to use it for managing your systems with Chef.

* `cookbooks/` - Cookbooks that you download or create.
* `roles/` - Stores roles in .rb or .json in the repository.
* `environments/` - Stores environments in .rb or .json in the repository.

Configuration
=============

`.chef` is a hidden directory that contains a knife configuration file (knife.rb) and the secret authentication key (private.pem).

* .chef/knife.rb
* .chef/private.pem

The `knife.rb` file is configured so that knife operations will run against the AWS OpsWorks managed Chef Automate server.

To get started, download and install the [Chef DK](https://downloads.chef.io/chef-dk).
After installation, use the Chef `knife` utility to manage the Chef Automate server.
For more information, see the [Chef documentation for knife](https://docs.chef.io/knife.html).

Quick-start Example
===================

Use Berkshelf to get cookbooks from a remote source and install an Apache Web Server
------------------------------------------------------------------------------------

Berkshelf is a tool to help you manage cookbooks and their dependencies. It downloads a specified cookbook into
local storage, also called the Berkshelf. You can specify which cookbooks and versions to use with your Chef server
and upload them. This Starter Kit contains a file, named Berksfile, that can contain your cookbooks.
Also included is the `chef-client` cookbook that configures the Chef client agent software on each node that you connect to your Chef server.
To learn more about this cookbook, see [Chef Client Cookbook](https://supermarket.chef.io/cookbooks/chef-client) in the Chef Supermarket.

1. Using a text editor, append another cookbook to your Berksfile to install software; for example, to install
the Apache web server application. Your Berksfile should resemble the following.
```
source 'https://supermarket.chef.io'
cookbook 'chef-client'
cookbook 'apache2'
```

2. Download and install the cookbooks on your local computer.
```
berks vendor
```

3. Upload the cookbook. You'll need to specify the CA-signed certificate that is included with the Starter Kit.
On Linux:
```
SSL_CERT_FILE='.chef/ca_certs/opsworks-cm-ca-2016-root.pem' berks upload
```
On Windows, run a Chef DK PowerShell command:
```
$env:SSL_CERT_FILE="ca_certs\opsworks-cm-ca-2016-root.pem"; berks upload
Remove-Item Env:\SSL_CERT_FILE
```

4. Verify the installation of the cookbook by showing a list of cookbooks that are currently available on the
Chef Automate server.
```
knife cookbook list
```

Adding Nodes Automatically in AWS OpsWorks for Chef Automate with the prepared userdata script
----------------------------------------------------------------------------------------------

To connect your first node to the AWS OpsWorks for Chef Automate server, use the **userdata.sh** script that is included in this Starter Kit. It uses the AWS OpsWorks AssociateNode API to connect a node to your newly created server.

To connect a node to your server, create an AWS Identity and Access Management (IAM) role to use as your EC2 instance profile. The following AWS CLI command launches an AWS CloudFormation stack that creates an IAM role for you named _myOpsWorksChefAutomateInstanceprofile_.

```
aws cloudformation --region <region> create-stack \
--stack-name myOpsWorksChefAutomateInstanceprofile \
--template-url https://s3.amazonaws.com/opsworks-cm-us-east-1-prod-default-assets/misc/opsworks-cm-nodes-roles.yaml \
--capabilities CAPABILITY_IAM
```
The userdata is ready to use. Edit the RUN_LIST setting to define which roles, cookbooks, or recipes you want to run on your new instance and upload to your server.

For example, add "recipe[chef-client],recipe[apache2]" to run the chef-client agent on your node every 30 minutes, and configure an Apache2 web server. These cookbooks must be uploaded to the server.


### Create your first node

The easiest way to create a new node is to use the [Amazon EC2 Launch Wizard](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launching-instance.html). Choose an Amazon Linux AMI. In Step 3: "Configure Instance Details", select _myOpsWorksChefAutomateInstanceprofile_ as your IAM role. In the "Advanced Details" section, upload the **userdata.sh** script.

You don't have to change anything for Step 4 and 5. Proceed to Step 6.

In Step 6, choose the appropriate rules to open ports. For example, open port numbers 443 and 80 for a web server.

Choose Review, and then choose Launch to proceed to the final Step 7. When your new node starts, it executes the RUN_LIST section of your userdata script.

For more information, see the [AWS OpsWorks for Chef Automate user guide](https://docs.aws.amazon.com/opsworks/latest/userguide/opscm-unattend-assoc.html).




Alternative: Attach an Amazon EC2 instance to the newly-launched Chef Automate server using knife bootstrap
-----------------------------------------------------------------------------------------------------------

1. Bootstrap a new Amazon EC2 instance.
```
knife bootstrap <IP address of the Amazon EC2 instance>  -N <instance name> -x <user name> -i <path to your ssh key file> --sudo --run-list "recipe[chef-client],recipe[apache2]"
```

2. Show the new node.
```
knife client show <instance name>
knife node show <instance name>
```


Learn more about using Chef Automate to configure your systems on the Learn Chef website
----------------------------------------------------------------------------------------
Visit the [Learn Chef tutorial site](https://learn.chef.io/tutorials/manage-a-node/opsworks/) to learn more about using AWS Opsworks for Chef Automate.
