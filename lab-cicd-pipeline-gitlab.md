```
title: Lab Pipeline As Code with GitLab
theme: ["pipeline-as-code"]
tools: ["gradle", "spring", "gitlab", "aws"]
dora: ["Version control", "Continuous integration", "Deployment automation", "Continuous delivery"]
```

# Goals

In this hands-on workshop, participants learn how to develop a pipeline through continuous integration, continuous delivery and continuous deployment. Your team members will deepen their knowledge and hone their skills in process automation and integration of new tools into the development cycle of a software application.

As you use more CI/CD tools, you will notice that the configuration of the YAML (a recursive acronym for _YAML Ain't Markup Language_) file for GitLab looks similar to the other CI/CD tools (except for Jenkins). Learning how to properly configure GitLab's pipeline is going to be a good investment for you.

# Resources

GitHub Demo Reference is there: https://github.com/devops-opensource/continuous-deployment

# Theory

Read article at [Gologic GitLab blog article](https://www.gologic.ca/en/continuous-deployment-with-pipeline-as-code-quick-start-series-3-gitlab/)

# Instructions

## Get Started

```
git clone https://github.com/devops-opensource/continuous-deployment
```

In the root folder, add a .gitlab.yml file.

## Build

The build is the first step necessary to obtain our artifact in the pipeline. We will obtain our application compiled and ready to be tested and published by the rest of our process.

```
# Base docker image to run pipeline
image: docker:latest
services:
  - docker:dind
  
# path to cache Maven dependencies
cache:
  paths:
    - .m2/

# Stages in pipeline
stages:
  - build

# Build maven step
maven-build:
  # Use maven docker image
  image: maven
  # Join to build stage
  stage: build
  # Command to execute in docker image
  script: "mvn package -Dmaven.repo.local=.m2"
  artifacts:
    paths:
      - target/*.jar
```

## Deploy

Now that we have an artifact ready to use from our continuous integration process we need to deploy it in an environment to start testing it!

```
# Stages in pipeline
stages:
  - build
  - deploy

# Deploy AWS step
aws-deploy:
  # Use aws docker with eb-cli
  image: chriscamicas/awscli-awsebcli
  # Join to deploy stage
  stage: deploy
  # Commands to execute to deploy application to AWS
  script:
  - echo "$AWS_ACCESS_KEY_ID"
  - eb init continuous-deployment-demo -p "64bit Amazon Linux 2017.09 v2.6.4 running Java 8" --region "ca-central-1"
  - eb create gitlab-env --single || true
  - eb setenv SERVER_PORT=5000
  - eb deploy gitlab-env
  - eb status
```

# Conclusion

The pipeline (.gitlab.yml) is defined. With each code change, the steps and stages are triggered to compile, store in the pipeline’s working folder, and deploy to AWS.

Thanks to its integration, changes to the source code are visible in the detailed view of the pipeline and, conversely, pipeline executions are visible in the source code change view. This integration facilitates branch status navigation and validation.
