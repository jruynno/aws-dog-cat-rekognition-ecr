# Pet Recognition System

### Description

### Features

### Technologies Used

### Architecture


# STAGES
## Stage 1: Create and train the Rekognition model

1. Click this  [1 CLICK DEPLOYMENT](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://learn-cantrill-labs.s3.amazonaws.com/aws-pet-rekognition-ecr/APPCFN.yaml&stackName=PETREKOGNITIONECR)

2. Go to the Rekognition console
![alt text](image.png)
    * On the left menu, locate and select `Use Custom Labels`
    * Click `Get started`
    * If this is your first time creating custom labels, it will ask you to create an `S3 bucket`
        * Click `Create S3 bucket`
    * On the left menu, click on `Projects`
    * Click `Create project`
    * Under `Project name`, Enter `skynet-cats-and-dogs`
    * Click on `Create project`

    * Click `Create a dataset`
    * Select `Start with a single dataset`
    * Select `Upload images from your computer`
    * Click `Create dataset`



3. Download the following zip files 
    * Unzip each file


4. Go back to Rekognition
    * Click `Add images`
    ![alt text](image-1.png)
    * Click `Choose files` to upload
    * Firstly, choose [Cat-Dataset-part1.zip](<Dog and Cats Images/Cat-Dataset-part1.zip>)
    * Select All images inside the folder 
    ![alt text](image-2.png)
    * Click `Open`
    * Click `Upload images (30)`
    ![alt text](image-3.png)
    * Wait for the success dialog to appear before moving to the next step
    ![alt text](image-4.png)
    * Click `Actions`, select `Add images to training dataset`
    ![alt text](image-5.png)
    * Upload the images inside [Cat-Dataset-part2.zip](<Dog and Cats Images/Cat-Dataset-part2.zip>)
    * Do the same process for the rest of the cat images folders
    * Once done, you will be presented with the total of `161` cat images. <br>
        *The screenshot attached is wrong. It should be `161` images*
    ![alt text](image-6.png) 

---

Now we will start labeling <br>
* Click on `Start labeling`
![alt text](image-7.png)
* Click `Add labels` <br>
*The screenshot attached is wrong. It should be `161` images*
![alt text](image-8.png)
* Select `Add labels`
* Enter `cat`, and `dog`
* Click on `Save`
![alt text](image-9.png)
* Select each images
* Click `Assign image-level label`
![alt text](image-10.png)
* Enter `cat`, and 
* Click `Asisgn`
![alt text](image-11.png)
* Move to the next page, and do the same process. Do this to all `161 images`
* Once done, Click `Save changes`
![alt text](image-12.png)

Follow the same process of uploading with `dog images`
* Once done uploading the `160 images`, Select `Unlabeled`
* Click `Start labeling` <br>
*Follow the same process of labeling with dog images. Except label them with `dog`*
![alt text](image-13.png)
![alt text](image-14.png)
* Check that you have `0 Unlabeled images`
![alt text](image-15.png)


* Now, Click on `Train model`, and Confirm that process
![alt text](image-16.png)

![alt text](image-17.png)


## STAGE 2- Create the ECR repository and build the Docker image

1. Go to `Elastic Container Registry`
![alt text](image-18.png)

* On the left menu, Select `Create repository`
* Enter `skynet-repo` for the repo name
* Click `Create`
![alt text](image-19.png)

* Note the URI of the repository. Mine is: `861276109659.dkr.ecr.us-east-1.amazonaws.com/skynet-repo`
![alt text](image-20.png)

2. Open EC2 in a new tab
![alt text](image-21.png)
* On the left menu, click on `Instances`
* Select `AWS-EC2-Docker`, right click `connect`
* On `Session Manager`,  Click `connect`

In the terminal window, enter the following commands:

    sudo amazon-linux-extras install docker -y
    sudo service docker start
    sudo usermod -a -G docker ec2-user
    sudo su - ec2-user
    unzip app.zip

* Next we are going to build and push the Docker image to the ECR repository. Enter the following commands replacing the <AWS Account ID> with the ID of your AWS account you copied in the previous step (you can find it in the upper-right corner of the AWS console):

    * Build the Docker image: `docker build -t skynet .`
    * Tag the Docker image: `docker tag skynet:latest <AWS Account ID>.dkr.ecr.us-east-1.amazonaws.com/skynet-repo:latest`
    * Login to the ECR repository: `aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS Account ID>.dkr.ecr.us-east-1.amazonaws.com`
    * Push the Docker image to the ECR repository: `docker push <AWS Account ID>.dkr.ecr.us-east-1.amazonaws.com/skynet-repo:latest`<br>

    
3. Go back to the ECR repo

* Click on the repository that you created in the previous step.

    You should see an image in the ECR repository with the "latest" tag under the Image tag column. Click on it to see the details and copy the URI of the image for the next stage. You should have something similar to this:

        <AWS Account ID>.dkr.ecr.us-east-1.amazonaws.com/skynet-repo:latest
![alt text](image-22.png)


## STAGE 3 - Create the ECS cluster, ECS task definition and ECS service

before you begin, we need to start the machine learning model

* Move to `Custom labels` 
* Click on the model name
![alt text](image-23.png)
* Select `Use model`
<br>
copy the ARN. Mine is: `arn:aws:rekognition:us-east-1:861276109659:project/skynet-cats-and-dogs/version/skynet-cats-and-dogs.2025-01-27T21.04.57/1737983097468`
* Click `Start`
![alt text](image-24.png)

1. Create the ECS cluster
* Go to ECS console
![alt text](image-25.png)

* On the left menu, locate and select `clusters`
* Click `create cluster`
* In the cluster name, Enter `SkynetCluster`
* VPC: select `A4L-AWS`
* Subnets: There should be available subnet, select that
* Click `Create`
![alt text](image-26.png)

### 2. Create the ECS task definition
*Before you can continue, you will need to have the stage 1 completed.*

* In the left-hand menu, click on `Task definitions`.

* Click `Create new task definition` button.
![alt text](image-27.png)

* Enter `SkynetTaskDefinition` as the name of the task definition.
---
Expand the `Infrastructure requirements`
* In the `Task size` section change the CPU and Memory values to the following ones:

    * CPU: `.5 vCPU`
    * Memory: `1 GB`

* Use the output ECSRoleName from the Cloudformation stack you deployed for both the `task role` and the `task execution role` sections.
![alt text](image-31.png)
---
Expand the `Container - 1`
* In the container details section enter the following details:

    * Container name: `skynet`
    * Image URI: The one you got in the previous stage

* In the environment variables section add the following:
![alt text](image-28.png)

    * Key: BUCKET_NAME
    * Value: The output `S3BucketName` from the Cloudformation stack you deployed
    ![alt text](image-29.png)
    ---
    * Key: MODEL_ARN
    * Value: The one you got in the first stage
    ![alt text](image-30.png)

    

* In the Monitoring and logging section uncheck `Use log collection`.
![alt text](image-32.png)

* Click `Create`


### 3. Create the ECS service

* In the left-hand menu, click on `Task definitions`.

* Select the `SkynetTaskDefinition` task definition
* click `Deploy` button and 
* then click `Create service`.
![alt text](image-33.png)

* In the existing cluster section select `SkynetCluster`.

* In the computer options section select Launch type and make sure the `FARGATE` launch type is selected.
![alt text](image-34.png)
---
Expand `Deployment configuration`
* Enter `SkynetService` as the service name.
![alt text](image-35.png)
---
Expand `Networking` 
* In the networking section enter the following details:

    * VPC: `A4L-AWS`
    * Subnets: The only subnet that is available
    * Security group: The output "SecurityGroup" from the Cloudformation stack you deployed
    * Public IP: Turned on
    
![alt text](image-36.png)

* Click `Create` button.

## STAGE 4 - Test the application
After 3-5 minutes, you should see the service `SkynetService` with the status `Active`
![alt text](image-37.png)


