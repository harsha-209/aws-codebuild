# aws-codebuild


AWS CodeBuild  integrate 
With sonarqube and maven and Docker and zap owasp


Overview
AWS CodeBuild is a fully managed build service in the cloud. CodeBuild compiles your source code, runs unit tests, and produces artifacts that are ready to deploy. CodeBuild eliminates the need to provision, manage, and scale your own build servers. It provides prepackaged build environments for popular programming languages and build tools such as Apache Maven, Gradle, and more. You can also customize build environments in CodeBuild to use your own build tools. CodeBuild scales automatically to meet peak build requests.
CodeBuild provides these benefits:
Fully managed – CodeBuild eliminates the need to set up, patch, update, and manage your own build servers.


On demand – CodeBuild scales on demand to meet your build needs. You pay only for the number of build minutes you consume.


Out of the box – CodeBuild provides preconfigured build environments for the most popular programming languages. All you need to do is point to your build script to start your first build.


How to run CodeBuild
You can use the AWS CodeBuild or AWS CodePipeline console to run CodeBuild. You can also automate the running of CodeBuild by using the AWS Command Line Interface (AWS CLI) or the AWS SDKs.




To run CodeBuild by using the CodeBuild console, AWS CLI, or AWS SDKs, see Run AWS CodeBuild directly.
As the following diagram shows, you can add CodeBuild as a build or test action to the build or test stage of a pipeline in AWS CodePipeline. AWS CodePipeline is a continuous delivery service that you can use to model, visualize, and automate the steps required to release your code. This includes building your code. A pipeline is a workflow construct that describes how code changes go through a release process.

To use CodePipeline to create a pipeline and then add a CodeBuild build or test action, see Use AWS CodePipeline with AWS CodeBuild. For more information about CodePipeline, see the AWS CodePipeline User Guide.
The CodeBuild console also provides a way to quickly search for your resources, such as repositories, build projects, deployment applications, and pipelines. Choose Go to resource or press the / key, and then enter the name of the resource. Any matches appear in the list. Searches are case insensitive. You only see resources that you have permissions to view. For more information, see Viewing resources in the console.









Create the buildspec file for artifacts

In this step, you create a build specification (build spec) file. A buildspec is a collection of build commands and related settings, in YAML format, that CodeBuild uses to run a build. Without a build spec, CodeBuild cannot successfully convert your build input into build output or locate the build output artifact in the build environment to upload to your output bucket.
Create this file, name it buildspec.yml, and then save it in the root (top level) directory.

This  below file generates artifacts and pushes to s3 buckets
version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto11
  pre_build:
    commands:
      - echo Nothing to do in the pre_build phase...
  build:
    commands:
      - echo Build started on `date`
      - mvn install
  post_build:
    commands:
      - echo Build completed on `date`
artifacts:
  files:
    - target/messageUtil-1.0.jar


Create the build project
In this step, you create a build project that AWS CodeBuild uses to run the build. A build project includes information about how to run a build, including where to get the source code, which build environment to use, which build commands to run, and where to store the build output. A build environment represents a combination of operating system, programming language runtime, and tools that CodeBuild uses to run a build. The build environment is expressed as a Docker image. For more information, see Docker Overview
 on the Docker Docs website.
 For this build environment, you instruct CodeBuild to use a Docker image that contains a version of the Java Development Kit (JDK) and Apache Maven.
 To create the build project


Sign in to the AWS Management Console and open the AWS CodeBuild console at https://console.aws.amazon.com/codesuite/codebuild/home
.


Use the AWS region selector to choose an AWS Region where CodeBuild is supported. For more information, see AWS CodeBuild Endpoints and Quotas in the Amazon Web Services General Reference.


If a CodeBuild information page is displayed, choose Create build project. Otherwise, on the navigation pane, expand Build, choose Build projects, and then choose Create build project.


On the Create build project page, in Project configuration, for Project name, enter a name for this build project (in this example, codebuild-demo-project). Build project names must be unique across each AWS account. If you use a different name, be sure to use it throughout this tutorial.




Note


 On the Create build project page, you might see an error message similar to the following: You are not authorized to perform this operation.. This is most likely because you signed in to the AWS Management Console as an IAM user who does not have permissions to create a build project.. To fix this, sign out of the AWS Management Console, and then sign back in with credentials belonging to one of the following IAM entities:



An administrator IAM user in your AWS account. For more information, see Creating Your First IAM Admin User and Group in the IAM User Guide.


An IAM user in your AWS account with the AWSCodeBuildAdminAccess, AmazonS3ReadOnlyAccess, and IAMFullAccess managed policies attached to that IAM user or to an IAM group that the IAM user belongs to. If you do not have an IAM user or group in your AWS account with these permissions, and you cannot add these permissions to your IAM user or group, contact your AWS account administrator for assistance. For more information, see AWS managed (predefined) policies for AWS CodeBuild.


Both options include administrator permissions that allow you to create a build project so you can complete this tutorial. We recommend that you always use the minimum permissions required to accomplish your task. For more information, see AWS CodeBuild permissions reference.




In Source, for Source provider, choose Amazon S3.


For Bucket, choose codebuild-region-ID-account-ID-input-bucket.


For S3 object key, enter MessageUtil.zip.


In Environment, for Environment image, leave Managed image selected.


For Operating system, choose Amazon Linux 2.


For Runtime(s), choose Standard.


For Image, choose aws/codebuild/amazonlinux2-x86_64-standard:2.0.


In Service role, leave New service role selected, and leave Role name unchanged.


For Buildspec, leave Use a buildspec file selected.


In Artifacts, for Type, choose Amazon S3.


For Bucket name, choose codebuild-region-ID-account-ID-output-bucket.


Leave Name and Path blank.


Choose Create build project.



Sonarqube integrate with aws code build

Regarding above sonarqube we have to create a sonarcloud service

Create a image in docker

Docker pull libararies/sonarqube
Port runs on 9000

This below file show installing  sonar-scaner and running those commands 



version: 0.2
env:
  variable:
    LOGIN: prod/sonar:0d01a200a2155c96e0fe7ac330ce9c27fbfaefcf
    HOST: prod/sonar:http://3.14.141.135:9000
    Organization: prod/sonar:Organization
    Project: prod/sonar:ajay2
phases:
  install:
    runtime-versions:
      java: openjdk8
  pre_build:
    commands:
      - apt-get update
      - apt-get install -y jq
      - apt-get install -y nodejs
      - apt-get install -y wget
      - apt-get install -y unzip
      - wget http://www-eu.apache.org/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
      - tar xzf apache-maven-3.5.4-bin.tar.gz
      - ln -s apache-maven-3.5.4 maven
      - wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.2.0.1873-linux.zip
      - unzip ./sonar-scanner-cli-4.2.0.1873-linux.zip
      - export PATH=$PATH:/sonar-scanner-4.2.0.1873-linux/bin/
      - npm install -y
  build:
    commands:
      - mvn test     
      - sonar-scanner   -Dsonar.projectKey=ajay2   -Dsonar.sources=.   -Dsonar.host.url=http://3.14.141.135:9000   -Dsonar.login=0d01a200a2155c96e0fe7ac330ce9c27fbfaefcf
      - sleep 5
      - curl https://sonarcloud.io/api/qualitygates/project_status?projectKey=$Project >result.json
      - cat result.json
      - if [ $(jq -r '.projectStatus.status' result.json) = ERROR ] ; then $CODEBUILD_BUILD_SUCCEEDING -eq 0 ;fi





Second new file for above script

version: 0.2

env:
  variables:
    SONAR_LOGIN: "MY_SONARQUBE_AUTHTOKEN"
    SONAR_HOST: "MY_SONARQUBE_URL"

    #You should use parameter-store here instead
phases:
  build:
    commands:
      - mvn test
      - mvn sonar:sonar -Dsonar.login=0d01a200a2155c96e0fe7ac330ce9c27fbfaefcf -Dsonar.host.url=http://3.14.141.135:9000
  post_build:
    commands:
      - mvn sonar:sonar -Dsonar.login=0d01a200a2155c96e0fe7ac330ce9c27fbfaefcf -Dsonar.host.url=http://3.14.141.135:9000


This above script please note  Run all commands in post build(like sonar-commands)



Create a buildspec.yaml for generating a docker image using codebuild





version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - $(aws ecr get-login --no-include-email --region $AWS_DEFAULT_REGION)
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...          
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG      
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker image...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG


Integarating both sast and maven war file  into s3 bucket
version: 0.2

phases:
  install:
    runtime-versions:
      java: openjdk8
  pre_build:
    commands:
      - echo Nothing to do in the pre_build phase...
  build:
    commands:
      - echo Build started on `date`
      - mvn install
  post_build:
    commands:
      - echo Build completed on `date`
      - mvn sonar:sonar -Dsonar.login=0d01a200a2155c96e0fe7ac330ce9c27fbfaefcf -Dsonar.host.url=http://3.21.159.38:9000
artifacts:
  files:
    - target/works-with-heroku-1.0.war



codebuild for owasp with zap

