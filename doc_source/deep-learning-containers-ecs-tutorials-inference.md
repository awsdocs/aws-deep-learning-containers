# Inference<a name="deep-learning-containers-ecs-tutorials-inference"></a>

This section shows how to run inference on AWS Deep Learning Containers for Amazon Elastic Container Service \(Amazon ECS\) using Apache MXNet \(Incubating\), PyTorch, TensorFlow, and TensorFlow 2\. You can also use Elastic Inference to run inference with AWS Deep Learning Containers\. For tutorials and more information on Elastic Inference, see [Using AWS Deep Learning Containers with Elastic Inference on Amazon ECS](https://docs.aws.amazon.com//elastic-inference/latest/developerguide/ei-dlc-ecs.html)\.

For a complete list of Deep Learning Containers, see [Deep Learning Containers Images](deep-learning-containers-images.md)\. 

**Note**  
MKL users: Read the [AWS Deep Learning Containers Intel Math Kernel Library \(MKL\) Recommendations](deep-learning-containers-mkl.md) to get the best training or inference performance\.

**Important**  
If your account has already created the Amazon ECS service\-linked role, then that role is used by default for your service unless you specify a role here\. The service\-linked role is required if your task definition uses the **awsvpc** network mode\. The role is also required if the service is configured to use service discovery, an external deployment controller, multiple target groups, or Elastic Inference accelerators in which case you should not specify a role here\. For more information, see [Using Service\-Linked Roles for Amazon ECS](https://docs.aws.amazon.com//AmazonECS/latest/developerguide/using-service-linked-roles.html) in the *Amazon ECS Developer Guide*\.

**Topics**
+ [TensorFlow inference](#deep-learning-containers-ecs-tutorials-inference-tf)
+ [Apache MXNet \(Incubating\) inference](#deep-learning-containers-ecs-tutorials-inference-mxnet)
+ [PyTorch inference](#deep-learning-containers-ecs-tutorials-inference-pytorch)

## TensorFlow inference<a name="deep-learning-containers-ecs-tutorials-inference-tf"></a>

The following examples use a sample Docker image that adds either CPU or GPU inference scripts to Deep Learning Containers from your host machine's command line\.

### CPU\-based inference<a name="deep-learning-containers-ecs-tutorials-inference-tf-cpu"></a>

Use the following example to run CPU\-based inference\.

1. Create a file named `ecs-dlc-cpu-inference-taskdef.json` with the following contents\. You can use this with either TensorFlow or TensorFlow 2\. To use it with TensorFlow 2, change the Docker image to a TensorFlow 2 image and clone the r2\.0 serving repository branch instead of r1\.15\.

   ```
   {
   	"requiresCompatibilities": [
   		"EC2"
   	],
   	"containerDefinitions": [{
   		"command": [
   			"mkdir -p /test && cd /test && git clone -b r1.15 https://github.com/tensorflow/serving.git && tensorflow_model_server --port=8500 --rest_api_port=8501 --model_name=saved_model_half_plus_two --model_base_path=/test/serving/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_cpu"
   		],
   		"entryPoint": [
   			"sh",
   			"-c"
   		],
   		"name": "tensorflow-inference-container",
   		"image": "763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-inference:1.15.0-cpu-py36-ubuntu18.04",
   		"memory": 8111,
   		"cpu": 256,
   		"essential": true,
   		"portMappings": [{
   				"hostPort": 8500,
   				"protocol": "tcp",
   				"containerPort": 8500
   			},
   			{
   				"hostPort": 8501,
   				"protocol": "tcp",
   				"containerPort": 8501
   			},
   			{
   				"containerPort": 80,
   				"protocol": "tcp"
   			}
   		],
   		"logConfiguration": {
   			"logDriver": "awslogs",
   			"options": {
   				"awslogs-group": "/ecs/tensorflow-inference-gpu",
   				"awslogs-region": "us-east-1",
   				"awslogs-stream-prefix": "half-plus-two",
   				"awslogs-create-group": "true"
   			}
   		}
   	}],
   	"volumes": [],
   	"networkMode": "bridge",
   	"placementConstraints": [],
   	"family": "tensorflow-inference"
   }
   ```

1. Register the task definition\. Note the revision number in the output and use it in the next step\.

   ```
   aws ecs register-task-definition --cli-input-json file://ecs-dlc-cpu-inference-taskdef.json
   ```

1. Create an Amazon ECS service\. When you specify the task definition, replace `revision_id` with the revision number of the task definition from the output of the previous step\.

   ```
   aws ecs create-service --cluster ecs-ec2-training-inference \
                          --service-name cli-ec2-inference-cpu \
                          --task-definition Ec2TFInference:revision_id \
                          --desired-count 1 \
                          --launch-type EC2 \
                          --scheduling-strategy="REPLICA" \
                          --region us-east-1
   ```

1. Verify the service and get the network endpoint by completing the following steps\.

   1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

   1. Select the `ecs-ec2-training-inference` cluster\.

   1. On the **Cluster** page, choose **Services** and then **cli\-ec2\-inference\-cpu**\.

   1. After your task is in a `RUNNING` state, choose the task identifier\.

   1. Under **Containers**, expand the container details\.

   1. Under **Name** and then **Network Bindings**, under **External Link** note the IP address for port 8501 and use it in the next step\.

   1. Under **Log Configuration**, choose **View logs in CloudWatch**\. This takes you to the CloudWatch console to view the training progress logs\.

1. To run inference, use the following command\. Replace the external IP address with the external link IP address from the previous step\.

   ```
   curl -d '{"instances": [1.0, 2.0, 5.0]}' -X POST http://<External ip>:8501/v1/models/saved_model_half_plus_two:predict
   ```

   The following is sample output\.

   ```
   {
       "predictions": [2.5, 3.0, 4.5
       ]
   }
   ```
**Important**  
If you are unable to connect to the external IP address, be sure that your corporate firewall is not blocking non\-standards ports, like 8501\. You can try switching to a guest network to verify\.

### GPU\-based inference<a name="deep-learning-containers-ecs-tutorials-inference-tf-gpu"></a>

 Use the following example to run GPU\-based inference\.

1. Create a file named `ecs-dlc-gpu-inference-taskdef.json` with the following contents\. You can use this with either TensorFlow or TensorFlow 2\. To use it with TensorFlow 2, change the Docker image to a TensorFlow 2 image and clone the r2\.0 serving repository branch instead of r1\.15\.

   ```
   {
   	"requiresCompatibilities": [
   		"EC2"
   	],
   	"containerDefinitions": [{
   		"command": [
   			"mkdir -p /test && cd /test && git clone -b r1.15 https://github.com/tensorflow/serving.git && tensorflow_model_server --port=8500 --rest_api_port=8501 --model_name=saved_model_half_plus_two --model_base_path=/test/serving/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_gpu"
   		],
   		"entryPoint": [
   			"sh",
   			"-c"
   		],
   		"name": "tensorflow-inference-container",
   		"image": "763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-inference:1.15.0-gpu-py36-cu100-ubuntu18.04",
   		"memory": 8111,
   		"cpu": 256,
   		"resourceRequirements": [{
   			"type": "GPU",
   			"value": "1"
   		}],
   		"essential": true,
   		"portMappings": [{
   				"hostPort": 8500,
   				"protocol": "tcp",
   				"containerPort": 8500
   			},
   			{
   				"hostPort": 8501,
   				"protocol": "tcp",
   				"containerPort": 8501
   			},
   			{
   				"containerPort": 80,
   				"protocol": "tcp"
   			}
   		],
   		"logConfiguration": {
   			"logDriver": "awslogs",
   			"options": {
   				"awslogs-group": "/ecs/TFInference",
   				"awslogs-region": "us-east-1",
   				"awslogs-stream-prefix": "ecs",
   				"awslogs-create-group": "true"
   			}
   		}
   	}],
   	"volumes": [],
   	"networkMode": "bridge",
   	"placementConstraints": [],
   	"family": "TensorFlowInference"
   }
   ```

1. Register the task definition\. Note the revision number in the output and use it in the next step\.

   ```
   aws ecs register-task-definition --cli-input-json file://ecs-dlc-gpu-inference-taskdef.json
   ```

1. Create an Amazon ECS service\. When you specify the task definition, replace `revision_id` with the revision number of the task definition from the output of the previous step\.

   ```
   aws ecs create-service --cluster ecs-ec2-training-inference \
                          --service-name cli-ec2-inference-gpu \
                          --task-definition Ec2TFInference:revision_id \
                          --desired-count 1 \
                          --launch-type EC2 \
                          --scheduling-strategy="REPLICA" \
                          --region us-east-1
   ```

1. Verify the service and get the network endpoint by completing the following steps\.

   1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

   1. Select the `ecs-ec2-training-inference` cluster\.

   1. On the **Cluster** page, choose **Services** and then **cli\-ec2\-inference\-cpu**\.

   1. After your task is in a `RUNNING` state, choose the task identifier\.

   1. Under **Containers**, expand the container details\.

   1. Under **Name** and then** Network Bindings**, under **External Link** note the IP address for port 8501 and use it in the next step\.

   1. Under **Log Configuration**, choose **View logs in CloudWatch**\. This takes you to the CloudWatch console to view the training progress logs\.

1. To run inference, use the following command\. Replace the external IP address with the external link IP address from the previous step\.

   ```
   curl -d '{"instances": [1.0, 2.0, 5.0]}' -X POST http://<External ip>:8501/v1/models/saved_model_half_plus_two:predict
   ```

   The following is sample output\.

   ```
   {
       "predictions": [2.5, 3.0, 4.5
       ]
   }
   ```
**Important**  
If you are unable to connect to the external IP address, be sure that your corporate firewall is not blocking non\-standards ports, like 8501\. You can try switching to a guest network to verify\.

## Apache MXNet \(Incubating\) inference<a name="deep-learning-containers-ecs-tutorials-inference-mxnet"></a>

Before you can run a task on your Amazon ECS cluster, you must register a task definition\. Task definitions are lists of containers grouped together\. The following examples use a sample Docker image that adds either CPU or GPU inference scripts to Deep Learning Containers from your host machine's command line\.

### CPU\-based inference<a name="deep-learning-containers-ecs-tutorials-inference-mxnet-cpu"></a>

 Use the following task definition to run CPU\-based inference\.

1. Create a file named `ecs-dlc-cpu-inference-taskdef.json` with the following contents\.

   ```
   {
   	"requiresCompatibilities": [
   		"EC2"
   	],
   	"containerDefinitions": [{
   		"command": [
   			"mxnet-model-server --start --mms-config /home/model-server/config.properties --models squeezenet=https://s3.amazonaws.com/model-server/models/squeezenet_v1.1/squeezenet_v1.1.model"
   		],
   		"name": "mxnet-inference-container",
   		"image": "763104351884.dkr.ecr.us-east-1.amazonaws.com/mxnet-inference:1.6.0-cpu-py36-ubuntu16.04",
   		"memory": 8111,
   		"cpu": 256,
   		"essential": true,
   		"portMappings": [{
   				"hostPort": 8081,
   				"protocol": "tcp",
   				"containerPort": 8081
   			},
   			{
   				"hostPort": 80,
   				"protocol": "tcp",
   				"containerPort": 8080
   			}
   		],
   		"logConfiguration": {
   			"logDriver": "awslogs",
   			"options": {
   				"awslogs-group": "/ecs/mxnet-inference-cpu",
   				"awslogs-region": "us-east-1",
   				"awslogs-stream-prefix": "squeezenet",
   				"awslogs-create-group": "true"
   			}
   		}
   	}],
   	"volumes": [],
   	"networkMode": "bridge",
   	"placementConstraints": [],
   	"family": "mxnet-inference"
   }
   ```

1. Register the task definition\. Note the revision number in the output and use it in the next step\.

   ```
   aws ecs register-task-definition --cli-input-json file://ecs-dlc-cpu-inference-taskdef.json
   ```

1. Create an Amazon ECS service\. When you specify the task definition, replace `revision_id` with the revision number of the task definition from the output of the previous step\.

   ```
   aws ecs create-service --cluster ecs-ec2-training-inference \
                          --service-name cli-ec2-inference-cpu \
                          --task-definition Ec2TFInference:revision_id \
                          --desired-count 1 \
                          --launch-type EC2 \
                          --scheduling-strategy REPLICA \
                          --region us-east-1
   ```

1. Verify the service and get the endpoint\.

   1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

   1. Select the `ecs-ec2-training-inference` cluster\.

   1. On the **Cluster** page, choose **Services** and then **cli\-ec2\-inference\-cpu**\.

   1. After your task is in a `RUNNING` state, choose the task identifier\.

   1. Under **Containers**, expand the container details\.

   1. Under **Name** and then **Network Bindings**, under **External Link** note the IP address for port 8081 and use it in the next step\.

   1. Under **Log Configuration**, choose **View logs in CloudWatch**\. This takes you to the CloudWatch console to view the training progress logs\.

1. To run inference, use the following command\. Replace the `external IP` address with the external link IP address from the previous step\.

   ```
   curl -O https://s3.amazonaws.com/model-server/inputs/kitten.jpg
   curl -X POST http://<External ip>/predictions/squeezenet -T kitten.jpg
   ```

   The following is sample output\.

   ```
   [
     {
       "probability": 0.8582226634025574,
       "class": "n02124075 Egyptian cat"
     },
     {
       "probability": 0.09160050004720688,
       "class": "n02123045 tabby, tabby cat"
     },
     {
       "probability": 0.037487514317035675,
       "class": "n02123159 tiger cat"
     },
     {
       "probability": 0.0061649843119084835,
       "class": "n02128385 leopard, Panthera pardus"
     },
     {
       "probability": 0.003171598305925727,
       "class": "n02127052 lynx, catamount"
     }
   ]
   ```
**Important**  
If you are unable to connect to the external IP address, be sure that your corporate firewall is not blocking non\-standards ports, like 8081\. You can try switching to a guest network to verify\.

### GPU\-based inference<a name="deep-learning-containers-ecs-tutorials-inference-mxnet-gpu"></a>

 Use the following task definition to run GPU\-based inference\.

```
{
	"requiresCompatibilities": [
		"EC2"
	],
	"containerDefinitions": [{
		"command": [
			"mxnet-model-server --start --mms-config /home/model-server/config.properties --models  squeezenet=https://s3.amazonaws.com/model-server/models/squeezenet_v1.1/squeezenet_v1.1.model"
		],
		"name": "mxnet-inference-container",
		"image": "763104351884.dkr.ecr.us-east-1.amazonaws.com/mxnet-inference:1.6.0-gpu-py36-cu101-ubuntu16.04",
		"memory": 8111,
		"cpu": 256,
		"resourceRequirements": [{
			"type": "GPU",
			"value": "1"
		}],
		"essential": true,
		"portMappings": [{
				"hostPort": 8081,
				"protocol": "tcp",
				"containerPort": 8081
			},
			{
				"hostPort": 80,
				"protocol": "tcp",
				"containerPort": 8080
			}
		],
		"logConfiguration": {
			"logDriver": "awslogs",
			"options": {
				"awslogs-group": "/ecs/mxnet-inference-gpu",
				"awslogs-region": "us-east-1",
				"awslogs-stream-prefix": "squeezenet",
				"awslogs-create-group": "true"
			}
		}
	}],
	"volumes": [],
	"networkMode": "bridge",
	"placementConstraints": [],
	"family": "mxnet-inference"
}
```

1. Use the following command to register the task definition\. Note the output of the revision number and use it in the next step\.

   ```
   aws ecs register-task-definition --cli-input-json file://<Task definition file>
   ```

1. To create the service, replace the `revision_id` with the output from the previous step in the following command\.

   ```
   aws ecs create-service --cluster ecs-ec2-training-inference \
                       --service-name cli-ec2-inference-gpu \
                       --task-definition Ec2TFInference:<revision_id> \
                       --desired-count 1 \
                       --launch-type "EC2" \
                       --scheduling-strategy REPLICA \
                       --region us-east-1
   ```

1. Verify the service and get the endpoint\.

   1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

   1. Select the `ecs-ec2-training-inference` cluster\.

   1. On the **Cluster** page, choose **Services** and then **cli\-ec2\-inference\-cpu**\.

   1. After your task is in a `RUNNING` state, choose the task identifier\.

   1. Under **Containers**, expand the container details\.

   1. Under **Name** and then **Network Bindings**, under **External Link** note the IP address for port 8081 and use it in the next step\.

   1. Under **Log Configuration**, choose **View logs in CloudWatch**\. This takes you to the CloudWatch console to view the training progress logs\.

1. To run inference, use the following command\. Replace the `external IP` address with the external link IP address from the previous step\.

   ```
   curl -O https://s3.amazonaws.com/model-server/inputs/kitten.jpg
   curl -X POST http://<External ip>/predictions/squeezenet -T kitten.jpg
   ```

   The following is sample output\.

   ```
   [
     {
       "probability": 0.8582226634025574,
       "class": "n02124075 Egyptian cat"
     },
     {
       "probability": 0.09160050004720688,
       "class": "n02123045 tabby, tabby cat"
     },
     {
       "probability": 0.037487514317035675,
       "class": "n02123159 tiger cat"
     },
     {
       "probability": 0.0061649843119084835,
       "class": "n02128385 leopard, Panthera pardus"
     },
     {
       "probability": 0.003171598305925727,
       "class": "n02127052 lynx, catamount"
     }
   ]
   ```
**Important**  
If you are unable to connect to the external IP address, be sure that your corporate firewall is not blocking non\-standards ports, like 8081\. You can try switching to a guest network to verify\.

## PyTorch inference<a name="deep-learning-containers-ecs-tutorials-inference-pytorch"></a>

Before you can run a task on your Amazon ECS cluster, you must register a task definition\. Task definitions are lists of containers grouped together\. The following examples use a sample Docker image that adds either CPU or GPU inference scripts to Deep Learning Containers\.

### CPU\-based inference<a name="deep-learning-containers-ecs-tutorials-inference-mxnet-cpu"></a>

 Use the following task definition to run CPU\-based inference\.

1. Create a file named `ecs-dlc-cpu-inference-taskdef.json` with the following contents\.

   ```
   {
       "requiresCompatibilities": [
           "EC2"
       ],
       "containerDefinitions": [{
           "command": [
               "mxnet-model-server --start --mms-config /home/model-server/config.properties --models densenet=https://dlc-samples.s3.amazonaws.com/pytorch/multi-model-server/densenet/densenet.mar"
           ],
           "name": "pytorch-inference-container",
           "image": "763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-inference:1.3.1-cpu-py36-ubuntu16.04",
           "memory": 8111,
           "cpu": 256,
           "essential": true,
           "portMappings": [{
                   "hostPort": 8081,
                   "protocol": "tcp",
                   "containerPort": 8081
               },
               {
                   "hostPort": 80,
                   "protocol": "tcp",
                   "containerPort": 8080
               }
           ],
           "logConfiguration": {
               "logDriver": "awslogs",
               "options": {
                   "awslogs-group": "/ecs/densenet-inference-cpu",
                   "awslogs-region": "us-east-1",
                   "awslogs-stream-prefix": "densenet",
                   "awslogs-create-group": "true"
               }
           }
       }],
       "volumes": [],
       "networkMode": "bridge",
       "placementConstraints": [],
       "family": "pytorch-inference"
   }
   ```

1. Register the task definition\. Note the revision number in the output and use it in the next step\.

   ```
   aws ecs register-task-definition --cli-input-json file://ecs-dlc-cpu-inference-taskdef.json
   ```

1. Create an Amazon ECS service\. When you specify the task definition, replace `revision_id` with the revision number of the task definition from the output of the previous step\.

   ```
   aws ecs create-service --cluster ecs-ec2-training-inference \
                          --service-name cli-ec2-inference-cpu \
                          --task-definition Ec2PTInference:revision_id \
                          --desired-count 1 \
                          --launch-type EC2 \
                          --scheduling-strategy REPLICA \
                          --region us-east-1
   ```

1. Verify the service and get the network endpoint by completing the following steps\.

   1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

   1. Select the `ecs-ec2-training-inference` cluster\.

   1. On the **Cluster** page, choose **Services** and then **cli\-ec2\-inference\-cpu**\.

   1. After your task is in a `RUNNING` state, choose the task identifiter\.

   1. Under **Containers**, expand the container details\.

   1. Under **Name** and then **Network Bindings**, under **External Link** note the IP address for port 8081 and use it in the next step\.

   1. Under **Log Configuration**, choose **View logs in CloudWatch**\. This takes you to the CloudWatch console to view the training progress logs\.

1. To run inference, use the following command\. Replace the `external IP` address with the external link IP address from the previous step\.

   ```
   curl -O https://s3.amazonaws.com/model-server/inputs/flower.jpg
   curl -X POST http://<External ip>/predictions/densenet -T flower.jpg
   ```
**Important**  
If you are unable to connect to the external IP address, be sure that your corporate firewall is not blocking non\-standards ports, like 8081\. You can try switching to a guest network to verify\.

### GPU\-based inference<a name="deep-learning-containers-ecs-tutorials-inference-mxnet-gpu"></a>

 Use the following task definition to run GPU\-based inference\.

```
{
    "requiresCompatibilities": [
        "EC2"
    ],
    "containerDefinitions": [{
        "command": [
            "mxnet-model-server --start --mms-config /home/model-server/config.properties --models densenet=https://dlc-samples.s3.amazonaws.com/pytorch/multi-model-server/densenet/densenet.mar"
        ],
        "name": "pytorch-inference-container",
        "image": "763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-inference:1.3.1-gpu-py36-cu101-ubuntu16.04",
        "memory": 8111,
        "cpu": 256,
        "essential": true,
        "portMappings": [{
                "hostPort": 8081,
                "protocol": "tcp",
                "containerPort": 8081
            },
            {
                "hostPort": 80,
                "protocol": "tcp",
                "containerPort": 8080
            }
        ],
        "logConfiguration": {
            "logDriver": "awslogs",
            "options": {
                "awslogs-group": "/ecs/densenet-inference-cpu",
                "awslogs-region": "us-east-1",
                "awslogs-stream-prefix": "densenet",
                "awslogs-create-group": "true"
            }
        }
    }],
    "volumes": [],
    "networkMode": "bridge",
    "placementConstraints": [],
    "family": "pytorch-inference"
}
```

1. Use the following command to register the task definition\. Note the output of the revision number and use it in the next step\.

   ```
   aws ecs register-task-definition --cli-input-json file://<Task definition file>
   ```

1. To create the service, replace the `revision_id` with the output from the previous step in the following command\.

   ```
   aws ecs create-service --cluster ecs-ec2-training-inference \
                       --service-name cli-ec2-inference-gpu \
                       --task-definition Ec2PTInference:<revision_id> \
                       --desired-count 1 \
                       --launch-type "EC2" \
                       --scheduling-strategy REPLICA \
                       --region us-east-1
   ```

1. Verify the service and get the network endpoint by completing the following steps\.

   1. Open the Amazon ECS console at [https://console\.aws\.amazon\.com/ecs/](https://console.aws.amazon.com/ecs/)\.

   1. Select the `ecs-ec2-training-inference` cluster\.

   1. On the **Cluster** page, choose **Services** and then **cli\-ec2\-inference\-cpu**\.

   1. Once your task is in a `RUNNING` state, choose the task identifier\.

   1. Under **Containers**, expand the container details\.

   1. Under **Name** and then **Network Bindings**, under **External Link** note the IP address for port 8081 and use it in the next step\.

   1. Under **Log Configuration**, choose **View logs in CloudWatch**\. This takes you to the CloudWatch console to view the training progress logs\.

1. To run inference, use the following command\. Replace the `external IP` address with the external link IP address from the previous step\.

   ```
   curl -O https://s3.amazonaws.com/model-server/inputs/flower.jpg
   curl -X POST http://<External ip>/predictions/densenet -T flower.jpg
   ```
**Important**  
If you are unable to connect to the external IP address, be sure that your corporate firewall is not blocking non\-standards ports, like 8081\. You can try switching to a guest network to verify\.

### Next steps<a name="deep-learning-containers-ecs-tutorials-inference-next"></a>

To learn about using Custom Entrypoints with Deep Learning Containers on Amazon ECS, see [Custom entrypoints](deep-learning-containers-ecs-tutorials-custom-entry.md)\. 