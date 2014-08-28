## Setting up the Tower Environment
---
This is for demonstration purposes only.  Security group rules presented here are ***not*** recommended for production use.  Please adapt the security group rules to meet your organizational needs.

---


### VPC, Subnets, and keypairs

#### VPC

Most folks are using VPCs in their AWS environments these days, if not, we'd recommend you create one for the purposes of this demo.  Either way make note of the VPC ID, as we'll use it later.

#### VPC Subnets

Create or use existing subnets in the above VPC.  These will be used for your AutoScale groups.  The Subnets should be able to route to the Internet. Make note of Subnet IDs.

#### EC2 Keypair
Create or use an existing ec2 keypair. Tower will use this to ssh into your instances.



### Creating Security Groups for Tower

In the EC2 console, we need to create 3 security groups :
* one for the instances that will be using Tower's call back feature
* one for the Tower server itself that allows you to SSH to Tower and use Tower's web interface.
* one for the instances that Tower will SSH into.


####Tower Callback Client Security Group

This is a group without any rules.  It will be assigned to the Launch Configuration of the instances that require callback access.  Make a note of the group ID after it is created.

![image](images/tower_callback_client_security_group.png)  


####Tower Server Security Group
Create a custom security group for your Tower instance to live in.  I give myself access to SSH, HTTPS, and 8080, and I give members of the security group I created above access to HTTPS as well.  Make a note of the group ID after it is created.

![image](images/tower_security_group.png)  

The security group should allow you to SSH into the Tower instance, have HTTPS access to the Tower instances, and should be reachable by HTTPS by your managed instances for the callback feature to work.  

#### Tower Client Security Group
This group is assigned to the instances that are managed by Tower.  It allows Tower to SSH into the instances.

![image](images/tower_client_group.png)



### Launching the Tower instance

Launch a moderately sized EC2 instance -- 4GB or more memory.  Make sure it is a being launched into the VPC referenced above and is member of the Tower Server Security Group you created.  

The next step is to install Ansible and Ansible Tower onto your newly launched instance.  You can find [installation instructions here](http://releases.ansible.com/ansible-tower/docs/tower_user_guide-latest.pdf).  Once you've setup and logged into your Tower instance, proceed to the next step.

Extra bonus points to the person that creates an Ansible playbook to provision the above!




## Basic Tower Configuration
	

Now open your browser and go to https://myinstance.compute-1.amazonaws.com and login to Tower with **admin**** as the user and **password**** as the passsword.  You'll be prompted to paste your license in there.

## Create an Organization


First, you'll want to create an Organization, typically named after the Organization you're working for.  In this case, we'll call our organization MyOrg.

![image](images/create_org.png)

We don't need to create any additional users or teams for the purposes of this demo.

## Configuring Tower Credentials

The next step is to configure some credentials for Tower.  Click on the Credentials tab and then click on the + icon to add a new set of credentials. 

### AWS Credentials:

![image](images/add_aws_creds.png)

### SSH Credentials

Now we'll configure a set of credentials that Tower will use to SSH to the instances.  These must be the ec2 keypair you created earlier.  

![image](images/add_ssh_cred.png)

## Configuring Tower to synchronize EC2 inventory

### Create the Inventory

Click on the Inventory tab and click on the + icon to add a new inventory and fill out that form as below:

![image](images/create_inv.png)



### Create the Inventory Group
Click on the newly created Inventory and click the + symbol to add a new group.  Fill out the form as below:

![image](images/add_group.png)

Click the Source tab and fill out the form as below.  Make sure to set the Source Variables:

    ---
    vpc_destination_variable: private_ip_address

![image](images/set_group_src.png)

Now click on the **Start sync process** icon.  ![image](images/start_sync.png)

