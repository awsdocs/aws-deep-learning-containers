# What are AWS Deep Learning Containers?<a name="what-is-dlc"></a>

Welcome to the User Guide for the AWS Deep Learning Containers\. 

AWS Deep Learning Containers \(Deep Learning Containers\) are a set of Docker images for training and serving models in TensorFlow, TensorFlow 2, PyTorch, and Apache MXNet \(Incubating\)\. Deep Learning Containers provide optimized environments with TensorFlow and MXNet, Nvidia CUDA \(for GPU instances\), and Intel MKL \(for CPU instances\) libraries and are available in the Amazon Elastic Container Registry \(Amazon ECR\)\.

## About this guide<a name="guide-contents"></a>

This guide helps you set up and use AWS Deep Learning Containers\. This guide also covers setting up Deep Learning Containers with Amazon EC2, Amazon ECS, Amazon EKS, and SageMaker\. It covers several use cases that are common for deep learning, for both training and inference\. This guide also provides several tutorials for each of the frameworks\.
+ To run training and inference on Deep Learning Containers for Amazon EC2 using MXNet, PyTorch, TensorFlow, and TensorFlow 2, see [Amazon EC2 Tutorials](deep-learning-containers-ec2.md) 
+ To run training and inference on Deep Learning Containers for Amazon ECS using MXNet, PyTorch, and TensorFlow, see [Amazon ECS tutorials](deep-learning-containers-ecs.md)
+ Deep Learning Containers for Amazon EKS offer CPU, GPU, and distributed GPU\-based training, as well as CPU and GPU\-based inference\. To run training and inference on Deep Learning Containers for Amazon EKS using MXNet, PyTorch, and TensorFlow, see [Amazon EKS Tutorials](deep-learning-containers-eks.md)
+ For an explanation of the Docker\-based Deep Learning Containers images, the list of available images, and how to use them, see [Deep Learning Containers Images](deep-learning-containers-images.md)
+ For information on security in Deep Learning Containers, see [Security in AWS Deep Learning Containers](security.md)
+ For a list of the latest Deep Learning Containers release notes, see [Release Notes for Deep Learning Containers](dlc-release-notes.md)

## Python 2 Support<a name="python2-deprecation"></a>

The Python open source community has officially ended support for Python 2 on January 1, 2020\. The TensorFlow and PyTorch community have announced that the TensorFlow 2\.1 and PyTorch 1\.4 releases will be the last ones supporting Python 2\. Previous releases of the Deep Learning Containers that support Python 2 will continue to be available\. However, we will provide updates to the Python 2 Deep Learning Containers only if there are security fixes published by the open source community for those versions\. Deep Learning Containers releases with the next versions of the TensorFlow and PyTorch frameworks will not include the Python 2 environments\.

## Prerequisites<a name="prerequisites"></a>

You should be familiar with command line tools and basic Python to successfully run the Deep Learning Containers\. Tutorials on how to use each framework are provided by the frameworks themselves\. However, this guide shows you how to activate each one and find the appropriate tutorials to get started\. 