# Distributed GPU Training<a name="deep-learning-containers-eks-tutorials-distributed-gpu-training"></a>

This section is for running distributed training on multi\-node GPU clusters\.

For a complete list of Deep Learning Containers, refer to [Deep Learning Containers Images](deep-learning-containers-images.md)\. 

**Topics**
+ [Set up your cluster for distributed training](#deep-learning-containers-eks-tutorials-distributed-gpu-setup)
+ [Apache MXNet \(Incubating\) distributed GPU training](#deep-learning-containers-eks-tutorials-distributed-gpu-training-mxnet)
+ [Apache MXNet \(Incubating\) with Horovod distributed GPU training](#deep-learning-containers-eks-tutorials-distributed-gpu-training-mxnet)
+ [TensorFlow with Horovod distributed GPU training](#deep-learning-containers-eks-tutorials-distributed-gpu-training-tf)
+ [PyTorch distributed GPU training](#deep-learning-containers-eks-tutorials-distributed-gpu-training-pytorch)
+ [Amazon S3 Plugin for PyTorch](#deep-learning-containers-eks-tutorials-distributed-gpu-pytorch-s3-plugin)

## Set up your cluster for distributed training<a name="deep-learning-containers-eks-tutorials-distributed-gpu-setup"></a>

To run distributed training on EKS, you need the following components installed on your cluster\. 
+ The default installation of [Kubeflow](https://www.kubeflow.org/docs/aws/deploy/install-kubeflow/) with required components, such as PyTorch operators, TensorFlow operators, and the NVIDIA plugin\.
+ Apache MXNet and MPI operators\.

Download and run the script to install the required components in the cluster\.

```
$ wget -O install_kubeflow.sh https://raw.githubusercontent.com/aws/deep-learning-containers/master/test/dlc_tests/eks/eks_manifest_templates/kubeflow/install_kubeflow.sh
$ chmod +x install_kubeflow.sh
$ ./install_kubeflow.sh <EKS_CLUSTER_NAME> <AWS_REGION>
```

## Apache MXNet \(Incubating\) distributed GPU training<a name="deep-learning-containers-eks-tutorials-distributed-gpu-training-mxnet"></a>

This tutorial shows how to run distributed training with Apache MXNet \(Incubating\) on your multi\-node GPU cluster using Parameter Server\. To run MXNet distributed training on EKS, you use the [Kubernetes MXNet\-operator](https://www.kubeflow.org/docs/components/mxnet/) named `MXJob`\. It provides a custom resource that makes it easy to run distributed or non\-distributed MXNet jobs \(training and tuning\) on Kubernetes\. This operator is installed in the previous setup step\.

Using a Custom Resource Definition \(CRD\) gives users the ability to create and manage MX Jobs just like builtin K8s resources\. Verify that the MXNet custom resource is installed\.

```
$ kubectl get crd
```

The output should include `mxjobs.kubeflow.org`\.

## Running MNIST distributed training with parameter server example

Create a pod file\(mx\_job\_dist\.yaml\) for your job according to the available cluster configuration and job to run\. There are 3 jobModes you need to specify: Scheduler, Server and Worker\. You can specify how many pods you want to spawn with the field replicas\. The instance type of the Scheduler, Server, and Worker will be of the type specified at cluster creation\.
+ Scheduler: There is only one scheduler\. The role of the scheduler is to set up the cluster\. This includes waiting for messages that each node has come up and which port the node is listening on\. The scheduler then lets all processes know about every other node in the cluster, so that they can communicate with each other\.
+ Server: There can be multiple servers which store the model’s parameters, and communicate with workers\. A server may or may not be co\-located with the worker processes\.
+ Worker: A worker node actually performs training on a batch of training samples\. Before processing each batch, the workers pull weights from servers\. The workers also send gradients to the servers after each batch\. Depending on the workload for training a model, it might not be a good idea to run multiple worker processes on the same machine\.
+ Provide container image you want to use with the field image\.
+ You can provide `restartPolicy` from one of the Always, OnFailure and Never\. It determines whether pods will be restarted when they exit or not\.
+ Provide container image you want to use with the field image\.

1. To create a MXJob template, modify the following code block according to your requirements and save it in a file named `mx_job_dist.yaml`\.

   ```
   apiVersion: "kubeflow.org/v1beta1"
   kind: "MXJob"
   metadata:
     name: <JOB_NAME>
   spec:
     jobMode: MXTrain
     mxReplicaSpecs:
       Scheduler:
         replicas: 1
         restartPolicy: Never
         template:
           spec:
             containers:
               - name: mxnet
                 image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-mxnet-training:1.8.0-gpu-py37-cu110-ubuntu16.04-example
       Server:
         replicas: <NUM_SERVERS>
         restartPolicy: Never
         template:
           spec:
             containers:
               - name: mxnet
                 image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-mxnet-training:1.8.0-gpu-py37-cu110-ubuntu16.04-example
       Worker:
         replicas: <NUM_WORKERS>
         restartPolicy: Never
         template:
           spec:
             containers:
               - name: mxnet
                 image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-mxnet-training:1.8.0-gpu-py37-cu110-ubuntu16.04-example
                 command:
                 - "python"
                 args:
                 - "/incubator-mxnet/example/image-classification/train_mnist.py"
                 - "--num-epochs"
                 - <EPOCHS>
                 - "--num-layers"
                 - <LAYERS>
                 - "--kv-store"
                 - "dist_device_sync"
                 - "--gpus"
                 - <GPUS>
                 resources:
                   limits:
                     nvidia.com/gpu: <GPU_LIMIT>
   ```

1. Run distributed training job with the pod file you just created\.

   ```
   $ # Create a job by defining MXJob
   kubectl create -f mx_job_dist.yaml
   ```

1. List the running jobs\.

   ```
   $ kubectl get mxjobs
   ```

1. To get status of a running job, run the following\. Replace the JOB variable with whatever the job's name is\.

   ```
   $ JOB=<JOB_NAME>
   kubectl get mxjobs $JOB -o yaml
   ```

   The output should be similar to the following:

   ```
   apiVersion: kubeflow.org/v1beta1
   kind: MXJob
   metadata:
     creationTimestamp: "2020-07-23T16:38:41Z"
     generation: 8
     name: kubeflow-mxnet-gpu-dist-job-3910
     namespace: mxnet-multi-node-training-3910
     resourceVersion: "688398"
     selfLink: /apis/kubeflow.org/v1beta1/namespaces/mxnet-multi-node-training-3910/mxjobs/kubeflow-mxnet-gpu-dist-job-3910
   spec:
     cleanPodPolicy: All
     jobMode: MXTrain
     mxReplicaSpecs:
       Scheduler:
         replicas: 1
         restartPolicy: Never
         template:
           metadata:
             creationTimestamp: null
           spec:
             containers:
             - image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-mxnet-training:1.8.0-gpu-py37-cu110-ubuntu16.04-example
               name: mxnet
               ports:
               - containerPort: 9091
                 name: mxjob-port
               resources: {}
       Server:
         replicas: 2
         restartPolicy: Never
         template:
           metadata:
             creationTimestamp: null
           spec:
             containers:
             - image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-mxnet-training:1.8.0-gpu-py37-cu110-ubuntu16.04-example
               name: mxnet
               ports:
               - containerPort: 9091
                 name: mxjob-port
               resources: {}
       Worker:
         replicas: 3
         restartPolicy: Never
         template:
           metadata:
             creationTimestamp: null
           spec:
             containers:
             - args:
               - /incubator-mxnet/example/image-classification/train_mnist.py
               - --num-epochs
               - "20"
               - --num-layers
               - "2"
               - --kv-store
               - dist_device_sync
               - --gpus
               - "0"
               command:
               - python
               image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-mxnet-training:1.8.0-gpu-py37-cu110-ubuntu16.04-example
               name: mxnet
               ports:
               - containerPort: 9091
                 name: mxjob-port
               resources:
                 limits:
                   nvidia.com/gpu: "1"
   status:
     conditions:
     - lastTransitionTime: "2020-07-23T16:38:41Z"
       lastUpdateTime: "2020-07-23T16:38:41Z"
       message: MXJob kubeflow-mxnet-gpu-dist-job-3910 is created.
       reason: MXJobCreated
       status: "True"
       type: Created
     - lastTransitionTime: "2020-07-23T16:38:41Z"
       lastUpdateTime: "2020-07-23T16:40:50Z"
       message: MXJob kubeflow-mxnet-gpu-dist-job-3910 is running.
       reason: MXJobRunning
       status: "True"
       type: Running
     mxReplicaStatuses:
       Scheduler:
         active: 1
       Server:
         active: 2
       Worker:
         active: 3
     startTime: "2020-07-23T16:40:50Z"
   ```
**Note**  
Status provides information about the state of the resources\.  
Phase \- Indicates the phase of a job and will be one of Creating, Running, CleanUp, Failed, or Done\.  
State \- Provides the overall status of the job and will be one of Running, Succeeded, or Failed\.

1. If you want to delete a job, change directories to where you launched the job and run the following:

   ```
   $ kubectl delete -f mx_job_dist.yaml
   ```

## Apache MXNet \(Incubating\) with Horovod distributed GPU training<a name="deep-learning-containers-eks-tutorials-distributed-gpu-training-mxnet"></a>

This tutorial shows how to setup distributed training of Apache MXNet \(Incubating\) models on your multi\-node GPU cluster that uses [Horovod](https://github.com/horovod/horovod)\. It uses an example image that already has a training script included, and it uses a 3\-node cluster with `node-type=p3.8xlarge`\. This tutorial runs the [Horovod example script](https://github.com/horovod/horovod/blob/master/examples/mxnet/mxnet_mnist.py) for MXNet on an MNIST model\.

1. Verify that the MPIJob custom resource is installed\.

   ```
   $ kubectl get crd
   ```

   The output should include `mpijobs.kubeflow.org`\.

1. Create a MPI Job template and define the number of nodes \(replicas\) and number of GPUs each node has \(gpusPerReplica\)\. Modify the following code block according to your requirements and save it in a file named `mx-mnist-horovod-job.yaml`\. 

   ```
   apiVersion: kubeflow.org/v1alpha2
   kind: MPIJob
   metadata:
     name: <JOB_NAME>
   spec:
     slotsPerWorker: 1 
     cleanPodPolicy: Running
     mpiReplicaSpecs:
       Launcher:
         replicas: 1
         template:
           spec:
             containers:
             - image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-mxnet-training:1.8.0-gpu-py37-cu110-ubuntu16.04-example
               name: <JOB_NAME>
               args:
               - --epochs
               - "10"
               - --lr
               - "0.001"
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
               - /horovod/examples/mxnet_mnist.py
       Worker:
         replicas: <NUM_WORKERS>
         template:
           spec:
             containers:
             - image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-mxnet-training:1.8.0-gpu-py37-cu110-ubuntu16.04-example
               name: mpi-worker
               resources:
                 limits:
                   nvidia.com/gpu: <GPUS>
   ```

1. Run the distributed training job with the pod file you just created\.

   ```
   $ kubectl create -f mx-mnist-horovod-job.yaml
   ```

1. Check the status\. The name of the job appears in the status\. If you're running any other tests or have previously run something, it appears in this list\. Run this several times until you see the status change to “Running”\.

   ```
   $ kubectl get pods -o wide
   ```

   You should see something similar to the following output:

   ```
   NAME                                        READY   STATUS   RESTARTS AGE
   
   mxnet-mnist-horovod-job-716-launcher-4wc7f   1/1    Running    0       31s
   mxnet-mnist-horovod-job-716-worker-0         1/1    Running    0       31s
   mxnet-mnist-horovod-job-716-worker-1         1/1    Running    0       31s
   mxnet-mnist-horovod-job-716-worker-2         1/1    Running    0       31s
   ```

1. Based on the name of the launcher pod above, check the logs to see the training output\.

   ```
   $ kubectl logs -f --tail 10 <LAUNCHER_POD_NAME>
   ```

1. You can check the logs to watch the training progress\. You can also continue to check “get pods” to refresh the status\. When the status changes to “Completed” you will know that the training job is done\.

1. To clean up and rerun a job:

   ```
   $ kubectl delete -f mx-mnist-horovod-job.yaml
   ```

### Next steps<a name="deep-learning-containers-eks-training-distributed-gpu-mxnet-next"></a>

To learn GPU\-based inference on Amazon EKS using MXNet with Deep Learning Containers, see [Apache MXNet \(Incubating\) GPU inference](deep-learning-containers-eks-tutorials-gpu-inference.md#deep-learning-containers-eks-tutorials-gpu-inference-mxnet)\. 

## TensorFlow with Horovod distributed GPU training<a name="deep-learning-containers-eks-tutorials-distributed-gpu-training-tf"></a>

This tutorial shows how to setup distributed training of TensorFlow models on your multi\-node GPU cluster that uses [Horovod](https://github.com/horovod/horovod)\. It uses an example image that already has a training script included, and it uses a 3\-node cluster with `node-type=p3.16xlarge`\. You can use this tutorial with either TensorFlow or TensorFlow 2\. To use it with TensorFlow 2, change the Docker image to a TensorFlow 2 image\.

1. Verify that the MPIJob custom resource is installed\.

   ```
   $ kubectl get crd
   ```

   The output should include `mpijobs.kubeflow.org`\.

1. Create a MPI Job template and define the number of nodes \(replicas\) and number of GPUs each node has \(gpusPerReplica\)\. Modify the following code block according to your requirements and save it in a file named `tf-resnet50-horovod-job.yaml`\. 

   ```
   apiVersion: kubeflow.org/v1alpha2
   kind: MPIJob
   metadata:
     name: <JOB_NAME>
   spec:
     slotsPerWorker: 1
     cleanPodPolicy: Running
     mpiReplicaSpecs:
       Launcher:
         replicas: 1
         template:
            spec:
              containers:
              - image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-tensorflow-training:1.15.5-gpu-py37-cu100-ubuntu18.04-example
                name: <JOB_NAME>
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
                 - /deep-learning-models/models/resnet/tensorflow/train_imagenet_resnet_hvd.py
                args: 
                 - --num_epochs
                 - "10"
                 - --synthetic
       Worker:
         replicas: <NUM_WORKERS>
         template:
           spec:
             containers:
             - image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-tensorflow-training:1.15.5-gpu-py37-cu100-ubuntu18.04-example
               name: tensorflow-worker
               resources:
                 limits:
                   nvidia.com/gpu: <GPUS>
   ```

1. Run the distributed training job with the pod file you just created\.

   ```
   $ kubectl create -f tf-resnet50-horovod-job.yaml
   ```

1. Check the status\. The name of the job appears in the status\. If you're running any other tests or have previously run other tests, they appear in this list\. Run this several times until you see the status change to “Running”\.

   ```
   $ kubectl get pods -o wide
   ```

   You should see something similar to the following output:

   ```
   NAME                                        READY   STATUS   RESTARTS AGE
   
   tf-resnet50-horovod-job-1794-launcher-9wbsg  1/1    Running    0       31s
   tf-resnet50-horovod-job-1794-worker-0        1/1    Running    0       31s
   tf-resnet50-horovod-job-1794-worker-1        1/1    Running    0       31s
   tf-resnet50-horovod-job-1794-worker-2        1/1    Running    0       31s
   ```

1. Based on the name of the launcher pod above, check the logs to see the training output\.

   ```
   $ kubectl logs -f --tail 10 <LAUNCHER_POD_NAME>
   ```

1. You can check the logs to watch the training progress\. You can also continue to check “get pods” to refresh the status\. When the status changes to “Completed” you will know that the training job is done\.

1. To clean up and rerun a job:

   ```
   $ kubectl delete -f tf-resnet50-horovod-job.yaml
   ```

### Next steps<a name="deep-learning-containers-eks-distributed-training-gpu-tf-next"></a>

To learn GPU\-based inference on Amazon EKS using TensorFlow with Deep Learning Containers, see [TensorFlow GPU inference](deep-learning-containers-eks-tutorials-gpu-inference.md#deep-learning-containers-eks-tutorials-gpu-inference-tf)\. 

## PyTorch distributed GPU training<a name="deep-learning-containers-eks-tutorials-distributed-gpu-training-pytorch"></a>

This tutorial will guide you on distributed training with PyTorch on your multi\-node GPU cluster\. It uses Gloo as the backend\.

1. Verify that the PyTorch custom resource is installed\.

   ```
   $ kubectl get crd
   ```

   The output should include `pytorchjobs.kubeflow.org`\.

1. Ensure that the NVIDIA plugin `daemonset` is running\.

   ```
   $ kubectl get daemonset -n kubeflow
   ```

   The output should should look similar to the following\.

   ```
   NAME                          DESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE SELECTOR AGE
   nvidia-device-plugin-daemonset   3         3      3        3        3    <none>  35h
   ```

1.  Use the following text to create a gloo\-based distributed data parallel job\. Save it in a file named `distributed.yaml`\.

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
           spec:
             containers:
             - name: "pytorch"
               image: "763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-pytorch-training:1.7.1-gpu-py36-cu110-ubuntu18.04-example"
               args:
                 - "--backend"
                 - "gloo"
                 - "--epochs"
                 - "5"
       Worker:
         replicas: 2
         restartPolicy: OnFailure
         template:
           spec:
             containers:
             - name: "pytorch"
               image: "763104351884.dkr.ecr.us-east-1.amazonaws.com/aws-samples-pytorch-training:1.7.1-gpu-py36-cu110-ubuntu18.04-example"
               args:
                 - "--backend"
                 - "gloo"
                 - "--epochs"
                 - "5"
               resources:
                 limits:
                   nvidia.com/gpu: 1
   ```

1. Run a distributed training job with the pod file you just created\.

   ```
   $ kubectl create -f distributed.yaml
   ```

1. You can check the status of the job using the following:

   ```
   $ kubectl logs kubeflow-pytorch-gpu-dist-job
   ```

   To view logs continuously, use:

   ```
   $ kubectl logs -f <pod>
   ```

See [Cleanup](deep-learning-containers-eks-kubeflow-setup.md#deep-learning-containers-eks-kubeflow-cleanup) for information on cleaning up a cluster after you are done using it\. 

## Amazon S3 Plugin for PyTorch<a name="deep-learning-containers-eks-tutorials-distributed-gpu-pytorch-s3-plugin"></a>

Deep Learning Containers include a plugin that enables you to use data from an Amazon S3 bucket for PyTorch training\. See the Amazon EKS [Amazon S3 Plugin for PyTorch GPU guide](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/deep-learning-containers-eks-tutorials-gpu-training.html#deep-learning-containers-eks-training-gpu-pytorch-s3-plugin) to get started\.

For more information and additional examples, see the [Amazon S3 Plugin for PyTorch](https://github.com/aws/amazon-s3-plugin-for-pytorch) repository\.

See [Cleanup](deep-learning-containers-eks-kubeflow-setup.md#deep-learning-containers-eks-kubeflow-cleanup) for information on cleaning up a cluster after you are done using it\. 

### Next steps<a name="deep-learning-containers-eks-distributed-training-gpu-pytorch-next"></a>

To learn GPU\-based inference on Amazon EKS using PyTorch with Deep Learning Containers, see [PyTorch GPU inference](deep-learning-containers-eks-tutorials-gpu-inference.md#deep-learning-containers-eks-tutorials-gpu-inference-pytorch)\. 