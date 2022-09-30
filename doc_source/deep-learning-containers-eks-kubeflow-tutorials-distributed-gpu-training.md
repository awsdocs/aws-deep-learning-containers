# Distributed GPU Training<a name="deep-learning-containers-eks-kubeflow-tutorials-distributed-gpu-training"></a>

This section is for distributed training on GPU\-based clusters\. 

Make sure that your cluster has GPU nodes before you run the examples\. If you do not have GPU nodes in your cluster, use the following command to add a nodegroup to your cluster\. Be sure to select an [Amazon EC2 instance](https://aws.amazon.com/ec2/instance-types/) \(`node-type`\) in the Accelerated Computing category\. 

```
$ eksctl create nodegroup --cluster $CLUSTER_NAME --region $CLUSTER_REGION \
      --nodes 2 --nodes-min 1 --nodes-max 3 --node-type p3.2xlarge
```

For a complete list of Deep Learning Containers, see [Deep Learning Containers Images](deep-learning-containers-images.md)\. 

**Topics**
+ [PyTorch distributed GPU training](#deep-learning-containers-eks-kubeflow-tutorials-distributed-gpu-training-pytorch)
+ [TensorFlow with Horovod distributed GPU training](#deep-learning-containers-eks-kubeflow-tutorials-distributed-gpu-training-tf)

## PyTorch distributed GPU training<a name="deep-learning-containers-eks-kubeflow-tutorials-distributed-gpu-training-pytorch"></a>

 This tutorial guides you through training a classification model on [MNIST with PyTorch](https://github.com/kubeflow/training-operator/tree/master/examples/pytorch/mnist) in a single node GPU instance running a container from Deep Learning Containers managed by Kubeflow on AWS\. The example uses Gloo as the backend\.

1.  Create a PyTorchJob\.

   1. Verify that the PyTorch custom resource is installed\.

      ```
      $ kubectl get crd
      ```

      The output should include `pytorchjobs.kubeflow.org`\.

   1. Ensure that the NVIDIA plugin `daemonset` is running\.

      ```
      $ kubectl get daemonset -n kube-system
      ```

      The output should look similar to the following\.

      ```
      NDESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE SELECTOR AGE
      nvidia-device-plugin-daemonset   3         3      3        3        3    <none>  35h
      ```

   1.  Use the following text to create a gloo\-based distributed data parallel job\. Save it in a file named `pt_distributed.yaml`\. 

      ```
      apiVersion: kubeflow.org/v1
      kind: PyTorchJob
      metadata:
        name: "kubeflow-pytorch-gpu-dist-job"
      spec:
        pytorchReplicaSpecs:
          Master:
            replicas: 1
            restartPolicy: OnFailure
            template:
              metadata:
                annotations:
                  sidecar.istio.io/inject: "false"
              spec:
                containers:
                - name: "pytorch"
                  image: "763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-pytorch-training:1.12.0-gpu-py38-cu116-ubuntu20.04-ec2-example-v1.0-2022-07-01-00-40-08"
                  args:
                    - "--backend"
                    - "gloo"
                    - "--epochs"
                    - "5"
          Worker:
            replicas: 2
            restartPolicy: OnFailure
            template:
              metadata:
                annotations:
                  sidecar.istio.io/inject: "false"
              spec:
                containers:
                - name: "pytorch"
                  image: "763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-pytorch-training:1.12.0-gpu-py38-cu116-ubuntu20.04-ec2-example-v1.0-2022-07-01-00-40-08"
                  args:
                    - "--backend"
                    - "gloo"
                    - "--epochs"
                    - "5"
                  resources:
                    limits:
                      nvidia.com/gpu: 1
      ```

   1. Run a distributed training job\.

      ```
      $ kubectl create -f pt_distributed.yaml -n ${NAMESPACE}
      ```

1. Monitor your PyTorchJob\.

   1. See the status section to monitor the job status\. Here is an example of output when the job is successfully completed\.

      ```
      $ kubectl get -o yaml pytorchjobs kubeflow-pytorch-gpu-dist-job ${NAMESPACE} 
      ```

   1. Check the logs for each pod\.

      The first command prints a list of pods for a specific PyTorchJob, as shown in the following example\.

      ```
      $ kubectl get pods -l job-name=kubeflow-pytorch-gpu-dist-job -o name -n ${NAMESPACE}
      ```

       The second command tails the logs for a specific pod\.

      ```
      $ kubectl logs pod name -n ${NAMESPACE}
      ```

See [Cleanup](deep-learning-containers-eks-kubeflow-setup.md#deep-learning-containers-eks-kubeflow-cleanup) for information about cleaning up a cluster after you finish using it\. 

## TensorFlow with Horovod distributed GPU training<a name="deep-learning-containers-eks-kubeflow-tutorials-distributed-gpu-training-tf"></a>

This tutorial guides you through distributed training with [Horovod](https://github.com/horovod/horovod) TensorFlow on a GPU cluster\. You will run this [distributed training example on ImageNet with ResNet based on TensorFlow](https://github.com/aws-samples/deep-learning-models/blob/master/legacy/models/resnet/tensorflow/train_imagenet_resnet_hvd.py) in your container from Deep Learning Containers, managed by Kubeflow on AWS\. 

The example requires a GPU instance with at least 2 GPUs\. You can use `node-type=p3.16xlarge` or above\. 

1.  Create an MPIJob\.

   1. Verify that the TensorFlow custom resource is installed\.

      ```
      $ kubectl get crd
      ```

      The output should include `mpijobs.kubeflow.org`\.

   1. Ensure that the NVIDIA plugin `daemonset` is running\.

      ```
      $ kubectl get daemonset -n kube-system
      ```

      The output should look similar to the following\.

      ```
      NDESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE SELECTOR AGE
      nvidia-device-plugin-daemonset   3         3      3        3        3    <none>  35h
      ```

   1.  Use the following text to create an MPIJob\. Save it in a file named `tf_distributed.yaml.`\. 

      ```
      apiVersion: kubeflow.org/v1
      kind: MPIJob
      metadata:
        name: tensorflow-tf-dist
      spec:
        slotsPerWorker: 1
        cleanPodPolicy: Running
        mpiReplicaSpecs:
          Launcher:
            replicas: 1
            template:
               metadata:
                 annotations:
                   sidecar.istio.io/inject: "false"
               spec:
                 containers:
                 - image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-tensorflow-training:2.9.1-gpu-py39-cu112-ubuntu20.04-ec2-example-v1.2-2022-06-09-20-07-24
                   name: tensorflow-launcher
                   command:
                    - mpirun
                    - -mca
                    - btl_tcp_if_exclude
                    - lo
                    - -mca
                    - pml
                    - ob1
                    - -mca
                    - btl
                    - ^openib
                    - --bind-to
                    - none
                    - -map-by
                    - slot
                    - -x
                    - LD_LIBRARY_PATH
                    - -x
                    - PATH
                    - -x
                    - NCCL_SOCKET_IFNAME=eth0
                    - -x
                    - NCCL_DEBUG=INFO
                    - -x
                    - MXNET_CUDNN_AUTOTUNE_DEFAULT=0
                    - python
                    - /deep-learning-models/models/resnet/tensorflow2/train_tf2_resnet.py
                   args:
                    - --num_epochs
                    - "10"
                    - --synthetic
          Worker:
            replicas: 2
            template:
              metadata:
                annotations:
                  sidecar.istio.io/inject: "false"
              spec:
                containers:
                - image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-tensorflow-training:2.9.1-gpu-py39-cu112-ubuntu20.04-ec2-example-v1.2-2022-06-09-20-07-24
                  name: tensorflow-worker
                  resources:
                    limits:
                      nvidia.com/gpu: 1
      ```

   1. Run a distributed training job\.

      ```
      $ kubectl create -f tf_distributed.yaml -n ${NAMESPACE}
      ```

1. Monitor your PyTorchJob\.

   1. See the status section to monitor the job status\. Here is an example of output after the job is successfully completed\.

      ```
      $ kubectl get -o yaml mpijob tensorflow-tf-dist -n ${NAMESPACE}
      ```

   1. Check the logs for each pod\.

      The first command prints a list of pods for a specific PyTorchJob, such as the following example\.

      ```
      $ kubectl get -o yaml mpijob tensorflow-tf-dist -n ${NAMESPACE}
      ```

       The second command tails the logs for a specific pod\.

      ```
      $ kubectl logs pod name -n ${NAMESPACE}
      ```

See [Cleanup](deep-learning-containers-eks-kubeflow-setup.md#deep-learning-containers-eks-kubeflow-cleanup) for information about cleaning up a cluster after you finish using it\. 