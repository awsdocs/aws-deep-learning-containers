# Inference<a name="deep-learning-containers-ec2-tutorials-inference"></a>

This section shows how to run inference on AWS Deep Learning Containers for Amazon Elastic Compute Cloud using Apache MXNet \(Incubating\), PyTorch, TensorFlow, and TensorFlow 2\. You can also use Elastic Inference to run inference with AWS Deep Learning Containers\. For tutorials and more information on Elastic Inference, see [Using AWS Deep Learning Containers with Elastic Inference on Amazon EC2](https://docs.aws.amazon.com/elastic-inference/latest/developerguide/ei-dlc-ec2.html)\.

For a complete list of Deep Learning Containers, refer to [Deep Learning Containers Images](deep-learning-containers-images.md)\. 

**Note**  
MKL users: read the [AWS Deep Learning Containers Intel Math Kernel Library \(MKL\) Recommendations](deep-learning-containers-mkl.md) to get the best training or inference performance\.

**Topics**
+ [TensorFlow Inference](#deep-learning-containers-ec2-tutorials-inference-tf)
+ [TensorFlow 2 Inference](#deep-learning-containers-ec2-tutorials-inference-tf2)
+ [Apache MXNet \(Incubating\) Inference](#deep-learning-containers-ec2-tutorials-inference-mxnet)
+ [PyTorch Inference](#deep-learning-containers-ec2-tutorials-inference-pytorch)

## TensorFlow Inference<a name="deep-learning-containers-ec2-tutorials-inference-tf"></a>

To demonstrate how to use Deep Learning Containers for inference, this example uses a simple *half plus two* model with TensorFlow Serving\. We recommend using the [Deep Learning Base AMI](https://docs.aws.amazon.com/dlami/latest/devguide/overview-base.html) for TensorFlow\. After you log into your instance, run the following:

```
$ git clone -b r1.15 https://github.com/tensorflow/serving.git
$ cd serving
$ git checkout r1.15
```

Use the commands here to start TensorFlow Serving with the Deep Learning Containers for this model\. Unlike the Deep Learning Containers for training, model serving starts immediately upon running the container and runs as a background process\.
+ For CPU instances:

  ```
  $ docker run -p 8500:8500 -p 8501:8501 --name tensorflow-inference --mount type=bind,source=$(pwd)/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_cpu,target=/models/saved_model_half_plus_two -e MODEL_NAME=saved_model_half_plus_two -d <cpu inference container>
  ```

  For example:

  ```
  $ docker run -p 8500:8500 -p 8501:8501 --name tensorflow-inference --mount type=bind,source=$(pwd)/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_cpu,target=/models/saved_model_half_plus_two -e MODEL_NAME=saved_model_half_plus_two -d 763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-inference:1.15.0-cpu-py36-ubuntu18.04
  ```
+ For GPU instances:

  ```
  $ nvidia-docker run -p 8500:8500 -p 8501:8501 --name tensorflow-inference --mount type=bind,source=$(pwd)/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_gpu,target=/models/saved_model_half_plus_two -e MODEL_NAME=saved_model_half_plus_two -d <gpu inference container>
  ```

  For example:

  ```
  $ nvidia-docker run -p 8500:8500 -p 8501:8501 --name tensorflow-inference --mount type=bind,source=$(pwd)/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_gpu,target=/models/saved_model_half_plus_two -e MODEL_NAME=saved_model_half_plus_two -d 763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-inference:1.15.0-gpu-py36-cu100-ubuntu18.04
  ```
+ For Inf1 instances:

  ```
  $ docker run -id --name tensorflow-inference -p 8500:8500 --device=/dev/neuron0  --cap-add IPC_LOCK  --mount type=bind,source={model_path},target=/models/{model_name}  -e MODEL_NAME={model_name} <neuron inference container>
  ```

  For example:

  ```
  $ docker run -id --name tensorflow-inference -p 8500:8500 --device=/dev/neuron0  --cap-add IPC_LOCK  --mount type=bind,source={model_path},target=/models/{model_name}  -e MODEL_NAME={model_name} 763104351884.dkr.ecr.us-west-2.amazonaws.com/tensorflow-inference-neuron:1.15.4-neuron-py37-ubuntu18.04-v1.1
  ```

Next, run inference with the Deep Learning Containers\.

```
$ curl -d '{"instances": [1.0, 2.0, 5.0]}' -X POST http://127.0.0.1:8501/v1/models/saved_model_half_plus_two:predict
```

The output is similar to the following:

```
{
    "predictions": [2.5, 3.0, 4.5
    ]
}
```

**Note**  
If you want to debug the container's output, you can attach to it using the container name, as in the following command:  

```
$ docker attach <your docker container name>
```
In this example you used `tensorflow-inference`\.

## TensorFlow 2 Inference<a name="deep-learning-containers-ec2-tutorials-inference-tf2"></a>

To demonstrate how to use Deep Learning Containers for inference, this example uses a simple *half plus two* model with TensorFlow 2 Serving\. We recommend using the [Deep Learning Base AMI](https://docs.aws.amazon.com//dlami/latest/devguide/overview-base.html) for TensorFlow 2\. After you log into your instance run the following\.

```
$ git clone -b r2.0 https://github.com/tensorflow/serving.git
$ cd serving
```

Use the commands here to start TensorFlow Serving with the Deep Learning Containers for this model\. Unlike the Deep Learning Containers for training, model serving starts immediately upon running the container and runs as a background process\.
+ For CPU instances:

  ```
  $ docker run -p 8500:8500 -p 8501:8501 --name tensorflow-inference --mount type=bind,source=$(pwd)/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_cpu,target=/models/saved_model_half_plus_two -e MODEL_NAME=saved_model_half_plus_two -d <cpu inference container>
  ```

  For example:

  ```
  $ docker run -p 8500:8500 -p 8501:8501 --name tensorflow-inference --mount type=bind,source=$(pwd)/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_cpu,target=/models/saved_model_half_plus_two -e MODEL_NAME=saved_model_half_plus_two -d 763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-inference:2.0.0-cpu-py36-ubuntu18.04
  ```
+ For GPU instances:

  ```
  $ nvidia-docker run -p 8500:8500 -p 8501:8501 --name tensorflow-inference --mount type=bind,source=$(pwd)/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_gpu,target=/models/saved_model_half_plus_two -e MODEL_NAME=saved_model_half_plus_two -d <gpu inference container>
  ```

  For example:

  ```
  $ nvidia-docker run -p 8500:8500 -p 8501:8501 --name tensorflow-inference --mount type=bind,source=$(pwd)/tensorflow_serving/servables/tensorflow/testdata/saved_model_half_plus_two_gpu,target=/models/saved_model_half_plus_two -e MODEL_NAME=saved_model_half_plus_two -d 763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-inference:2.0.0-gpu-py36-cu100-ubuntu18.04
  ```
**Note**  
Loading the GPU model server may take some time\.

Next, run inference with the Deep Learning Containers\.

```
$ curl -d '{"instances": [1.0, 2.0, 5.0]}' -X POST http://127.0.0.1:8501/v1/models/saved_model_half_plus_two:predict
```

The output is similar to the following\.

```
{
    "predictions": [2.5, 3.0, 4.5
    ]
}
```

**Note**  
To debug the container's output, you can use the name to attach to it as shown in the following command:  

```
$ docker attach <your docker container name>
```
This example used `tensorflow-inference`\.

## Apache MXNet \(Incubating\) Inference<a name="deep-learning-containers-ec2-tutorials-inference-mxnet"></a>

To begin inference with Apache MXNet \(Incubating\), this example uses a pretrained model from a public S3 bucket\.

For CPU instances, run the following command\.

```
$ docker run -it --name mms -p 80:8080  -p 8081:8081 <your container image id> \
mxnet-model-server --start --mms-config /home/model-server/config.properties \
--models squeezenet=https://s3.amazonaws.com/model-server/models/squeezenet_v1.1/squeezenet_v1.1.model
```

For GPU instances, run the following command:

```
$ nvidia-docker run -it --name mms -p 80:8080  -p 8081:8081 <your container image id> \
mxnet-model-server --start --mms-config /home/model-server/config.properties \
--models squeezenet=https://s3.amazonaws.com/model-server/models/squeezenet_v1.1/squeezenet_v1.1.model
```

The configuration file is included in the container\.

With your server started, you can now run inference from a different window by using the following command\.

```
$ curl -O https://s3.amazonaws.com/model-server/inputs/kitten.jpg
curl -X POST http://127.0.0.1/predictions/squeezenet -T kitten.jpg
```

After you are done using your container, you can remove it using the following command:

```
$ docker rm -f mms
```

### MXNet Inference with GluonCV<a name="deep-learning-containers-ec2-tutorials-inference-mxnet-gluoncv"></a>

To begin inference using GluonCV, this example uses a pretrained model from a public S3 bucket\.

For CPU instances, run the following command\.

```
$ docker run -it --name mms -p 80:8080  -p 8081:8081 <your container image id> \
mxnet-model-server --start --mms-config /home/model-server/config.properties \
--models gluoncv_yolo3=https://dlc-samples.s3.amazonaws.com/mxnet/gluon/gluoncv_yolo3.mar
```

For GPU instances, run the following command\.

```
$ nvidia-docker run -it --name mms -p 80:8080  -p 8081:8081 <your container image id> \
mxnet-model-server --start --mms-config /home/model-server/config.properties \
--models gluoncv_yolo3=https://dlc-samples.s3.amazonaws.com/mxnet/gluon/gluoncv_yolo3.mar
```

The configuration file is included in the container\.

With your server started, you can now run inference from a different window by using the following command\.

```
$ curl -O https://dlc-samples.s3.amazonaws.com/mxnet/gluon/dog.jpg
curl -X POST http://127.0.0.1/predictions/gluoncv_yolo3/predict -T dog.jpg
```

Your output should look like the following:

```
{
  "bicycle": [
    "[ 79.674225  87.403786 409.43515  323.12167 ]",
    "[ 98.69891  107.480446 200.0086   155.13412 ]"
  ],
  "car": [
    "[336.61322   56.533463 499.30566  125.0233  ]"
  ],
  "dog": [
    "[100.50538 156.50375 223.014   384.60873]"
  ]
}
```

After you are done using your container, you can remove it using this command\.

```
$ docker rm -f mms
```

## PyTorch Inference<a name="deep-learning-containers-ec2-tutorials-inference-pytorch"></a>

Deep Learning Containers with PyTorch version 1\.6 and later use TorchServe for inference calls\. Deep Learning Containers with PyTorch version 1\.5 and earlier use `mxnet-model-server` for inference calls\.

### PyTorch 1\.6 and later<a name="dlc-ec2-tutorials-inference-pytorch-1.6"></a>

To run inference with PyTorch, this example uses a model pretrained on Imagenet from a public S3 bucket\. Inference is served using TorchServe\. For more information, see this blog on [Deploying PyTorch inference with TorchServe](https://aws.amazon.com/blogs/machine-learning/serving-pytorch-models-in-production-with-the-amazon-sagemaker-native-torchserve-integration/)\.

For CPU instances:

```
$ docker run -itd --name torchserve -p 80:8080  -p 8081:8081 <your container image id> \
torchserve --start --ts-config /home/model-server/config.properties \
--models pytorch-densenet=https://torchserve.s3.amazonaws.com/mar_files/densenet161.mar
```

For GPU instances

```
$ nvidia-docker run -itd --name torchserve -p 80:8080  -p 8081:8081 <your container image id> \
torchserve --start --ts-config /home/model-server/config.properties \
--models pytorch-densenet=https://torchserve.s3.amazonaws.com/mar_files/densenet161.mar
```

If you have docker\-ce version 19\.03 or later, you can use the `--gpus` flag when you start Docker\.

The configuration file is included in the container\.

With your server started, you can now run inference from a different window by using the following\.

```
$ curl -O https://s3.amazonaws.com/model-server/inputs/flower.jpg
curl -X POST http://127.0.0.1:80/predictions/pytorch-densenet -T flower.jpg
```

After you are done using your container, you can remove it using the following\.

```
$ docker rm -f torchserve
```

### PyTorch 1\.5 and earlier<a name="dlc-ec2-tutorials-inference-pytorch-1.5"></a>

To run inference with PyTorch, this example uses a model pretrained on Imagenet from a public S3 bucket\. Similar to MXNet containers, inference is served using mxnet\-model\-server, which can support any framework as the backend\. For more information, see [Model Server for Apache MXNet](https://github.com/awslabs/mxnet-model-server) and this blog on [Deploying PyTorch inference with MXNet Model Server](https://aws.amazon.com/blogs/machine-learning/deploying-pytorch-inference-with-mxnet-model-server/)\.

For CPU instances:

```
$ docker run -itd --name mms -p 80:8080  -p 8081:8081 <your container image id> \
mxnet-model-server --start --mms-config /home/model-server/config.properties \
--models densenet=https://dlc-samples.s3.amazonaws.com/pytorch/multi-model-server/densenet/densenet.mar
```

For GPU instances

```
$ nvidia-docker run -itd --name mms -p 80:8080  -p 8081:8081 <your container image id> \
mxnet-model-server --start --mms-config /home/model-server/config.properties \
--models densenet=https://dlc-samples.s3.amazonaws.com/pytorch/multi-model-server/densenet/densenet.mar
```

If you have docker\-ce version 19\.03 or later, you can use the `--gpus` flag when you start Docker\.

The configuration file is included in the container\.

With your server started, you can now run inference from a different window by using the following\.

```
$ curl -O https://s3.amazonaws.com/model-server/inputs/flower.jpg
curl -X POST http://127.0.0.1/predictions/densenet -T flower.jpg
```

After you are done using your container, you can remove it using the following\.

```
$ docker rm -f mms
```

### Next steps<a name="deep-learning-containers-ecs-tutorials-inference-next"></a>

To learn about using custom entrypoints with Deep Learning Containers on Amazon ECS, see [Custom entrypoints](deep-learning-containers-ecs-tutorials-custom-entry.md)\. 