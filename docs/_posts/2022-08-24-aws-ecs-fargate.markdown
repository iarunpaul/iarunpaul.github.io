---
layout: post
title:  "AWS ECS Fargate with microservices deployment"
date:   2022-08-23 13:27:00 +0530
categories: jekyll update
published: false
---

I could find a lot of tutorials where ECS fargate was used to deploy a dockerized container with ease.

But I found it bit difficult to get an example deployment of a microservices application with all the internal communications with the application microservices layers.

And, of ource I bumped into many troubles to make it up; I thought I will just document it for an easy walkthrough for anyone else to do this again.

I could find an example from [amazon blogs](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/deploy-java-microservices-on-amazon-ecs-using-aws-fargate.html),  and thankfully the repo was also available at [here](https://github.com/aws-samples/amazon-ecs-java-microservices).

So lets get to it,

The deployment consists of three stages:

- Part One: Moving existing Java Spring application to a container deployed using ECS
- Part Two: Breaking the monolith apart into microservices on ECS
- Part Three: Create a continuous integration and continuous delivery

The turial gave prerequisites links as follows:

#### Prerequisites
You will need to have the latest version of the AWS CLI and maven installed before running the deployment script. If you need help installing either of these components, please follow the links below:

- [Installing the AWS CLI](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)
- [Installing Maven](https://maven.apache.org/install.html)
- [Installing Docker](https://docs.docker.com/engine/installation/)
- [Installing Python](https://www.python.org/downloads/)
- [Installing JQ](https://stedolan.github.io/jq/download/)

But unfortunately `Maven` installation didn't work for me.

>**Note**: The set up uses Ubuntu-18.04 Distro in WSL2

I made it work with the following the installation commands:
```bash
tar xzvf apache-maven-3.8.6-bin.tar.gz
```
Tarball is available [here](https://maven.apache.org/download.cgi).

But apache maven has dependency on JDK but unfortunately the JDK by oracle is not open source anymore in Ubuntu-16.04 or higher and hence I had to install the openjdk to keep it going.

The [Ask Ubuntu](https://askubuntu.com/questions/761127/how-do-i-install-openjdk-7-on-ubuntu-16-04-or-higher) and [Stackoverflow](https://stackoverflow.com/questions/16263556/installing-java-7-on-ubuntu) helped me to figure out that.

Better install  `openjdk-8` or higher.

```bash
sudo apt-get install openjdk-8-jdk
```

Further struggled with the `PATH` variable and `JAVA-HOME` ENV variable set up, since it was not properly documented.

Finally I did this to make it work.

```bash
PATH=$PWD/apache-maven-3.8.6/bin:$PATH
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
```
Then I decided to run the python script to get it moving.
And run the script..
```sh
python setup.py -m setup -r us-east-1
```
But I was welcomed with more errors.

This time with python module import error.
```sh
Traceback (most recent call last):
  File "setup.py", line 11, in <module>
    import boto3
```

I tried to install the module:
```zsh
pip3 install boto3
```

But the error sticked.

Yes..you are right...I took the wrong python...
Then I imported with `python2`
```zsh
sudo apt install python-pip
pip install boto3
```

Now the script ran with AWS errors:

```zsh
INFO:__main__:Mode: setup
/home/iarunpaul/.local/lib/python2.7/site-packages/boto3/compat.py:86: PythonDeprecationWarning: Boto3 will no longer support Python 2.7 starting July 15, 2021. To continue receiving service updates, bug fixes, and security updates please upgrade to Python 3.6 or later. More information can be found here: https://aws.amazon.com/blogs/developer/announcing-end-of-support-for-python-2-7-in-aws-sdk-for-python-and-aws-cli-v1/
  warnings.warn(warning, PythonDeprecationWarning)
INFO:botocore.credentials:Found credentials in shared credentials file: ~/.aws/credentials
ERROR:__main__:An error occurred (InvalidClientTokenId) when calling the CreateRole operation: The security token included in the request is invalid
```

I will have to look at it and to be continued with my troubleshooting....