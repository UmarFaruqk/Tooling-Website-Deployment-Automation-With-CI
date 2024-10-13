## Tooling-Website-Deployment-Automation-With-CI
In previous Project, we introduced *horizontal scalability* concept, which allow us to add new Web Servers to our Tooling Website and you have successfully deployed a set up with 2 Web Servers and also a Load Balancer to distribute traffic between them. If it is just two or three servers - it is not a big deal to configure them manually. Imagine that you would need to repeat the same task over and over again adding dozens or even hundreds of servers.
DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is Automation of routine tasks.
In this project we are going to start automating part of our routine tasks with a free and open source automation server - Jenkins. It is one of the mostl popular CI/CD tools.
Acording to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.
In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub *https://github.com/<yourname>/tooling* will be automatically be updated to the Tooling Website.

# Step-By-Step Process
1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
2. Install JDK (since Jenkins is a Java-based application) but first update the terminal bu running *sudo apt update* then run *sudo apt install default-jdk-headless* ![reference image](/Pictures/pic1.PNG)
3. Then run  following commands *sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null*
4. Then this *sudo apt-get update* and this *sudo apt-get install jenkins*
5. Making sure Jenkins is up and running *sudo systemctl status jenkins* ![reference image](/Pictures/pic2.PNG)
6. By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound Rule in your EC2 Security Group ![reference image](/Pictures/pic3.PNG)

# Perform initial Jenkins setup
1. From your browser access *http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080* where you will be prompted to provide a default admin password ![reference image](/Pictures/pic4.PNG)
2. Retrieve it from your server using  *sudo cat /var/lib/jenkins/secrets/initialAdminPassword*
3. Then you will be asked which plugings to install - choose suggested plugins. ![reference image](/Pictures/pic5.PNG)
4. Once plugins installation is done - create an admin user and you will get your Jenkins server address. The installation is completed! ![reference image](/Pictures/pic6.PNG)

# Configure Jenkins to retrieve source codes from GitHub using Webhooks
In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will will be triggered by GitHub webhooks and will execute a 'build' task to retrieve codes from GitHub and store it locally on Jenkins server.
1. Enable webhooks in your GitHub under the project you want to connect to jenkins then click on the  settings ![reference image](/Pictures/pic7.PNG) there you will insert your jenkins public IP Address with it port number, the content type should be app/json the remaining setting should remain like that.
2. If you do everything you should see this ![reference image](/Pictures/pic10.PNG)
3.  Go to Jenkins web console, click "New Item" and create a "Freestyle project" ![reference image](/Pictures/pic8.PNG)
4.  To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself. In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository. ![reference image](/Pictures/pic9.PNG)
5.  Save the configuration and let us try to run the build. For now we can only do it manually. Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under #1 ![reference image](/Pictures/pic11.PNG)
6.  You can open the build and check in "Console Output" if it has run successfully and if you have  congratulations! You have just made your very first Jenkins build!
7.  Click "Configure" your job/project and add these two configurations: Configure triggering the job from GitHub webhook ![reference image](/Pictures/pic12.PNG) and Configure "Post-build Actions" to archive all the files - files resulted from a build are called "artifacts" ![reference image](/Pictures/pic13.PNG)
8. Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch (Depends on where file is saved mind is on the main). 
9.  Update your terminal this step is not neccessary ![reference image](/Pictures/pic14.PNG)
10. Refresh your browser and You will see that a new build has been launched automatically (by webhook) and you can see its results - artifacts, saved on Jenkins server. ![reference image](/Pictures/pic15.PNG)
11. You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as 'push' because the changes are being 'pushed' and files transfer is initiated by GitHub)
12. By default, the artifacts are stored on Jenkins server locally run this command to check ![reference image](/Pictures/pic16.PNG)

# Configure Jenkins to copy files to NFS server via SSH
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to */mnt/apps* directory.
Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".
1. On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item. Then On "Available" tab search for "Publish Over SSH" plugin and install it ![reference image](/Pictures/pic17.PNG)
2. Configure the job/project to copy artifacts over to NFS server. On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.
3. Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:
1. Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty) ![reference image](/Pictures/pic18.PNG)
2. Arbitrary name
3. Hostname - can be private IP address of your NFS server
4. Username - ec2-user (since NFS server is based on EC2 with RHEL 8)
5. Remote directory - /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server ![reference image](/Pictures/pic19.PNG)
6. Test the configuration and make sure the connection returns Success then save. Remember, that TCP port 22 on NFS server must be open to receive SSH connections. ![reference image](/Pictures/pic20.PNG)
7. open your Jenkins job/project configuration page and add another one "Post-build Action" ![reference image](/Pictures/pic21.PNG)
8. Configure it to send all files produced by the build into our previouslys define remote directory. In our case we want to copy all files and directories - so we use **. If you want to apply some particular pattern to define which files to send - use this syntax. ![reference image](/Pictures/pic21.PNG)
9. Save this configuration and go ahead, change something in *README.MD* file in your GitHub Tooling repository. 
10. Webhook will trigger a new job and in the "Console Output" of the job you will find something like this ![reference image](/Pictures/pic22.PNG)
11. To make sure that the files in */mnt/apps* have been udated - connect via SSH/Putty to your NFS server and check README.MD file. Run *cat /mnt/apps/README.md* 
If you see the changes you had previously made in your GitHub - the job works as expected. ![reference image](/Pictures/pic23.PNG)

**Congratulations!**
