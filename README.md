# How to Deploy

## Prerequisites
In order to deploy this 3 tier web app locally you will need the following tools installed
 - Docker 
 - Docker-compose
 - Git

You will also need
 - A [github account](https://github.com/)
 - Public and private ssh keys

If you using a mac
 - Go to the following [link](https://store.docker.com/editions/community/docker-ce-desktop-mac) to install docker and docker compose
 - Go to the following [link](https://git-scm.com/download/mac) to install Git for Mac
 - If you do not have a public and private ssh key on your computer, you can install them with the following and accept the defaults with no passphrase
 ```
 $ ssh-keygen
 ```



 In order to deploy this web app to the cloud. Collect the following information from your hub admin
 - URL of the oracle cloud infrastructure console
 - Tenancy name
 - User name
 - Password 

## Abstract
This repo contains the code for the backend service that is built using Nodejs and the frontend service that uses React. It also contains the necessary files to containerize both apps and a docker compose file that will simply deploy both apps and a MongoDB instance with a single command,  ```docker-compose up```. This is a simple TODO list that has a bug, which once is fixed can be deployed to "Production". You wil be tasked with provisioning the infrastructure to Oracle Cloud Infrastructure and deploying this 3 tier web app. 


 ## Step 1 Forking
 1. Go to [https://github.com/schmidtp0740/react-todolist](https://github.com/schmidtp0740/react-todolist)
 
 2. Fork the repo, this will clone this repo under your name. From now on work on the repo under your name
 3. Click **Clone or Download**, to clone the repo under your name
 4. Copy the url
 5. Clone the repo to your desktop
    - open up your terminal
    - change into the directory that you want to clone the repo in and clone the repo

 ```
 $ cd /User/<your computer name>/
 $ git clone <url you copied>
 ```
 6. Change into the directory
 ```
 $ cd /react-todolist
 ```

 ## Run the Web App Locally
 1. Build the docker images
 ```
 $ docker-compose build
 ```
 2. Docker-compose will output each step that is happening and some may be in red which is okay, wait until you recieve the prompt again
    - This process may take several minutes

 3. Use Docker-compose to spin up 3 containers. One that will run your react-frontend code, another that runs your backend code, and the last one that runs an instance of MongoDB
 ```
 $ docker-compose up
 ```


 4. Go to your browser and visit [http://localhost](http://localhost)
 5. Try to add a new task
 6. You will notice it will refuse to add a new task because there is a bug
 7. Open react-todolist/backend/app.js in your favorite editor
 8. Uncomment lines 12 - 19(remove the backslashes)
 ```
 // mongoose.connect('mongodb://mongodb')
 //     .then(() => {
 //       console.log('Backend Started');
 //     })
 //     .catch(err => {
 //         console.error('Backend error:', err.stack);
 //         process.exit(1);
 //     });
 ```
 9. Save the file
 10. You will notice the app will refresh in the browser because we have Hot Reloading in Docker :grin:
 11. Commit the change to your github repo
  - it may ask for your github username and password if this is the first time you are commiting from your laptop
 ```
 $ git add .
 $ git commit -m "fixed bug"
 $ git push origin master
 ```
 12. The change should now show in your repo! 
 13. You are now ready to deploy to the cloud!!

 Lets observe what we have just done. You just downloaded code from a repo that needed Nodejs and MongoDB but you never installed them on your local computer! Try and check that you have node and mongodb installed locally, I dare you... Docker allows you to quickly create a container ( aka very small portable virtual machines) that house each tier of the web app. This makes it very easy to run this web app anywhere that has docker installed and with a single command ```docker-compose up```. Now lets see how we can deploy this to the cloud...

 ## Access Oracle Cloud Infrastructure Console
 To begin, login to the OCI Console page to provision the necessary resources to run our web app.

 1. Go the the login page, this will most likely be provided by your hub admin. It should be similar to [https://console.us-ashburn-1.oraclecloud.com](https://console.us-ashburn-1.oraclecloud.com) or [https://console.us-phoenix-1.oraclecloud.com](https://console.us-phoenix-1.oraclecloud.com)

 2. Enter the Tenancy name
 3. Click **Continue** to go to Oracle's IDCS to enter the username and password
 
 

## Create the Networking Resources
 Before we create some cloud servers we need to provision the networking resources. Every cloud compute instance runs in a Virtual Cloud Network (VCN). This way we can restrict the access to the compute instances and direct networking traffic aka allow everyone to access our web app from anywhere.

 1. At the top left of the home screen, click **Menu**
 2. Hover over **Networking** and click **Virtual Cloud Networks**
 3. Click **Create Virtual Cloud Network**
    - Select the Compartment you would like all your resources to be located in
    - **Name**: *OCI-DEMOVCN*
    - Select **Create Virtual Cloud Network Plus Related Resources**
 4. You will be redirected to the VCN details page 
 5. On the left, click **Security Lists**
 6. Click **Default Security List for \<VCN Name\>**
 7. Click **Edit All Rules**
 8. Scroll down and click **Add Rule**
 9. Enter the following details for the new rule:
    - **SOURCE TYPE**: *CIDR*
    - **SOURCE CIDR**: *0.0.0.0/0*
    - **IP PROTOCOL**: *TCP*
    - **SOURCE PORT RANGE**: *ALL*
    - **DESTINATION PORT RANGE**: *6200, 80*

 You have now just create a Virtual Cloud Network (VCN) and allowed the necessary ports to be opened to run your web app.

## Create a Compute Instance
 This step will show you how to create a compute instance that will run your web app

 1. At the top right, click **Menu**
 2. Hover over **Compute** and click **Instances**
 3. Click **Create Instance**
 4. Fill in the necessary details
    - **NAME**: WebApp
    - **AVAILABILITY DOMAIN**: Select *tTie:US-ASHBURN-AD-1*
    - **BOOT VOLUME**: Select *ORACLE-PROVIDED OS IMAGE*
    - **IMAGE OPERATING SYSTEM**: Select *Oracle Linux 7.5*
    - **SHAPE TYPE**: Select *VIRTUAL MACHINE*
    - **SHAPE**: Select *VM.Standard1.1 (1 OCPU, 7GB RAM)*
    - **IMAGE VERSION**: Select the image with the latest tag
    - **SSH KEY**: paste in the contents of your public ssh key, to store the contents of your public key. Go to your terminal and run:
    ```
    $ cat /Users/<your computer name>/.ssh/id_rsa.pub | pbcopy
    ```
    - **VIRTUAL CLOUD NETWORK**: Select *OCI-DEMOVCN*
    - **SUBNET**: Select *Public Subnet tTie:US-ASHBURN-AD-1*
    - Leave **ASSIGN PUBLIC IP ADDRESS** checked
    - Click **Create Instance**

    You will be taken to the details page of your newly created instance. It will show up as yellow with a tag of **Provisioning**. When it turns green and says **Running**. Locate the **Public Ip Address** and copy it down.

## SSH into your instance
 1. In your terminal run the following to connect to your cloud instance
 ```
 $ ssh opc@<public ip address>
 ```

 2. Disable SELinux for docker-compose
 ```
 $ sudo setenforce 0
 ```

 3. Update the yum repos so we can install docker and docker compose
 ```
 $ sudo yum update -y
 ```

 4. Install git and docker onto the cloud instance
 ```
 $ sudo yum install -y git docker-engine
 ```

 5. Start the docker service
 ```
 $ sudo systemctl start docker
 ```

 6. Add the docker permissions to the user opc and enter a new session as opc
 ```
 $ sudo usermod -aG docker opc
 $ sudo su opc
 ```

 7. Install docker-compose onto the machine
 ```
 $ sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/bin/docker-compose
 $ sudo chmod +x /usr/bin/docker-compose
 ```

 8. Clone your repo down to your compute instance
 ```
 $ git clone <url of your git repo>
 ```
 9. Change into the directory and run the web app in the background
   - the -d arguement allows docker to run in the background
 ```
 $ cd react-todolist
 $ docker-compose up -d
 ```
 10. Now go to your browser and enter in the public ip address of the instance to visit your page from the internet!



