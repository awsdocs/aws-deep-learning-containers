# Training<a name="deep-learning-containers-ecs-tutorials-training"></a>

This section shows how to run training on AWS Deep Learning Containers for Amazon Elastic Container Service using Apache MXNet \(Incubating\), PyTorch, TensorFlow, and TensorFlow 2\.

For a complete list of Deep Learning Containers, refer to [Deep Learning Containers Images](deep-learning-containers-images.md)\. 

**Note**  
MKL users: Read the [AWS Deep Learning Containers Intel Math Kernel Library \(MKL\) Recommendations](deep-learning-containers-mkl.md) to get the best training or inference performance\.

**Important**  
If your account has already created the Amazon ECS service\-linked role, that role is used by default for your service unless you specify a role here\. The service\-linked role is required if your task definition uses the **awsvpc** network mode or if the service is configured to use service discovery\. The role is also required if the service uses an external deployment controller, multiple target groups, or Elastic Inference accelerators in which case you should not specify a role here\. For more information, see [Using Service\-Linked Roles for Amazon ECS](https://docs.aws.amazon.com//AmazonECS/latest/developerguide/using-service-linked-roles.html) in the *Amazon ECS Developer Guide*\.

**Topics**
+ [TensorFlow training](#deep-learning-containers-ecs-tutorials-training-tf)
+ [Apache MXNet \(Incubating\) training](#deep-learning-containers-ecs-tutorials-training-mxnet)
+ [PyTorch training](#deep-learning-containers-ecs-tutorials-training-pytorch)
+ [Amazon S3 Plugin for PyTorch](#deep-learning-containers-ecs-tutorials-pytorch-s3-plugin)
+ [Next steps](#deep-learning-containers-ecs-training-pytorch-next)

## TensorFlow training<a name="deep-learning-containers-ecs-tutorials-training-tf"></a>

Before you can run a task on your ECS cluster, you must register a task definition\. Task definitions are lists of containers grouped together\. The following example uses a sample Docker image that adds training scripts to Deep Learning Containers\. You can use this script with either TensorFlow or TensorFlow 2\. To use it with TensorFlow 2, change the Docker image to a TensorFlow 2 image\.

1. Create a file named `ecs-deep-learning-container-training-taskdef.json` with the following contents\.
   + For CPU

     ```
     {
     	"requiresCompatibilities": [
     		"EC2"
     	],
     	"containerDefinitions": [{
     		"command": [
     			"mkdir -p /test && cd /test && git clone https://github.com/fchollet/keras.git && chmod +x -R /test/ && python keras/examples/mnist_cnn.py"
     		],
     		"entryPoint": [
     			"sh",
     			"-c"
     		],
     		"name": "tensorflow-training-container",
     		"image": "763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-inference:1.15.2-cpu-py36-ubuntu18.04",
     		"memory": 4000,
     		"cpu": 256,
     		"essential": true,
     		"portMappings": [{
     			"containerPort": 80,
     			"protocol": "tcp"
     		}],
     		"logConfiguration": {
     			"logDriver": "awslogs",
     			"options": {
     				"awslogs-group": "awslogs-tf-ecs",
     				"awslogs-region": "us-east-1",
     				"awslogs-stream-prefix": "tf",
     				"awslogs-create-group": "true"
     			}
     		}
     	}],
     	"volumes": [],
     	"networkMode": "bridge",
     	"placementConstraints": [],
     	"family": "TensorFlow"
     }
     ```
   + For GPU

     ```
     {
           "requiresCompatibilities": [
             "EC2"
           ],
           "containerDefinitions": [
             {
         "command": [
                     "mkdir -p /test && cd /test && git clone https://github.com/fchollet/keras.git && chmod +x -R /test/ && python keras/examples/mnist_cnn.py"
                  ],
                  "entryPoint": [
                     "sh",
                     "-c"
                  ],
               "name": "tensorflow-training-container",
               "image": "763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-training:1.15.2-gpu-py37-cu100-ubuntu18.04",
               "memory": 6111,
               "cpu": 256,
               "resourceRequirements" : [{
                 "type" : "GPU",
                 "value" : "1"
               }],
               "essential": true,
               "portMappings": [
                 {
                   "containerPort": 80,
                   "protocol": "tcp"
                 }
               ],
               "logConfiguration": {
                   "logDriver": "awslogs",
                   "options": {
                       "awslogs-group": "awslogs-tf-ecs",
                       "awslogs-region": "us-east-1",
                       "awslogs-stream-prefix": "tf",
           				"awslogs-create-group": "true"
                   }
               }
             }
           ],
           "volumes": [],
           "networkMode": "bridge",
           "placementConstraints": [],
           "family": "tensorflow-training"
         }
     ```

1. Register the task definition\. Note the revision number in the output and use it in the next step\.

   ```
   aws ecs register-task-definition --cli-input-json file://ecs-deep-learning-container-training-taskdef.json
   ```

1. Create a task using the task definition\. You need the revision number from the previous step and the name of the cluster you created during setup

   ```
   aws ecs run-task --cluster ecs-ec2-training-inference --task-definition tf:1
   ```

1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

1. Select the `ecs-ec2-training-inference` cluster\.

1. On the **Cluster** page, choose **Tasks**\.

1. After your task is in a `RUNNING` state, choose the task identifier\.

1. Under **Containers**, expand the container details\.

1. Under **Log Configuration**, choose **View logs in CloudWatch**\. This takes you to the CloudWatch console to view the training progress logs\.

### Next steps<a name="deep-learning-containers-ecs-training-tf-next"></a>

To learn inference on Amazon ECS using TensorFlow with Deep Learning Containers, see [TensorFlow inference](deep-learning-containers-ecs-tutorials-inference.md#deep-learning-containers-ecs-tutorials-inference-tf)\. 

## Apache MXNet \(Incubating\) training<a name="deep-learning-containers-ecs-tutorials-training-mxnet"></a>

Before you can run a task on your Amazon Elastic Container Service cluster, you must register a task definition\. Task definitions are lists of containers grouped together\. The following example uses a sample Docker image that adds training scripts to Deep Learning Containers\.

1. Create a file named `ecs-deep-learning-container-training-taskdef.json` with the following contents\.
   + For CPU

     ```
     {
        "requiresCompatibilities":[
           "EC2"
        ],
        "containerDefinitions":[
           {
              "command":[
                 "git clone -b 1.4 https://github.com/apache/incubator-mxnet.git && python /incubator-mxnet/example/image-classification/train_mnist.py"
              ],
              "entryPoint":[
                 "sh",
                 "-c"
              ],
              "name":"mxnet-training",
              "image":"763104351884.dkr.ecr.us-east-1.amazonaws.com/mxnet-training:1.6.0-cpu-py36-ubuntu16.04",
              "memory":4000,
              "cpu":256,
              "essential":true,
              "portMappings":[
                 {
                    "containerPort":80,
                    "protocol":"tcp"
                 }
              ],
              "logConfiguration":{
                 "logDriver":"awslogs",
                 "options":{
                    "awslogs-group":"/ecs/mxnet-training-cpu",
                    "awslogs-region":"us-east-1",
                    "awslogs-stream-prefix":"mnist",
                    "awslogs-create-group":"true"
                 }
              }
           }
        ],
        "volumes":[
     
        ],
        "networkMode":"bridge",
        "placementConstraints":[
     
        ],
        "family":"mxnet"
     }
     ```
   +  For GPU

     ```
     {
        "requiresCompatibilities":[
           "EC2"
        ],
        "containerDefinitions":[
           {
              "command":[
                 "git clone -b 1.4 https://github.com/apache/incubator-mxnet.git && python /incubator-mxnet/example/image-classification/train_mnist.py --gpus 0"
              ],
              "entryPoint":[
                 "sh",
                 "-c"
              ],
              "name":"mxnet-training",
              "image":"763104351884.dkr.ecr.us-east-1.amazonaws.com/mxnet-training:1.6.0-gpu-py36-cu101-ubuntu16.04",
              "memory":4000,
              "cpu":256,
              "resourceRequirements":[
                 {
                    "type":"GPU",
                    "value":"1"
                 }
              ],
              "essential":true,
              "portMappings":[
                 {
                    "containerPort":80,
                    "protocol":"tcp"
                 }
              ],
              "logConfiguration":{
                 "logDriver":"awslogs",
                 "options":{
                    "awslogs-group":"/ecs/mxnet-training-gpu",
                    "awslogs-region":"us-east-1",
                    "awslogs-stream-prefix":"mnist",
                    "awslogs-create-group":"true"
                 }
              }
           }
        ],
        "volumes":[
     
        ],
        "networkMode":"bridge",
        "placementConstraints":[
     
        ],
        "family":"mxnet-training"
     }
     ```

1. Register the task definition\. Note the revision number in the output and use it in the next step\.

   ```
   aws ecs register-task-definition --cli-input-json file://ecs-deep-learning-container-training-taskdef.json
   ```

1. Create a task using the task definition\. You need the revision number from the previous step\.

   ```
   aws ecs run-task --cluster ecs-ec2-training-inference --task-definition mx:1
   ```

1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

1. Select the `ecs-ec2-training-inference` cluster\.

1. On the **Cluster** page, choose **Tasks**\.

1. After your task is in a `RUNNING` state, choose the task identifier\.

1. Under **Containers**, expand the container details\.

1. Under **Log Configuration**, choose **View logs in CloudWatch**\. This takes you to the CloudWatch console to view the training progress logs\.

### Next steps<a name="deep-learning-containers-ecs-training-mxnet-next"></a>

To learn inference on Amazon ECS using MXNet with Deep Learning Containers, see [Apache MXNet \(Incubating\) inference](deep-learning-containers-ecs-tutorials-inference.md#deep-learning-containers-ecs-tutorials-inference-mxnet)\. 

## PyTorch training<a name="deep-learning-containers-ecs-tutorials-training-pytorch"></a>

Before you can run a task on your Amazon ECS cluster, you must register a task definition\. Task definitions are lists of containers grouped together\. The following example uses a sample Docker image that adds training scripts to Deep Learning Containers\.

1. Create a file named `ecs-deep-learning-container-training-taskdef.json` with the following contents\.
   + For CPU

     ```
     {
        "requiresCompatibilities":[
           "EC2"
        ],
        "containerDefinitions":[
           {
              "command":[
                 "git clone https://github.com/pytorch/examples.git && python examples/mnist/main.py --no-cuda"
              ],
              "entryPoint":[
                 "sh",
                 "-c"
              ],
              "name":"pytorch-training-container",
              "image":"763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.5.1-cpu-py36-ubuntu16.04",
              "memory":4000,
              "cpu":256,
              "essential":true,
              "portMappings":[
                 {
                    "containerPort":80,
                    "protocol":"tcp"
                 }
              ],
              "logConfiguration":{
                 "logDriver":"awslogs",
                 "options":{
                    "awslogs-group":"/ecs/pytorch-training-cpu",
                    "awslogs-region":"us-east-1",
                    "awslogs-stream-prefix":"mnist",
                    "awslogs-create-group":"true"
                 }
              }
           }
        ],
        "volumes":[
     
        ],
        "networkMode":"bridge",
        "placementConstraints":[
     
        ],
        "family":"pytorch"
     }
     ```
   +  For GPU

     ```
     {
           "requiresCompatibilities": [
             "EC2"
           ],
           "containerDefinitions": [
             {
         "command": [
                     "git clone https://github.com/pytorch/examples.git && python examples/mnist/main.py"
                  ],
                  "entryPoint": [
                     "sh",
                     "-c"
                  ],
               "name": "pytorch-training-container",
               "image": "763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.5.1-gpu-py36-cu101-ubuntu16.04",
               "memory": 6111,
               "cpu": 256,
               "resourceRequirements" : [{
                 "type" : "GPU",
                 "value" : "1"
               }],
               "essential": true,
               "portMappings": [
                 {
                   "containerPort": 80,
                   "protocol": "tcp"
                 }
               ],
               "logConfiguration": {
                   "logDriver": "awslogs",
                   "options": {
                       "awslogs-group": "/ecs/pytorch-training-gpu",
                       "awslogs-region": "us-east-1",
                       "awslogs-stream-prefix": "mnist",
                           "awslogs-create-group": "true"
                   }
               }
             }
           ],
           "volumes": [],
           "networkMode": "bridge",
           "placementConstraints": [],
           "family": "pytorch-training"
         }
     ```

1. Register the task definition\. Note the revision number in the output and use it in the next step\.

   ```
   aws ecs register-task-definition --cli-input-json file://ecs-deep-learning-container-training-taskdef.json
   ```

1. Create a task using the task definition\. You need the revision identifier from the previous step\.

   ```
   aws ecs run-task --cluster ecs-ec2-training-inference --task-definition pytorch:1
   ```

1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

1. Select the `ecs-ec2-training-inference` cluster\.

1. On the **Cluster** page, choose **Tasks**\.

1. After your task is in a `RUNNING` state, choose the task identifier\.

1. Under **Containers**, expand the container details\.

1. Under **Log Configuration**, choose **View logs in CloudWatch**\. This takes you to the CloudWatch console to view the training progress logs\.

## Amazon S3 Plugin for PyTorch<a name="deep-learning-containers-ecs-tutorials-pytorch-s3-plugin"></a>

Deep Learning Containers include a plugin that enables you to use data from an Amazon S3 bucket for PyTorch training\.

1. To begin using the Amazon S3 plugin in Amazon ECS, set up your `AWS_REGION` environment variable with the region of your choice\.

   ```
   export AWS_REGION=us-east-1
   ```

1. Create a file named `ecs-deep-learning-container-pytorch-s3-plugin-taskdef.json` with the following contents\.
   + For CPU

     ```
     {
        "requiresCompatibilities":[
           "EC2"
        ],
        "containerDefinitions":[
           {
              "command":[
                 "git clone https://github.com/aws/amazon-s3-plugin-for-pytorch.git && python amazon-s3-plugin-for-pytorch/examples/s3_imagenet_example.py"
              ],
              "entryPoint":[
                 "sh",
                 "-c"
              ],
              "name":"pytorch-s3-plugin-container",
              "image":"763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.8.1-cpu-py36-ubuntu18.04-v1.6",
              "memory":4000,
              "cpu":256,
              "essential":true,
              "portMappings":[
                 {
                    "containerPort":80,
                    "protocol":"tcp"
                 }
              ],
              "logConfiguration":{
                 "logDriver":"awslogs",
                 "options":{
                    "awslogs-group":"/ecs/pytorch-s3-plugin-cpu",
                    "awslogs-region":"us-east-1",
                    "awslogs-stream-prefix":"imagenet",
                    "awslogs-create-group":"true"
                 }
              }
           }
        ],
        "volumes":[
     
        ],
        "networkMode":"bridge",
        "placementConstraints":[
     
        ],
        "family":"pytorch-s3-plugin"
     }
     ```
   +  For GPU

     ```
     {
           "requiresCompatibilities": [
             "EC2"
           ],
           "containerDefinitions": [
             {
         "command": [
                     "git clone https://github.com/aws/amazon-s3-plugin-for-pytorch.git && python amazon-s3-plugin-for-pytorch/examples/s3_imagenet_example.py"
                  ],
                  "entryPoint": [
                     "sh",
                     "-c"
                  ],
               "name": "pytorch-s3-plugin-container",
               "image": "763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.8.1-gpu-py36-cu111-ubuntu18.04-v1.7",
               "memory": 6111,
               "cpu": 256,
               "resourceRequirements" : [{
                 "type" : "GPU",
                 "value" : "1"
               }],
               "essential": true,
               "portMappings": [
                 {
                   "containerPort": 80,
                   "protocol": "tcp"
                 }
               ],
               "logConfiguration": {
                   "logDriver": "awslogs",
                   "options": {
                       "awslogs-group": "/ecs/pytorch-s3-plugin-gpu",
                       "awslogs-region": "us-east-1",
                       "awslogs-stream-prefix": "imagenet",
                           "awslogs-create-group": "true"
                   }
               }
             }
           ],
           "volumes": [],
           "networkMode": "bridge",
           "placementConstraints": [],
           "family": "pytorch-s3-plugin"
         }
     ```

1. Register the task definition\. Note the revision number in the output and use it in the next step\.

   ```
   aws ecs register-task-definition --cli-input-json file://ecs-deep-learning-container-pytorch-s3-plugin-taskdef.json
   ```

1. Create a task using the task definition\. You need the revision identifier from the previous step\.

   ```
   aws ecs run-task --cluster ecs-pytorch-s3-plugin --task-definition pytorch-s3-plugin:1
   ```

1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

1. Select the `ecs-pytorch-s3-plugin` cluster\.

1. On the **Cluster** page, choose **Tasks**\.

1. After your task is in a `RUNNING` state, choose the task identifier\.

1. Under **Containers**, expand the container details\.

1. Under **Log Configuration**, choose **View logs in CloudWatch**\. This takes you to the CloudWatch console to view the Amazon S3 plugin example logs\.

For more information and additional examples, see the [Amazon S3 Plugin for PyTorch](https://github.com/aws/amazon-s3-plugin-for-pytorch) repository\.

## Next steps<a name="deep-learning-containers-ecs-training-pytorch-next"></a>

To learn inference on Amazon ECS using PyTorch with Deep Learning Containers, see [PyTorch inference](deep-learning-containers-ecs-tutorials-inference.md#deep-learning-containers-ecs-tutorials-inference-pytorch)\. 