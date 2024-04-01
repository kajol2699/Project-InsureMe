# Project-InsureMe 

InsureMe was having trouble managing their software because it was all one big piece. </br>
As they grew bigger, it became even harder to manage. <br>

### Requirements:

#### Automated Deployment:</br>
Whenever a developer makes changes to the code and pushes them to the master branch of the Git repository, </br>
Jenkins should automatically start a deployment process.
</br>
#### CI/CD Pipeline: </br>
Jenkins should: </br>
 
* Check out the latest code from the master branch.</br>
* Compile and test the code to ensure it works correctly.</br>
* Package the application into a container using Docker.</br>
* Deploy the containerized application to a preconfigured test server on AWS.</br>

With DevOps Approch I used several devops tools such as  <br>

- Git: Managed code changes with version control. </br>
- Jenkins: Automated integration, testing, and deployment processes. </br>
- Docker: Containerized applications for consistency and scalability. </br>
- Ansible: Automated server configuration and infrastructure management. </br>
- Selenium: Automated testing of web applications. </br>
- AWS: Provided infrastructure for hosting and deploying the application. </br>
- Together, these tools streamlined development, testing, and deployment, ensuring efficient management of the InsureMe project. </br>


Infrastructure Setup:

You create two virtual servers (EC2 instances) on Amazon Web Services (AWS): one called Master and the other called Node.
These servers will host your application and manage its deployment.
CI/CD Setup:

You install Jenkins on the Master server to automate the process of building, testing, and deploying your application.
Think of Jenkins as a robot that helps you with repetitive tasks like deploying code automatically when it changes.
Jenkins Job Configuration:

You set up Jenkins to watch your code repository on GitHub.
Whenever someone makes changes to the code and pushes them to GitHub, Jenkins automatically kicks off a process to update and deploy your application.
Docker Setup:

You use Docker to package your application and its dependencies into a container, making it easy to deploy and run anywhere.
Docker makes sure your application runs consistently in different environments.
Pipeline Configuration:

You define a sequence of steps (pipeline) in Jenkins to build, test, and deploy your application automatically.
This pipeline runs every time someone makes changes to the code, ensuring that the latest version of your application is always available.
Build Pipeline:

Whenever someone pushes changes to the code, Jenkins pulls the latest code, builds the application, creates a Docker image, and pushes it to DockerHub (a service for storing Docker images).
Then, it deploys the updated application to your servers on AWS using Ansible, a tool for automating server configuration.
Final Output:

With this setup, you have a fully automated process for building, testing, and deploying your application.
Whenever you or your team make changes to the code, Jenkins takes care of the rest, ensuring that your application is always up-to-date and running smoothly on your servers.

#Steps:
## Tech Stack
✓ AWS - For creating ec2 machines as servers and deploy the web application. </br>
✓ Git - For version control for tracking changes in the code files </br>
✓ Jenkins - For continuous integration and continuous deployment  </br>
✓ Docker - For deploying containerized applications </br>
✓ Ansible - Configuration management tools  </br>
✓ Selenium - For automating tests on the deployed web application </br>

## Step 1: Create Infrastructure
Create two ec2 instance 
1. Master
2. Node
### Master
1. ami = ubuntu
2. instance type = t2.medium
3. ports =  22, 8080
   
Take SSH and Connect to Instance
## CI/CD Setup

### Install Jenkins for Automation:
### Install Jenkins on the EC2 instance to automate deployment: Install Java
sudo apt update
sudo apt install  openjdk-11-jdk




#jenkins
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \   </br>
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key  </br>
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null   </br>
sudo apt-get update    </br>
sudo apt-get install jenkins   </br>
sudo systemctl start jenkins    </br>
sudo systemctl enable jenkins   </br>


Access Jenkins in a web browser using the public IP of your EC2 instance.
publicIp:8080


### Install Necessary Plugins in Jenkins:

#### Goto Manage Jenkins →Plugins → Available Plugins →

##### Install below plugins:
1.Maven Integration plugin  </br>
2.Docker  </br>
3.Docker Commons  </br>
4.Docker Pipeline  </br>
5.Docker API  </br>
6.docker-build-step  </br>

##### Add tools in  Dashboard->Manage Jenkins-> Tools


#### Create Job in Jenkins : go to Dashboard->Item Name


As soon as the developer pushes the updated code on the GIT master branch, the Jenkins job should be triggered using a GitHub Webhook and Jenkins job should be triggered

To Create Webhook: go to settings of your github repository 
In Payload URL: Add yours jenkins url

In Jenkins Job ->Configuration->choose GitHub hook trigger for GITScm polling


## Install  Docker:

sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

Add DockerHub Credentials:

To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps: </br>
Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials." </br>
Click on "System" and then "Global credentials (unrestricted)."  </br>
Click on "Add Credentials" on the left side.    </br>
Choose "Secret text" as the kind of credentials.  </br>
Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker"). </br>
Click "OK" to save your DockerHub credentials. </br>

pipeline {
    
     agent any

    stages {
        stage('Code-Checkout') {
            steps {
              checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-token', url: 'https://github.com/kajol2699/Project-InsureMe.git']])
            }
        }
        
        stage('Code-Build') {
            steps {
               sh 'mvn clean package'
            }
        }
        
        stage('Containerize the application'){
            steps { 
               echo 'Creating Docker image'
               sh "docker build -t kaju912/insurance:latest ."
            }
        }
        
        stage('Docker Push') {
    	agent any
          steps {
       	withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            	sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                sh 'docker push kaju912/insurance:latest'
        }
      }
    }
    
      
    stage('Code-Deploy') {
        steps {
           ansiblePlaybook credentialsId: 'ansible', installation: 'ansible', playbook: 'ansible-playbook.yml', vaultTmpPath: ''       
        }
      }
    
   }
}



Build Pipeline:




Final Output:
