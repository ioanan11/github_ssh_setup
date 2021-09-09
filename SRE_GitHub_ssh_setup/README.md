# How to do SSH setup between LocalHost and GitHub

![alt text](https://github.com/ioanan11/github_ssh_setup/blob/main/SRE_GitHub_ssh_setup/Screenshot%202021-09-08%20101922.png)

From localhost (master) on git bash use the command:

	ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

For "enter passphrase" just press enter. 

For "enter same passphrase" press enter again.  
	
Then use command:

	cat id_rsa.pub

And paste the key you get into git hub when you create an SSH key. You can create an SSH key from settings on github. 

Then, when creating a new repo use SSH instead of HTTP.

# CI-CD with Jenkins Task

![alt text](https://github.com/ioanan11/github_ssh_setup/blob/main/SRE_GitHub_ssh_setup/Screenshot%202021-09-09%20092509.png)


## 1. We need a Public/Private Key pair

Follow the steps above

## 2. We need a Webhook on GitHub

- on the repo on GitHub select Settings

-Webhooks -> Add Webhook

-Payload URL: 

	http://jenkins_ip:8080/github-webhook/

-Content Type: json

-Select: Just the push event


## 3. Create Jenkins Jobs

-we need 3 jobs: one for continuous integration, one for merging and one for deployment.



### How to create a Jenkins job

Select "New Item" on Jenkins Dashboard

Enter a name

Select "Freestyle Project"	



**JOB 1: Continous Integration** 

(named it ioana_ci)

General

-Discard old builds and keep max number of builts to 3
		
-GitHub project add the HTTP URL of the repo

	
Office 365 Connector

-Restrict where this project can be run

	sparta-ubuntu-node


Source Code Management
			
-Select Git

-Repositories:

Repository URL: insert SSH URL from GitHub repo

Credentials: Add Jenkins and select SSH Username with private key as Kind. Then set a description and enter key manually from gitbash (Make sure you copy paste the beggining and end of the key!)

Branches to build: */dev


Build Triggers

-Select GitHub hook trigger for GITScm polling


Build Environment

-Select Provide Node & npm bin/ folder to path


Build

-Add build step -> Execute shell

	cd app
	npm install
	npm start


Post-build actions

-Add post-build action -> Build other projects

-Merge job: Project name (ioana_merge)

-Select Trigger only if built is stable


**JOB 2: Merge**

(named it ioana_merge)

General

-Discard old builds and keep max number of builts to 3

-GitHub project add the HTTP URL of the repo


Office 365 Connector

-Restrict where this project can be run

        sparta-ubuntu-node


Source Code Management

-Select Git

-Repositories:

Repository URL: insert SSH URL from GitHub repo

Credentials: Add Jenkins and select SSH Username with private key as K>

Branches to build: */dev


Build Environment

-Provide Node & npm bin/folder to path


Post-build actions

-Add post build action > Git Publisher

-Push only if build succeeds

-In branches: 
	branch to push > main
	target remote name: origin

-Add another post-build action > Build other projects

-Write job name that will deploy (ioana_cde)

-Select Trigger only if build is stable

Note: Make sure Git Publisher is above Build other projects




**JOB 3: Continous Deployment**

(named it ioana_cde)


General

-Discard old builds and keep max number of builts to 3

-GitHub project add the HTTP URL of the repo


Office 365 Connector

-Restrict where this project can be run

        sparta-ubuntu-node


Source Code Management

-Select Git

-Repositories:

Repository URL: insert SSH URL from GitHub repo

Credentials: Add Jenkins and select SSH Username with private key as K>

Branches to build: */dev


Build Environment

-Provide node & npm bin/ folder to path

-Select SSH agent 

	Specific credentials -> SSH Key for the EC2 instance


Build

-Add build step -> Execute Shell

	ssh -A -o "StrictHostKeyChecking=no" ubuntu@ec2-<IP>.eu-west-1.compute.amazonaws.com << EOF	

	rm -rf CICD_Jenkins

	git clone https://github.com/andujiuba/CICD_Jenkins.git

	cd CICD_Jenkins

	export DB_HOST=mongodb://52.215.204.66:27017/posts
	# navigate to app folder

	cd app
	npm install
	node seeds/seed.js

	#pm2 kill
	#pm2 start app.js

	nohup node app.js > /dev/null 2>&1 &

# To debug ssh into your ec2 and run the above commands

EOF

## 4. AWS EC2 Instance

Copy EC2 Instance from previously created AMIs (app EC2)

Connect the instance to your VPC and public subnet

Make sure the settings in the selected Security Group are:
-SSH (22), my IP
-SSH (22), Jenkins IP/32
-HTTP(80), 0.0.0.0/0
-Custom TCP (3000), 0.0.0.0/0

Key pair SSH option

Make sure NACL is configured to allow SSH(22) with Jenkins IP/32

Note: Jenkins IP modifies whenever the server is rebooted

## 5. Triggering everything to automate

On dev branch we can make any change which when pushed to dev will trigger CI job. Then, if tests are successful, the merge job will be triggered and will merge dev to main on GitHub. Then, finally, app will be running on public ip on port 3000. 
