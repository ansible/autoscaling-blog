# Provisioning an Autoscaling Infrastructure using Ansible Tower

## Introduction

Sure you've provisioned some instances into EC2 using Ansible.  And you probably also deployed your application there as well. But you probably have *not* provisioned an autoscaling EC2 infrastructure using Ansible Tower.  This article is designed to show you how to do just that.


## Requirements

Autoscaling is dependent on Ansible Tower's callback mechanism.  For those who aren't familiar, Tower is our commercial product.  It's free-for-life if your are using 10 servers or less.

In our case, we'll be using the Ansible Tower AMI available on the AWS Marketplace to speed up the process.

## Setting up the Tower Server

Create a custom security group for your Tower instance to live in:

![image](images/tower security group.png)

First launch the Tower AMI into the EC2 region of your choice. [Follow this link](http://www.ansible.com/tower) and click on the **Launch Tower in EC2** button.  You'll be greeted with a simple form to fill out.


After that, you'll be presented with a CLICK HERE link.  When you click that you'll be redirected to AWS.

For our testing purposes, we'll be launching the Ansible Tower (10 instances) license.  There's no additional charge to use this other than the standard EC2 operating charges.  Go ahead and click on that, and then on the next page, click on the yellow continue button.

On this page, we'll make sure that we're selecting the latest version of Tower available on AWS.  At the time of this writing, that version is 1.4.11.   Also choose your region.  For the purpose of this article we'll be using US East.  The default instance type for Tower is m1.large, so go ahead and leave it at that for now.

As far as the VPC is concerned, we'll be launching ours into our default VPC.  Choose an appropriate Subnet for your VPC also.

The default security group is going to allow SSH, HTTP, and HTTPS.  I'd recommend you restrict the IP addresses that are allowed to connect to these ports as well.

Finally, select the EC2 keypair to use to launch your instance, and click the Launch button at the top right of the screen.



## Gather some information 
 
We need to gather some bits of data from the AWS console to supply as variables to our playbooks.

In the AWS console, make note of the GroupID the security Group the Tower instance is using.  It should be something like ```Ansible Tower -10 instances--1-4-8-AutogenByAWSMP-```.

We'll also need to gather the VPC ID that we'll be using for the demo.  For the purposes of this demo, it should be the same VPC ID we used to launch the Tower instance.  

Hold on to this information for now, we'll user it later.

## Basic Tower Configuration

Find the public address of the Tower instance you just launched in the EC2 console.  We'll ssh to it find the randomally generated admin password for tower in the login message:

	ssh ubuntu@myinstance.compute-1.amazonaws.com
	Welcome to Ubuntu 12.04.4 LTS (GNU/Linux 3.2.0-60-virtual x86_64)
	
	  Welcome to AnsibleWorks AWX!
	
	  Log into the web interface here:
	
	    https://myinstance.compute-1.amazonaws.com/
	
	    Username: admin
	    Password: 2v9P7rmpfsHd
	
	  The documentation for AWX is available here:
	
	    http://www.ansibleworks.com/ansibleworks-awx/
	
	  For help, email support@ansibleworks.com.
	
	7 packages can be updated.
	3 updates are security updates.
	

Now open your browser and go to https://myinstance.compute-1.amazonaws.com and login to Tower with admin as the user and the password you found in the previous step.

Firstly, you'll want to create an Organization, typically named after the Organization you're working for.  In this case, we'll call our organization Ansible

We don't need to create any additional users or teams for the purposes of this demo.

## Configuring Tower Credentials

The next step is to configure some credentials for Tower.  Click on the Credentials tab and then click on the + icon to add a new set of credentials. 

The first thing we'll do is create some AWS Credentials:

| Property | Value |
|----------| ----- |
|Name| AWS Creds |
|Description| Credentials For AWS|
|Does this credential belong to a team or user?| User|
|User that owns this credential| admin|
|Type|AWS|
|Access Key|yourawsaccesskey|
|Secret Key|yourawsecretkey|

Click save when done.

Now we'll configure a set of credentials that Tower will use to SSH to the instances.  Click on the + icon to add a new set of credentials, and fill out the form as described below:

| Property | Value |
|----------| ----- |
|Name| EC2 SSH Key |
|Description| SSH Key Used to connect to EC2|
|Does this credential belong to a team or user?| User|
|User that owns this credential| admin|
|Type|Machine|
|SSH Username|ec2-user|
|SSH Private Key|pasteyoursshprivatekeyhere|
|Key Password|Paste Your keypassword here|
|Confirm Key Password|Paste Your keypassword here|


## Setting up Tower to pull playbooks from GitHub

I've already created a project on Github that covers what we need to do.  That project [can be found here](https://github.com/jsmartin/the_light).  To add that project to Tower, perform the following steps:  

Click the Projects Tab, and click the + icon to add a new project.  Fill in the form as described below:

| Property | Value |
|----------| ----- |
|Name| AutoScale Demo |
|Description| Demonstration of AutoScaling with Tower|
|Organization| YourOrg|
|SCM URL|https://github.com/jsmartin/the_light.git|

And click **Save**

## Configuring Tower to synchronize EC2 inventory

Click on the Inventory tab and click on the + icon to add a new inventory and fill out that form as below:


| Property | Value |
|----------| ----- |
|Name| Ansible Inventory |
|Description| Demonstration of AutoScaling with Tower|
|Organization| YourOrg|



Click on the newly created Inventory and click the + symbol to add a new group.  Fill out the form as below:


| Property | Value |
|----------| ----- |
|Name| EC2 |
|Description| EC2 machines|

Click the Source tab and fill out the form as below:

| Property | Value |
|----------| ----- |
|Source| Amazon EC2 |
|Cloud Credential| AWS Creds|
|Regions| US East|
|Update Options| Update on Launch|
|Source Variables| vpc_destination_variable: private_ip_address |

Click Save.

Now click on the "Start sync process".


## Creating the Configuration Template
This template will be used to configure the applications -- it is launched with cofnig.yml playbook.


| Property | Value |
|----------| ----- |
|Name| Configure Application |
|Description| Configures Application|
|Job Type | Run|
|Inventory| EC2|
|Project| AutoScale Demo|
|Playbook|config.yml|
|Machine Credential|EC2 SSH Key|

Click on the **Allow Callbacks** check box.  After doing so two additional textboxes  will appear: Callback URL and Host Config Key.

Click on the Magic Wand icon to the right of the Host Config Key text box.

A dialog will appear with the call back URL and Host Key.  Make note of the host configuration key and the job template ID (highligthed below).  These are unique to your Tower install, so please do not use the values below, they are only provided for reference.

Host callbacks are enabled for this template. The callback URL is: /api/v1/job_templates/```27```/callback/
The host configuration key is: ```c5ecde738fc822ec8429743e32895572```


## Creating the Provisioning Job Template

This job template will be used to spin up the EC2 infrastructure using the infra.yml playbook.  

Fill in the following information:

| Property | Value |
|----------| ----- |
|Name| Provision Infrastructure |
|Description| Provisions EC2 infrastructure|
|Job Type | Run|
|Inventory| Ansible Inventory|
|Project| AutoScale Demo|
|Playbook|infra.yml|
|Machine Credential|EC2 SSH Key|
|Cloud Credentials|AWS Creds|

In the Extra Variables section, paste in the following in, making sure you substitue the key_name, vpc_id, tower_group, tower_address, template_id, and host_config_key with the values we discovered earlier. If you're using a region other than us-east1, you'll have to substitue the proper AMI for a RHEL 6.5 instance as well.


	region: us-east-1
	app_name: myapp
	zones:
	- us-east-1c
	- us-east-1d
	tower_group: sg-bea1cddb
	vpc_id: vpc-7caf5119
	ami: ami-8d756fe4
	max_size: 4
	min_size: 2
	desired_capacity: 2
	key_name: jmartin
	instance_size: m1.small
	tower_address: 10.1.1.10
	template_id: "66"
	host_config_key: 1496e94617b2530a04a72bd30e676f8d

Assign the EC2 credentials we created earlier using the **Cloud Credentials** dialog.  The template will create the following items:

### EC2 Infrastructure 

| Item  | Description |
|-----------------------------|------------------------------------------------------------------------------------------|
| Admin Access Security Group | Who can access the application servers. |
| App Security Group | Self-referencing group that allows members of this group to access all ports of members. |
| RDS Instance | A PostgreSQL backed RDS instance for the app servers. |
| Load Balancer | A load balancer for the app servers. |
| Launch Configuration | A configuration used by the AutoScale group. |
| AutoScale Group | Defines the number of instances and the load balancer membership. |
| Scale Up & Down Policies | Policies that when alarmed will dictate change to occur on AutoScale group.  |
| Scale Up & Down Alarms | Alarms that will trigger their relative scale up and down policies. |




## Tying it all together

Now that we have the two Project Templates and we've enabled callbacks for the configuration template, we have to find a way for Tower to configure instances spun up in the AutoScale group on demand.


A Launch Configuration defines how instances in an AutoScale group will launch -- things like instance size, key names, AMI, security groups, and most importantly in this case, user data.  EC2 user data can be used as a script to run upon instance launch.  At first you might thing, great, let's just our embed our callback script to Tower in the user data.  The problem is that Launch Configurations *can't be modified* after creation.  So if you need to use a different job template id, or a different host configuration key, you'd need to delete the AutoScale group and create it again.  So we don't have to do that in the future, it may be a good idea to abstract that a bit.  Here's how the workflow would work:

Create a script that performs the callback to Tower.  Place that script in a S3 bucket.  You should typically have one callback script per AutoScale group.  In this case we'll create a callback script called ```myapp_autoscale.sh```, the contents would like something like this:

	#!/bin/bash
	TOWER_ADDRES=10.0.0.1
	HOST_CONFIG_KEY=1496e94617b2530a04a72bd30e676f8d
	TEMPLATE_ID=66
    
	
	retry_attempts=10
	attempt=0
	while [[ $attempt -lt $retry_attempts ]]
	do
	  status_code=`curl -s -i --data "host_config_key=$HOST_CONFIG_KEY" https://$TOWER_ADDRESS/api/v1/job_templates/$TEMPLATE_ID/callback/ | head -n 1 | awk '{print $2}'`
	  if [[ $status_code == 202 ]]
	    then
	    exit 0
	  fi
	  attempt=$(( attempt + 1 ))
	  echo "${status_code} received... retrying in 1 minute. (Attempt ${attempt})"
	  sleep 60
	done
	exit 1


Now if you take a look at the Launch Configuration's userdata in ```infra.yml```, you'll see that we fetch this script and run it.

	- name: create launch config
	  local_action:
	    module: ec2_lc
	    name: "{{ app_name }}"
	    image_id: "{{ ami }}"
	    key_name: "{{ key_name }}"
	    region: "{{ region }}"
	    security_groups: "{{ app_security_group.group_id }},{{ admin_access.group_id }}"
	    instance_type: "{{ instance_size }}"
	    user_data: |
	               #! /bin/bash 
	               s3cmd get s3://mycompany/myapp_autoscale.sh /root/myapp_autoscale.sh
	               chmod 755 /root/autoscale.sh
	               /root/autoscale.sh
	  tags: launch_config


Now instead of having to delete and recreate your launch configuration when some part of your callback script changes, you can just change the script you have stored in S3.

Make sure that your AMI supports using UserData as scripts.  At the time of this writing, The Amazon Linux, Ubuntu, Debian, and RHEL 6.x AMIs support this feature.  The official CentOS AMIs do not.


## Running the Playbooks

First we'll need to provision the AWS infrastructure, so find the **provisiong infrastructure** template and click the launch icon to kick it off.  
The following happens automatically:

1. The EC2 infrastructure playbook is run and all EC2 components described in the EC2 infrastructure table are created.
2. The AutoScale Group is created and the initial 2 instances are launched
3. The instances will both download the script you placed in S3 and run it
4. That script phones home to Tower and requests Tower to configure the server with the template ID (in this case, the Configure Application template) that is in the URL.
5. Tower checks to make sure that the instances are part of the inventory specified in the job template and also validates the host configuration key.
6. Tower then configures the instances with the Configure Application template.
7. The instances are configured and running your application










