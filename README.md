# Demo Project 2

CD - Deploy Application from Jenkins Pipeline to EC2 Instance (automatically with docker)

## Technologies Used

AWS, Jenkins, Docker, Linux, Git, Java, Maven, Docker Hub

## Project Description

- Prepare AWS EC2 Instance for deployment (Install Docker)
- Create ssh key credentials for EC2 server on Jenkins
- Extend the previous CI pipeline with deploy step to ssh into the remote EC2 instance and deploy newly built image from Jenkins server
- Configure security group on EC2 Instance to allow access to our web application

### Details of project

- Configure ssh connection

  The first step of this project was to install a plugin called SSH Agent, which enables Jenkins to connect to an EC2 instance via SSH.

  New credentials were also created to allow Jenkins to establish this SSH connection. The credentials were of type SSH Username with Private Key, using the username provided by AWS (ec2-user), and the private key generated during the launch instance process of the previous project.

- Configure deploy stage

  This pipeline focuses only on the deploy stage, so the previous stages contain only echo commands. The deploy stage, however, was fully implemented in this project to automatically run the Docker image inside the EC2 instance using the following configuration:

  ```
    stage("deploy") {
      steps {
        script {
          def dockerCmd = 'docker run -p 3080:3080 -d mauriciocamilo/demo-app:1.0'
            sshagent(['ec2-server-key']) {
              sh "ssh -o StrictHostKeyChecking=no ec2-user@54.211.179.187 ${dockerCmd}"
            }
        }
      }
    }
  ```
  
  The SSH connection includes the instance address, the -o flag to suppress the popup, and the Docker command to run the image. It's important to note that a Docker login must be performed on the instance before running the pipeline, as docker run will pull the image from Docker Hub. Additionally, the security group needs to allow access from the Jenkins IP address and open the new port used by the application.

  ![Diagram](./images/aws-pipeline-1.png)

  As seen in the logs, the SSH connection was successfully established, and the image was pulled from Docker Hub.

- Complete pipeline

  In this phase of the project, a complete pipeline was executed, including the build, jar, image, and deploy stages. These stages were configured in the jenkins-jobs branch, which also uses the shared library created in the Jenkins module. Additionally, a new environment variable was added to the pipeline:
  
