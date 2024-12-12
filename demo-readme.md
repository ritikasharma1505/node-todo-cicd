
### Demo CI-CD pipeline setup for Node-todo-app: Jenkins CICD and GitHub Integration

- Create EC2 Instance and login via SSH

- Install Java JDK (pre-requisite)

```
sudo apt update
sudo apt install openjdk-17-jre
java -version

```

Install Jenkins

```
# Download and save the Jenkins GPG key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \ https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add the Jenkins repository with the correct signed-by option
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \ https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

# Update the package list
sudo apt-get update

# Install Jenkins
sudo apt-get install jenkins -y

```
- Access Jenkins server on:

```
http://<public-ip>:8080
```
Note:  Open port 8080 for CustomTCP (Jenkins Server)

- Create admin user and Install suggested plugins


- Generate key-pair via SSH for GitHub Integration with Jenkins
```
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_rsa
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub

cd .ssh/
~/.ssh$ ls
authorized_keys  id_rsa  id_rsa.pub
~/.ssh$ cat id_rsa

~/.ssh$ cat id_rsa.pub

```

- GitHub Integration - Go to GitHub Profile settings --> Select SSH and GPG keys and Create a SSH key with the help of previously generated public key
Named it: ci-cd-pipeline-key

- Create freestyle project:

- Run commands: for manual demo later this will be turned into docker image and container concept

```
sudo apt install nodejs

sudo apt install npm

sudo npm install

node app.js

```
Note:  Open port 8000 for CustomTCP (node-todo-app for manually running it)

- Install docker:

```
sudo apt install docker.io -y
docker ps
sudo usermod -aG docker $USER && newgrp docker
```

Location where Jenkins job is saved: cd /var/lib/jenkins/workspace/demo-ci-cd-pipeline

- Let's Dockerize the node-todo-app:

```
docker build . -t node-todo-app
docker images
docker run -d --name node-todo-app-v1 -p 8000:8000 node-todo-app:latest
docker ps
docker kill <container-id> 
```

- Let's automate this and run from Jenkins:

Goto Jenkins job
Build steps : execute shell commands (add) same commands we used to dockerize the container, now we are going to automate this process with Jenkins tool

```
docker build . -t node-todo-app
docker run -d --name node-todo-app-v1 -p 8000:8000 node-todo-app:latest
```
Error encountered - Permissions issue
This could be cause jenkins user also needs permission to docker daemon

```
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

Access the app on browser:

kill the container, as we move forward to webHooks concept which will eliminate the tedious manual intervention to click on "build now" for job to run

```
docker kill <container-id-b9825fe74324>
```


- Install plugin "GitHub Integration" on Jenkin server

- Now go to Github and create webhook by visiting your repo's setting's Webhook option, create one with the necessary updates. Make sure to git jenkins payload  url in below format:

```
http://3.95.211.173:8080/github-webhook/

```

- Now go back to your ci-cd job and configure Build triggers as "GitHub hook trigger for GITScm polling"

- Try making changes on the repo , views --> todo.ejs and commit changes and you will see, the build is automatically triggered on commits made to the repository.


**Checkout "demo-images" folder for clear outputs**


Trouble shooting :

Permission issue: give permission to jenkins user for access to docker daemon 
Container name already in use: change the 'container name' in the "Execute shell commands" section of job's build configure
Restart jenkins: if updates not applied, and after any plugin installation