version: 0.2
phases:
  install:
    commands:
      - sudo nohup /usr/local/bin/dockerd --host=unix:///var/run/docker.sock --host=tcp://127.0.0.1:2375 --storage-driver=overlay2&
      - timeout 15 sh -c "until docker info; do echo .; sleep 1; done"
  build:
    commands:
      - docker images
      - touch ajay
      - docker run -v $(pwd):/zap/wrk/:rw -tid --name ajay54 owasp/zap2docker-weekly zap-baseline.py -t https://243gvx5a47.execute-api.us-eas$
      - docker images
      - docker ps -a
      - docker exec -it ajay54 /bin/bash
      - ls

  post_build:
    commands:
      - pwd
      - ls





docker run -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-weekly zap-baseline.py -t https://243gvx5a47.execute-api.us-east-2.amazonaws.com/prod -g gen.conf -r testreport.html


https://www.zaproxy.org/docs/docker/about/





version: 0.2

phases:
  install:
    runtime-versions:
      java: openjdk8
  pre_build:
    commands:
      - echo Nothing to do in the pre_build phase...
  build:
    commands:
      - echo Build started on `date`
      - mvn install
  post_build:
    commands:
      - echo Build completed on `date`
      - mvn sonar:sonar -Dsonar.login=0d01a200a2155c96e0fe7ac330ce9c27fbfaefcf -Dsonar.host.url=http://3.16.137.88:9000
artifacts:
  files:
    - target/works-with-heroku-1.0.war

      





https://aws.amazon.com/blogs/containers/scanning-images-with-trivy-in-an-aws-codepipeline/




Aquva trievy and its ecr pushing image

version: 0.2

phases:
  install:
    runtime-versions:
        docker: 18
  pre_build:
    commands:
      - nohup /usr/local/bin/dockerd --host=unix:///var/run/docker.sock --host=tcp://127.0.0.1:2375 --storage-driver=overlay2&
      - timeout 15 sh -c "until docker info; do echo .; sleep 1; done"
      - apt-get install wget apt-transport-https gnupg
      - wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | apt-key add -
      - echo deb https://aquasecurity.github.io/trivy-repo/deb bionic main | tee -a /etc/apt/sources.list.d/trivy.list
      - apt-get update
      - apt-get install -y trivy
      - $(aws ecr get-login --region $AWS_DEFAULT_REGION --no-include-email)
      - REPOSITORY_URI=805392809179.dkr.ecr.us-east-2.amazonaws.com/trendmicro
  build:
    commands:
      - docker build -t $REPOSITORY_URI:success .
  post_build:
    commands:
      - trivy --no-progress --exit-code 0 --severity HIGH,CRITICAL $REPOSITORY_URI:success
      - docker push $REPOSITORY_URI:success












Dast with zaproxy in code build


version: 0.2
phases:
  install:
    commands:
      - sudo nohup /usr/local/bin/dockerd --host=unix:///var/run/docker.sock --host=tcp://127.0.0.1:2375 --storage-driver=overlay2&
      - timeout 15 sh -c "until docker info; do echo .; sleep 1; done"
  build:
    commands:
      - docker images
      - touch ajay
      - docker run  -t  owasp/zap2docker-stable zap-baseline.py -t https://www.example.com || true >> report.html
  post_build:
    commands:
      - pwd
      - ls
      - cat report.html




Nodejs project 


version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto11
      golang: 1.13
      nodejs: 10
  build:
    commands:
      - echo Building the Node code...
      - npm install
      - npm test
      - npm install test
      - wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.3.0.1492-linux.zip
      - unzip ./sonar-scanner-cli-3.3.0.1492-linux.zip
      - ls
      - pwd
      - cd sonar-scanner-3.3.0.1492-linux
      - ls
      - cd bin
      - ls
      - export PATH=$PATH:$pwd/sonar-scanner-3.3.0.1492-linux/bin/
      - export PATH=$pwd/sonar-scanner-3.3.0.1492-linux/bin:$PATH
      - sonar-scanner  -Dsonar.login=a33f67f17db1c1d5dc1b929de1574d3ac563cc24 -Dsonar.host.url=http://18.220.196.198:9000
  post_build:
    commands:
      - echo Build completed on `date`
      - sonar:sonar -Dsonar.login=d7063144e831c0f8e214c21e4a13afaa97beb67e -Dsonar.host.url=http://18.220.196.198:9000










Correctly for nodejs project

version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 10
      java: corretto11
  build:
    commands:
      - echo Building the Node code...
      - npm install
      - wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.3.0.1492-linux.zip
      - unzip sonar-scanner-cli-3.3.0.1492-linux.zip
      - export PATH=$PATH:./sonar-scanner-3.3.0.1492-linux/bin/
      - sonar-scanner  -Dsonar.login=a33f67f17db1c1d5dc1b929de1574d3ac563cc24 -Dsonar.host.url=http://18.220.196.198:9000  -Dsonar.projectKey=ram
  post_build:
    commands:
      - echo Build completed on `date`

