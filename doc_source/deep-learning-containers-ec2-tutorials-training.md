# Training<a name="deep-learning-containers-ec2-tutorials-training"></a>

This section shows how to run training on AWS Deep Learning Containers for Amazon EC2 using Apache MXNet \(Incubating\), PyTorch, TensorFlow, and TensorFlow 2\.

For a complete list of Deep Learning Containers, refer to [Deep Learning Containers Images](deep-learning-containers-images.md)\. 

**Note**  
MKL users: Read the [AWS Deep Learning Containers Intel Math Kernel Library \(MKL\) Recommendations](deep-learning-containers-mkl.md) to get the best training or inference performance\.

**Topics**
+ [TensorFlow training](#deep-learning-containers-ec2-tutorials-training-tf)
+ [Apache MXNet \(Incubating\) training](#deep-learning-containers-ec2-tutorials-training-mxnet)
+ [PyTorch training](#deep-learning-containers-ec2-tutorials-training-pytorch)
+ [Amazon S3 Plugin for PyTorch](#deep-learning-containers-ec2-training-pytorch-s3-plugin)
+ [Next steps](#deep-learning-containers-ec2-training-pytorch-next)

## TensorFlow training<a name="deep-learning-containers-ec2-tutorials-training-tf"></a>

After you log into your Amazon EC2 instance, you can run TensorFlow and TensorFlow 2 containers with the following commands\. You must use `nvidia-docker` for GPU images\. 
+ For CPU\-based training, run the following\.

  ```
  $ docker run -it <CPU training container>
  ```
+ For GPU\-based training, run the following\.

  ```
  $ nvidia-docker run -it <GPU training container>
  ```

 The previous command runs the container in interactive mode and provides a shell prompt inside the container\. You can then run the following to import TensorFlow\. 

```
$ python
```

```
>> import tensorflow
```

 Press Ctrl\+D to return to the bash prompt\. Run the following to begin training: 

```
git clone https://github.com/fchollet/keras.git
```

```
$ cd keras
```

```
$ python examples/mnist_cnn.py
```

### Next steps<a name="deep-learning-containers-ec2-training-tf-next"></a>

To learn inference on Amazon EC2 using TensorFlow with Deep Learning Containers, see [TensorFlow Inference](deep-learning-containers-ec2-tutorials-inference.md#deep-learning-containers-ec2-tutorials-inference-tf)\. 

## Apache MXNet \(Incubating\) training<a name="deep-learning-containers-ec2-tutorials-training-mxnet"></a>

To begin training with Apache MXNet \(Incubating\) from your Amazon EC2 instance, run the following command to run the container:
+ For CPU

  ```
  $ docker run -it <CPU training container>
  ```
+ For GPU

  ```
  $ nvidia-docker run -it <GPU training container>
  ```

In the terminal of the container, run the following to begin training\.
+ For CPU

  ```
  $ git clone -b v1.4.x https://github.com/apache/incubator-mxnet.git
  python incubator-mxnet/example/image-classification/train_mnist.py
  ```
+ For GPU

  ```
  $ git clone -b v1.4.x https://github.com/apache/incubator-mxnet.git
  python incubator-mxnet/example/image-classification/train_mnist.py --gpus 0
  ```

### MXNet training with GluonCV<a name="deep-learning-containers-ec2-training-mxnet-gluoncv"></a>

In the terminal of the container, run the following to begin training using GluonCV\. GluonCV v0\.6\.0 is included in the Deep Learning Containers\.
+ For CPU

  ```
  $ git clone -b v0.6.0 https://github.com/dmlc/gluon-cv.git
  python gluon-cv/scripts/classification/cifar/train_cifar10.py --model resnet18_v1b
  ```
+ For GPU

  ```
  $ git clone -b v0.6.0 https://github.com/dmlc/gluon-cv.git
  python gluon-cv/scripts/classification/cifar/train_cifar10.py --num-gpus 1 --model resnet18_v1b
  ```

### Next steps<a name="deep-learning-containers-ec2-training-mxnet-next"></a>

To learn inference on Amazon EC2 using MXNet with Deep Learning Containers, see [Apache MXNet \(Incubating\) Inference ](deep-learning-containers-ec2-tutorials-inference.md#deep-learning-containers-ec2-tutorials-inference-mxnet)\. 

## PyTorch training<a name="deep-learning-containers-ec2-tutorials-training-pytorch"></a>

To begin training with PyTorch from your Amazon EC2 instance, use the following commands to run the container\. You must use **nvidia\-docker** for GPU images\. 
+ For CPU

  ```
  $ docker run -it <CPU training container>
  ```
+ For GPU

  ```
  $ nvidia-docker run -it <GPU training container>
  ```
+ If you have docker\-ce version 19\.03 or later, you can use the \-\-gpus flag with docker:

  ```
  $ docker run -it --gpus <GPU training container>
  ```

 Run the following to begin training\. 
+ For CPU

  ```
  $ git clone https://github.com/pytorch/examples.git
  $ python examples/mnist/main.py --no-cuda
  ```
+ For GPU

  ```
  $ git clone https://github.com/pytorch/examples.git
  $ python examples/mnist/main.py
  ```

### PyTorch distributed GPU training with NVIDIA Apex<a name="deep-learning-containers-ec2-training-pytorch-apex"></a>

NVIDIA Apex is a PyTorch extension with utilities for mixed precision and distributed training\. For more information on the utilities offered with Apex, see the [NVIDIA Apex website](https://nvidia.github.io/apex/)\. Apex is currently supported by Amazon EC2 instances in the following families:
+ [Amazon EC2 P3 Instances](https://aws.amazon.com/ec2/instance-types/p3/)
+ [Amazon EC2 P2 Instances](https://aws.amazon.com/ec2/instance-types/p2/)
+ [Amazon EC2 G4 Instances](https://aws.amazon.com/ec2/instance-types/g4/)
+ [Amazon EC2 G3 Instances](https://aws.amazon.com/ec2/instance-types/g3/)

To begin distributed training using NVIDIA Apex, run the following in the terminal of the GPU training container\. This example requires at least two GPUs on your Amazon EC2 instance to run parallel distributed training\. 

```
$ git clone https://github.com/NVIDIA/apex.git && cd apex
$ python -m torch.distributed.launch --nproc_per_node=2 examples/simple/distributed/distributed_data_parallel.py
```

## Amazon S3 Plugin for PyTorch<a name="deep-learning-containers-ec2-training-pytorch-s3-plugin"></a>

Deep Learning Containers include a plugin that enables you to use data from an Amazon S3 bucket for PyTorch training\.

1. To begin using the Amazon S3 plugin in Deep Learning Containers, check to make sure that your Amazon EC2 instance has full access to Amazon S3\. [Create an IAM role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) that grants Amazon S3 access to an Amazon EC2 instance and attach the role to your instance\. You can use the [AmazonS3FullAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AmazonS3FullAccess$serviceLevelSummary) or [AmazonS3ReadOnlyAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess$serviceLevelSummary) policies\.

1.  Set up your `AWS_REGION` environment variable with the region of your choice\.

   ```
   export AWS_REGION=us-east-1
   ```

1. Use the following commands to run a container that is compatible with the Amazon S3 plugin\. You must use **nvidia\-docker** for GPU images\. 
   + For CPU

     ```
     docker run -it 763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.8.1-cpu-py36-ubuntu18.04-v1.6
     ```
   + For GPU

     ```
     nvidia-docker run -it 763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.8.1-gpu-py36-cu111-ubuntu18.04-v1.7
     ```

1. Run the following to test an example\.

   ```
   git clone https://github.com/aws/amazon-s3-plugin-for-pytorch.git
   cd amazon-s3-plugin-for-pytorch/examples
   python s3_cv_iterable_shuffle_example.py
   ```

For more information and additional examples, see the [Amazon S3 Plugin for PyTorch](https://github.com/aws/amazon-s3-plugin-for-pytorch) repository\.

## Next steps<a name="deep-learning-containers-ec2-training-pytorch-next"></a>

To learn inference on Amazon EC2 using PyTorch with Deep Learning Containers, see [PyTorch Inference ](deep-learning-containers-ec2-tutorials-inference.md#deep-learning-containers-ec2-tutorials-inference-pytorch)\. 