AWS CI/CD Pipeline for Bitbucket to Elastic Beanstalk Deployment
This repository provides a comprehensive guide to setting up an AWS CI/CD pipeline that fetches code from Bitbucket, builds it using AWS CodeBuild, and deploys it to an AWS Elastic Beanstalk (EBS) environment.
Table of Contents
•	Overview
•	Prerequisites
•	Setup Instructions
•	BuildSpec.yaml Explanation
•	Create the CI/CD Pipeline
•	License
Overview
This CI/CD pipeline automates the deployment of a Java application to AWS Elastic Beanstalk, integrating with Bitbucket for source control and AWS CodeBuild for building the application. The pipeline also provisions an Amazon RDS instance for database needs.
Prerequisites
Before setting up the pipeline, ensure you have:
1.	An AWS account with permissions to create resources (RDS, EBS, CodeBuild).
2.	A Bitbucket account to host your source code.
3.	Java (Corretto 17) and Maven knowledge for the application.
Setup Instructions
1. Create RDS Instance
•	Launch an Amazon RDS instance with the MySQL database and port 3306.
2. Create Elastic Beanstalk Environment
•	Create an Elastic Beanstalk environment which will automatically provision two EC2 instances.
3. Configure Security Groups
•	Allow the EC2 instances to access the RDS database:
o	Go to the RDS Security Group.
o	Add an inbound rule allowing traffic from the EC2 instances on port 3306.
4. Access the Database
•	Connect to the RDS instance from one of the EC2 instances:
bash
Copy code
mysql -h <RDS-EP> -u <username> -p<password> dbname
•	Make necessary changes in the database and exit.
5. Migrate GitHub to Bitbucket
1.	Create an SSH key and add the public key to your Bitbucket account.
2.	Use the following command to add the fingerprint:
bash
Copy code
ssh -T git@bitbucket.org
6. Create a Bitbucket Repository
•	Set up a new repository in Bitbucket for your project.
7. Clone the Source Code
1.	Clone your GitHub repository locally.
2.	Update the Git configuration:
bash
Copy code
git remote rm origin
git remote add origin <bitbucket-url>
3.	Push the code to Bitbucket.
8. Create AWS CodeBuild Project
•	Create a new AWS CodeBuild project with the necessary parameters, including:
o	Source repository (Bitbucket URL)
o	Branch name
9. Create BuildSpec.yaml File
Add the following buildspec.yaml file to your project:
version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto17
  pre_build:
    commands:
      - apt-get update
      - apt-get install -y jq 
      - wget https://archive.apache.org/dist/maven/maven-3/3.9.8/binaries/apache-maven-3.9.8-bin.tar.gz
      - tar xzf apache-maven-3.9.8-bin.tar.gz
      - ln -s apache-maven-3.9.8 maven
      - sed -i 's/jdbc.password=admin123/jdbc.password=q1wF046nCkVIdQjSmp9o/' src/main/resources/application.properties
      - sed -i 's/jdbc.username=admin/jdbc.username=admin/' src/main/resources/application.properties
      - sed -i 's/db01:3306/vprords.cdocqq0oodyp.ap-south-1.rds.amazonaws.com:3306/' src/main/resources/application.properties
  build:
    commands:
      - mvn install
  post_build:
    commands:
      - mvn package
artifacts:
  files:
    - '**/*'
  base-directory: 'target/vprofile-v2'

10. Configure the Build Spec in CodeBuild
•	Paste the buildspec.yaml content in the appropriate section of your CodeBuild project.
11. Run the Build
•	Execute the build command and monitor the logs for any issues.
12. Create the CI/CD Pipeline
1.	Create a new AWS CodePipeline.
2.	Configure the following values:
o	Version control (Bitbucket)
o	Runtime environment (Elastic Beanstalk)
o	Build environment (CodeBuild)
3.	Run the AWS Pipeline and confirm successful deployment.

