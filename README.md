# **Emojification-of-text-using-Deep-Learning**

## How to deploy

* Install the docker for windows 

<a href="https://docs.docker.com/desktop/install/windows-install/"><p>Docker-Install-Win</p></a>

* Create a Dockerfile in the same directory as the project

```Dockerfile
FROM python:3
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 5000
ENTRYPOINT ["python3"]
CMD ["application.py"]
```

* The above Dockerfile instructs the docker to install python from the Docker hub and then create an ‘app’ directory to copy all the contents of our web app. It then installs all the packages mentioned in the requirements.txt file. Then the port 5000 is exposed to access the application from outside the container. By default, the application.py file is run using the CMD command.

    - Run in terminal
```bash
docker build -t flaskapp:latest .
```

* The ‘.’ at the end of this command copies all the files in the current directory to build the image. We will then run the docker container at port 5000 using the docker image that is created above. The command is as follows:

```
docker run -p 5000:5000 flaskapp
```
Check if the application is accessible outside the container by launching the app on port 5000 in the browser.

<hr>

## Pushing the Docker image to AWS ECR

* Go and make a new account on AWS and then create a new repository in ECR. Copy the URI of the repository and then run the following command in the terminal to tag the image with the URI of the repository.

Link for AWS free tier: <a href = "https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all"><p>aws-tier.com</p></a>

* We have to install the awscli in our desktop by using ```pip install awscli --upgrade --user``` in order to perform actions in AWS directly from the terminal. We have to configure our AWS credentials using the command ```aws configure``` from our terminal where we enter our access key ID, secret access key, default region, and output format.

* We will be using the AWS Elastic Container Registry to store and deploy our docker container images. We have to first create a repository in AWS ECR in order to store Docker images. This is done using the following command:

```
aws ecr create-repository --repository-name ml_flask_app
```

* Then to get permission to access the AWS ECR we enter the following command-

```
$(aws ecr get-login --region region_name --no-include-email)
```

* Then we tag our MLapp container in local with the repository that is created. Here account_id and region name differs for each user.

```
docker tag flaskapp:latest aws_account_id.dkr.ecr.region_name.amazonaws.com/
```

* Then we have to push our local docker image to the repository created in AWS ECR.

```
docker push aws_account_id.dkr.ecr.region_name.amazonaws.com/ml_flask_app
```

<hr>

## Deploying the ML app on AWS EC2

* We have to create a new EC2 instance in AWS. We have to select the Ubuntu Server 18.04 LTS (HVM), SSD Volume Type as the AMI. Then we have to select the t2.micro instance type. Then we have to configure the security group to allow HTTP traffic on port 80. Then we have to create a new key pair and download it. We have to save the key pair in a safe location as it is used to access the instance.

After pushing the image to AWS ECR, we have to create an EC2 instance in which we can serve the web app. AWS offers many instances in the free tier range and we can make use of that. We will be launching a Linux machine with most of the configurations which are set by default and the security groups alone which are changed as follows:

SSH -> Only from our IP and

Custom TCP -> Port 5000.

These security group rules are necessary for our web app to run. Once the instance is launched and is running, we can ssh into the instance by entering the following command in the terminal in the same folder in which our ‘pem’ file is present.


<h2>Know this before you go ahead</h2>
<!-- ---------- -->
* To run the ssh command on a Windows machine, you will need to install an SSH client such as PuTTY or OpenSSH.

Once you have an SSH client installed, you can use the following steps to connect to an EC2 instance using the ssh command:

* Open the command prompt or PowerShell.
* Change to the directory where the test_ml_app.pem file is located.
* Run the ssh command with the necessary arguments:

```
ssh -i "test_ml_app.pem" ec2-user@ec2-12-345-67-89.us-east-1.compute.amazonaws.com
```

* Replace ```ec2-12-345-67-89.us-east-1.compute.amazonaws.com``` with the public DNS of your EC2 instance.

Alternatively, you can use PuTTY to connect to your EC2 instance. To do this, follow these steps:

* Launch PuTTY.
* In the "Host Name (or IP address)" field, enter the public DNS of your EC2 instance.
* In the "Category" pane on the left, navigate to "Connection > SSH > Auth".
* Click on the "Browse" button and select the test_ml_app.pem file.
* Click "Open" to establish the connection.
----------

```
ssh -i "test_ml_app.pem" ec2-user@ec2-12-345-67-89.us-east-1.compute.amazonaws.com
```

* Once we are logged into the instance we can install docker by using the following command and start it.

```
sudo yum install -y docker

sudo docker service start
```

* We have to configure the AWS credentials as we did before by entering aws configure command again. Then enter the following commands so that the ec2 user is added to perform docker commands in Linux machine.

```
sudo groupadd docker

sudo gpasswd -a ${USER} docker

sudo service docker restart
```

* Then exit the instance and ssh again into the instance. Run the following commands to pull the docker image from AWS ECR and then run the docker container in the machine.

```
docker pull account_id.dkr.ecr.region_name.amazonaws.com/ml_flask_app_:latest

docker run -p 5000:5000 account_id.dkr.ecr.region_name.amazonaws.com/ml_flask_app
```
* Get the public IP of the instance from the instance details page and add port 5000 while launching it in the browser. The app is finally up and running on AWS.

