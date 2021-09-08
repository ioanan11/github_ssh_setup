**How to do SSH setup between LocalHost and GitHub**

![alt text](https://github.com/ioanan11/github_ssh_setup/blob/main/SRE_GitHub_ssh_setup/Screenshot%202021-09-08%20101922.png)

From localhost (master) on git bash use the command:

	ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

For "enter passphrase" just press enter. 

For "enter same passphrase" press enter again.  
	
Then use command:

	cat id_rsa.pub

And paste the key you get into git hub when you create an SSH key. You can create an SSH key from settings on github. 

Then, when creating a new repo use SSH instead of HTTP.
