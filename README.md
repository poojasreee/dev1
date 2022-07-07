Devops Project

In this project we will be learning how to create a simple pipeline 

What is pipeline in jenkins ?
Pipelines are Jenkins jobs enabled by the Pipeline plugin and built with simple text scripts based on the Groovy 
programming language.
Pipelines leverage the power of multiple steps to execute both simple and complex tasks according to parameters that 
you establish.

To run pipelines, prerequisites - you need to have a Jenkins instance that is set up with the appropriate plugins. 

So in order to create the pipeline, I have used following tools
- Jenkins
- Git as source code repository
- Maven to build the application
- Tomcat 7 to deploy the application

So here we will be creating a simple pipeline that will perform the following operations 
- How to checkout code from repository may be GIT or SVN
- How to create a build.(May be jar or war)
- How to publish the results if required
- How to archive the artifacts generated from the Job
- And lastly how to deploy your application

enter your pipeline syntax.
Pipelines are written as Groovy scripts  that tell Jenkins what to do when they are run

Each stage can have multiple steps
A “step” is a single task that is part of sequence. Steps tell Jenkins what to do.
Click on Save to finish the configuration. Jenkins will redirect you to the initial page of your project.
To execute the project, click on Build Now link.

Console Output
- Every build has a specific page showing everything that happened, just click on the build number link. 
It has various information available like the logs of the process, the user that started the process, how 
much time it took and many others great information.

If you are new to pipeline creation, you might want to start by creating simple groovy scirpts for this 
you can use Snippet Generator 

source code :

https://github.com/poojasreee/dev1

through pipeline script launched jenins server:

# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- main

stages:
  - stage: Build
    jobs:
      - job: BuildWebApp
        variables:
          buildConfiguration: 'Release'
        pool:
          vmImage: 'ubuntu-latest'

        steps:
        - task: UseDotNet@2
          inputs:
            packageType: 'sdk'
            version: '5.x'
        - task: Bash@3
          inputs:
            targetType: 'inline'
            script: |
              #!/bin/bash
              az login 
              az group create --name testrg --location eastasia
              az network vnet create -g testrg -n testVnet --address-prefix 10.0.0.0/16 --subnet-name testSubnet --subnet-prefix 10.0.0.0/24
              az network nic create -g testrg --vnet-name testVnet --subnet testSubnet -n testnic
              az vm create -n testvm -g testrg --image UbuntuLTS --location eastasia --public-ip-sku Standard --custom-data cloud-init-jenkins.txt --authentication-type password --admin-username azureuser --admin-password Pooja@123456789
              az vm open-port --port 8080 --resource-group testrg --name testvm

hit ip:8080  -- jenkins page will open

service jenkins status
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
configured jenkins
installed maven manually in jenkins server

in jenkins console added maven integration plugin
configured global tool configuration -- added maven path


Lets begin with the pipeline configuration. 
For this we need to login to Jenkins
Click New Item on your Jenkins home page,
select Pipeline,
Provide a name lets say – MyFirstPipeline
and click OK.
Description : I am providing as : This is my first pipeline

copy the declarative pipeline:

pipeline{
agent any
stages{

  stage('CheckOutCode'){
    steps{
    git branch: 'master',  url: 'https://github.com/poojasreee/dev1.git'
	
	}
  }
  stage('Build'){
  steps{
  sh  "mvn clean package"
  }
  }
  stage('Deploy'){
    steps{
        sshagent(['deployusr']) {
         sh "scp -o StrictHostKeyChecking=no webapp/target/webapp.war azureuser@20.228.207.172:/opt/tomcat/webapps"
       }
      }
  }
}
}

save & click build now

once done go step wise fst clone git nect build war & then deploy

created a server for tomcat installed tomcat & configired ssh plugin & added key files in it after that i run deploy stage for 
copying war to webapp folder

once succeeded click on the url
