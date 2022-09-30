# Amazon ECS setup<a name="deep-learning-containers-ecs-setup"></a>

This topic shows how to setup AWS Deep Learning Containers with Amazon Elastic Container Service\.

**Topics**
+ [Prerequisites](#deep-learning-containers-ecs-setup-prerequisites)
+ [Setting up Amazon ECS for Deep Learning Containers](#deep-learning-containers-ecs-setting-up-ecs)

## Prerequisites<a name="deep-learning-containers-ecs-setup-prerequisites"></a>

This setup guide assumes that you have completed the following prerequisites:
+ Install and configure the latest version of the AWS CLI\. For more information about installing or upgrading the AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)\.
+ Complete the steps in [Setting Up with Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/get-set-up-for-amazon-ecs.html)\.
+ One of the following is true:
  + Your user has administrator access\. For more information, see [Setting Up with Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/get-set-up-for-amazon-ecs.html)\.
  + Your user has the IAM permissions to create a service role\. For more information, see [Creating a Role to Delegate Permissions to an AWS Service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html)\.
  + A user with administrator access has manually created these IAM roles so that they're available on the account to be used\. For more information, see [Amazon ECS Service Scheduler IAM Role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service_IAM_role.html) and [Amazon ECS Container Instance IAM Role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/instance_IAM_role.html) in the *Amazon Elastic Container Service Developer Guide*\.
+ The Amazon CloudWatch Logs IAM policy is added to the Amazon ECS Container Instance IAM role, which allows Amazon ECS to send logs to Amazon CloudWatch\. For more information, see [CloudWatch Logs IAM Policy](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_cloudwatch_logs.html) in the *Amazon Elastic Container Service Developer Guide*\.
+ Generate a key pair\. For more information see [Amazon EC2 Key Pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)\. 
+ Create a new security group or update an existing security group to have the ports open for your desired inference server\.
  + For MXNet inference, ports 80 and 8081 open to TCP traffic\.
  + For TensorFlow inference, ports 8501 and 8500 open to TCP traffic\.

  For more information see [Amazon EC2 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)\.

## Setting up Amazon ECS for Deep Learning Containers<a name="deep-learning-containers-ecs-setting-up-ecs"></a>

This section explains how to set up Amazon ECS to use Deep Learning Containers\. 

**Important**  
If your account has already created the Amazon ECS service\-linked role, then that role is used by default for your service unless you specify a role here\. The service\-linked role is required if your task definition uses the **awsvpc** network mode or if the service is configured to use any of the following: Service discovery, an external deployment controller, multiple target groups, or Elastic Inference accelerators\. If this is the case, you should not specify a role here\. For more information, see [Using Service\-Linked Roles for Amazon ECS](https://docs.aws.amazon.com//AmazonECS/latest/developerguide/using-service-linked-roles.html) in the *Amazon ECS Developer Guide*\.

Run the following actions from your host\.

1. Create an Amazon ECS cluster in the Region that contains the key pair and security group that you created previously\.

   ```
   aws ecs create-cluster --cluster-name ecs-ec2-training-inference --region us-east-1
   ```

1. Launch one or more Amazon EC2 instances into your cluster\. For GPU\-based work, refer to [Working with GPUs on Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-gpu.html) in the *Amazon ECS Developer Guide* to inform your instance type selection\. If you select a GPU instance type, be sure to then choose the Amazon ECS GPU\-optimized AMI\. For CPU\-based work, you can use the Amazon Linux or Amazon Linux 2 ECS\-optimized AMIs\. For more information about compatible instance types and Amazon ECS\-optimized AMI IDs, see [Amazon ECS\-optimized AMIs](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-optimized_AMI.html)\. In this example, you launch one instance with a GPU\-based AMI with 100 GB of disk size in us\-east\-1\. 

   1. Create a file named `my_script.txt` with the following contents\. Reference the same cluster name that you created in the previous step\.

      ```
      #!/bin/bash
      echo ECS_CLUSTER=ecs-ec2-training-inference >> /etc/ecs/ecs.config
      ```

   1. \(Optional\) Create a file named `my_mapping.txt` with the following content, which changes the size of the root volume after the instance is created\.

      ```
      [
          {
              "DeviceName": "/dev/xvda",
              "Ebs": {
                  "VolumeSize": 100
              }
          }
      ]
      ```

   1. Launch an Amazon EC2 instance with the Amazon ECS\-optimized AMI and attach it to the cluster\. Use the security group ID and key pair name that you created and replace them in the following command\. To get the latest Amazon ECS\-optimized AMI ID, see [Amazon ECS\-optimized AMIs](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-optimized_AMI.html) in the *Amazon Elastic Container Service Developer Guide*\.

      ```
      aws ec2 run-instances --image-id ami-0dfdeb4b6d47a87a2 \
                             --count 1 \
                             --instance-type p2.8xlarge \
                             --key-name key-pair-1234 \
                             --security-group-ids sg-abcd1234 \
                             --iam-instance-profile Name="ecsInstanceRole" \
                             --user-data file://my_script.txt \
                             --block-device-mapping file://my_mapping.txt \
                             --region us-east-1
      ```

      In the Amazon EC2 console, you can verify that this step was successful by the `instance-id` from the response\.

You now have an Amazon ECS cluster with container instances running\. Verify that the Amazon EC2 instances are registered with the cluster with the following steps\.

**To verify that the Amazon EC2 instance is registered with the cluster**

1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

1. Select the cluster with your registered Amazon EC2 instances\.

1. On the **Cluster** page, choose **ECS Instances**\.

1. Verify that the **Agent Connected** value is **True** for the `instance-id` created in previous step\. Also, note the [CPU available and memory available from the console](https://console.aws.amazon.com/ecs/home?#/clusters/ecs-ec2-training-inference/containerInstances) as these values can be useful in the following tutorials\. It might take a few minutes to appear in the console\.

### Next steps<a name="deep-learning-containers-ecs-tutorials-training-next"></a>

To learn about training and inference with Deep Learning Containers on Amazon ECS, see [Amazon ECS tutorials](deep-learning-containers-ecs.md)\. 