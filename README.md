# Docker-Data-Science-Workflow

![Overall](../master/Diagram.png)

This is a workflow I developed that allow me to spin up a more powerful machine when working with data.
By using Docker and GIT/Github, I can easily transfer between my development computer and an instance in
the cloud.

## Steps to Setup

Before you can use this, there is some prep work you will have to do:

1. Create AWS Account
2. Create IAM user/role that can access S3
3. Create AMI Image.
4. Store data in S3

I will not go into detail on how to create an AWS Account, IAM role or launching an EC2 instance, but I will outline creating the AMI Image.

#### Create AMI

1. Launch a micro(t2) Instance in EC2 and use the Amazon Linux AMI. Select a security group that will allow you to access the Jupyter notebook.
2. SSH into the Instance.
3. Perform updates (sudo yum update)
4. Install git (sudo yum -y git)
5. Install Docker (sudo yum -u docker)
6. Change the user settings (sudo usermod -a -G docker ec2-user)
7. Install docker compose. Update with the latest version (sudo curl -L https://github.com/docker/compose/releases/download/1.9.0/docker-compose-`uname -s`-`uname -m` | sudo tee /usr/local/bin/docker-compose > /dev/null )
8. Update the folder settings (sudo chmodchmod +x /usr/local/bin/docker-compose)
9. Start the service (sudo service docker start)
10. Turn the config on (sudo chkconfig docker on)
11. In the EC2 console, create an Image (Actions>>Image>>Create Image)

Now when you need to launch a new instance, you can build off of this image which will already have git, docker and docker-compose installed.

## Steps to Launch

1. Create a git repo in github
2. Clone the repo on your development computer
3. Add the Dockerfile and docker-compose Files
4. Add any additional libraries to be installed in the Dockerfile
5. Add the .env file and add in the project name, and the AWS credentials
6. Add the .pem file to the folder for easy access
7. Commit the dockerfile and docker-compose files to git. DO NOT commit the .env file or the .pem file.
8. Launch an EC2 instance using the AMI built previously
9. Execute command: "git clone <repo>" to clone the repo
10. Create a .env file on the instance ("touch .env", then "nano .env")
11. Execute command "docker-compose up". Then docker will download the neccessary files and link to jupyter notebook will appear.
12. Copy and paste the link into your browser and change "localhost" with the public IP address of the EC2 instance.

Note: Any notebook created will be in a src folder.


## Code to Access S3

#### JSON

client = boto3.client('s3') <br/><br/>
obj = client.get_object(Bucket='bucket_name', Key='file_name') <br/><br/>
json_data = obj['Body'].read().decode('utf-8') <br/><br/>
data = pd.read_json(json_data, lines=True) <br/><br/>

#### CSV

s3 = boto3.client('s3') <br/><br/>
obj = s3.get_object(Bucket='bucket_name', Key='file_name') <br/><br/>
data = pd.read_csv(io.BytesIO(obj['Body'].read())) <br/><br/>
