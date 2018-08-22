# How to Deploy

## Prerequisites
In order to deploy this 3 tier web app locally you will need the following tools installed
 - Docker
 - Docker-compose
 - git
 - create a [github account](https://github.com/)
 - a public and private ssh key

If you using a mac
 - go to the following [link](https://store.docker.com/editions/community/docker-ce-desktop-mac) to install docker and docker compose
 - go to the following [link](https://git-scm.com/download/mac) to install git for mac
 - If you do not have a public and private ssh key on your computer you can install them with the following and accept the default with no passphrase
 ```
 $ ssh-keygen
 ```



 In order to deploy this web app to the cloud. Collect the following information form your hub admin
 - url of the oracle cloud infrastructure console
 - Tenancy name
 - user name
 - password 

## Abstract
This repo contains the code for the backend service in Nodejs and the code for the frontend that is in React. It also contains the necessary files to containerize both apps and a docker compose file that will deploy both apps with ```$ docker-compose up``` along with a mongoDB instance to store the data. This is a simple TODO list. There is a missing snippet in the backend code that is once fixed will be ready to deploy in "Production" aka the cloud. Our cloud provider is Oracle Cloud Infrastructure that provides best in performance infrastructure.

 ## Architecture
This architecture illustrates what the web app will look like in Oracle Cloud Infrastructure.

![Architecture](./images/architecture.jpg)

 ## Step 1 Forking
 1. Go to [https://github.com/schmidtp0740/react-todolist](https://github.com/schmidtp0740/react-todolist)
 ![](./images/pic1.jpg)
 2. Fork the repo
 ![](./images/pic2.jpg)
 3. Click **Clone or Download**
 ![](./images/pic3.jpg)
 4. Copy the url
 ![](./images/pic4.jpg)
 5. Clone the repo to your desktop
    
    a. open up your terminal (git bash if on windows)
    b. change into the directory that you want to clone the repo in and clone the repo

 ```
 $ cd /User/<your computer name>/
 $ git clone <url you copied>
 ```
 6. change into the directory
 ```
 $ cd /react-todolist
 ```

 ## Run the Web App Locally
 1. Build the docker images
 ```
 $ docker-compose build
 ```
 2. you will begin to see a lot of logs and they may be in red which is okay, wait until you recieve the prompt again

    a. this process may take several minutes
 3. Run the 3 containers which will the react frontend, backend and the mongo db container
 ```
 $ docker-compose up
 ```
 4. you will see the logs and see all the instances run
 5. Go to your browser and visit [http://localhost](http://localhost)
 6. Try to add a new task
 7. You will notice it will refuse to add a new task because there is a bug
 8. Go to backend/app.js
 9. Uncomment lines 12 - 19(remove the backslashes)
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
 10. Save the file
 11. You will notice the app will refresh in the background in the browser because we have hotreloading :grin:
 12. You are now ready to deploy to the cloud!!

 Lets observe what we have just done. You just downloaded code from a repo that needed Nodejs and MongoDB but you never installed them on your local computer! Try and check that you have node and mongodb installed locally, I dare you... Docker allows you to quickly create a container ( aka very small portable virtual machines) that houses each component of the web app. This makes it very easy to run this web app anywhere that has docker installed and can run a single command ``` docker-compose up```. Now lets see how we can deploy this to the cloud...

 ## Access Oracle Cloud Infrastructure Console
 To begin we will login to the OCI Console page to provision the necessary resources to run our web app.

 1. go the the login page, this will most likely be provided by your hub admin. It should be similar to [https://console.us-ashburn-1.oraclecloud.com](https://console.us-ashburn-1.oraclecloud.com) or [https://console.us-phoenix-1.oraclecloud.com](https://console.us-phoenix-1.oraclecloud.com)
 ![](./images/pic5.jpg)

 2. Enter the Tenancy name
 ![](./images/pic6.jpg)
 3. Click **Continue** to go to Oracle's IDCS to enter the username and password
 ![](./images/pic7.jpg)
 
 

## Create the Networking Resources
 Before we create some cloud servers we need to provision the networking resources. Every cloud compute instance runs in a Virtual Cloud Network(VCN). This way we can restrict the access to the compute instances and direct networking traffic aka allow everyone to access our webapp from anywhere.

 1. At the top left click **Menu**
 2. Hover over **Networking** and click **Virtual Cloud Networks**
 3. Click **Create Virtual Cloud Network**
    - Select the Compartment you would like all your resources to be located in
    - **Name**: *OCI-DEMOVCN*
    - Select **Create Virtual Cloud Network Plus Related Resources
 4. On the left click **Security Lists**
 5. Click **Default Security List for \<VCN Name\>**
 6. Click **Edit All Rules**
 7. Scroll down and click **Add Rule**
 8. Enter the following details for the new that popped up
    - **SOURCE TYPE**: *CIDR*
    - **SOURCE CIDR**: *0.0.0.0/0*
    - **IP PROTOCOL**: *TCP*
    - **SOURCE PORT RANGE**: *ALL*
    - **DESTINATION PORT RANGE**: *6200, 80*

 You have now just create a virtual cloud network(VCN) and allowed the necessary ports to run your web app.

## Create a Compute Instance
 This step will show you how to create a compute instance that will run your web app

 1. At the top right click **Menu**
 2. Hover over **Compute** and click **Instances**
 3. Click **Create Instance**
 4. Fill in the necessary details
    - **NAME**: WebApp
    - **AVAILABILITY DOMAIN**: Select *tTie:US-ASHBURN-AD-1*
    - **BOOT VOLUME**: Select *ORACLE-PROVIDED OS IMAGE*
    - **IMAGE OPERATING SYSTEM**: Select *Oracle Linux 7.5*
    - **SHAPE TYPE**: Select *VIRTUAL MACHINE*
    - **SHAPE**: Select *VM.Standard1.1 (1 OCPU, 7GB RAM)
    - **IMAGE VERSION**: Select the image with the latest tag
    - **SSH KEY**: paste in the contents of your public ssh key, to store the contents of your public key and store in the clipboard, in your terminal run the following
    ```
    $ cat /Users/<your compute name>/.ssh/id_rsa.pub | pbcopy
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

 2. disable SELinux for docker-compose
 ```
 $ sudo setenforce 0
 ```

 2. Update the yum repos so we can install docker and docker compose
 ```
 $ sudo yum update -y
 ```

 3. Install git and docker onto the cloud instance
 ```
 $ sudo yum install -y git docker-engine
 ```

 4. start the docker service
 ```
 $ sudo systemctl start docker
 ```

 5. add the docker permissions to the user opc and enter a new session as opc
 ```
 $ sudo usermod -aG docker opc
 $ sudo su opc
 ```

 4. Install docker-compose onto the machine
 ```
 $  sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/bin/docker-compose
 $ sudo chmod +x /usr/bin/docker-compose
 ```

 5. Clone your repo down to your compute instance
 ```
 $ git clone <url of your git repo>
 ```
 6. change into the directory and run the web app in the background
 ```
 $ cd react-todolist
 $ docker-compose up -d
 ```
 the -d arguement allows docker to run in the background
 7. Now go to your browser and enter in the public ip address of the instance to visit your page from the internet!



