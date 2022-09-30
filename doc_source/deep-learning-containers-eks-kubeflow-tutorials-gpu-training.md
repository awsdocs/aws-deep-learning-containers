# GPU Training<a name="deep-learning-containers-eks-kubeflow-tutorials-gpu-training"></a>

This section demonstrates how to train a model on GPU instances using [Kubeflow training operator](https://www.kubeflow.org/docs/components/training/) and Deep Learning Containers\. 

Make sure that your cluster has GPU nodes before you run the examples\. If you do not have GPU nodes in your cluster, use the following command to add a nodegroup to your cluster\. Be sure to select an [Amazon EC2 instance](https://aws.amazon.com/ec2/instance-types/) \(`node-type`\) in the Accelerated Computing category\. 

```
$ eksctl create nodegroup --cluster $CLUSTER_NAME --region $CLUSTER_REGION \
      --nodes 2 --nodes-min 1 --nodes-max 3 --node-type p3.2xlarge
```

For a complete list of Deep Learning Containers, see [Deep Learning Containers Images](deep-learning-containers-images.md)\. For tips about configuration settings when using the Intel Math Kernel Library \(MKL\), see [AWS Deep Learning Containers Intel Math Kernel Library \(MKL\) Recommendations](deep-learning-containers-mkl.md)\. 

**Topics**
+ [PyTorch GPU training](#deep-learning-containers-eks-tutorials-gpu-training-pytorch)
+ [TensorFlow GPU training](#deep-learning-containers-eks-kubeflow-tutorials-gpu-training-tf)

## PyTorch GPU training<a name="deep-learning-containers-eks-tutorials-gpu-training-pytorch"></a>

Your deployment of Kubeflow on AWS comes with [PyTorchJob]( https://www.kubeflow.org/docs/components/training/pytorch/)\. This is the Kubeflow implementation of [Kubernetes custom resource](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) that is used to run distributed PyTorch training jobs on Amazon EKS\. 

 This tutorial shows how to train a model with PyTorch in a single node GPU instance\. You will run this [PyTorch MNIST example](https://github.com/pytorch/examples/blob/main/mnist/main.py) in your container from Deep Learning Containers, that is managed by Kubeflow on AWS\. 

1.  Create a PyTorchJob\.

   1. Create the job configuration file\.

      Open `vi` or `vim`, then copy and paste the following content\. Save this file as `pytorch.yaml`\. 

      ```
      apiVersion: "kubeflow.org/v1"
      kind: PyTorchJob
      metadata:
        name: pytorch-training
      spec:
        pytorchReplicaSpecs:
          Worker:
            restartPolicy: OnFailure
            template:
              metadata:
                annotations:
                  sidecar.istio.io/inject: "false"
              spec:
                containers:
                  - name: pytorch
                    imagePullPolicy: Always
                    image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.12.0-gpu-py38-cu116-ubuntu20.04-ec2
                    command:
                      - "/bin/sh"
                      - "-c"
                    args:
                    - "git clone https://github.com/pytorch/examples.git && python examples/mnist/main.py --no-cuda --epochs=1"
                    env:
                    - name: OMP_NUM_THREADS
                      value: "36"
                    - name: KMP_AFFINITY
                      value: "granularity=fine,verbose,compact,1,0"
                    - name: KMP_BLOCKTIME
                      value: "1"
                    resources:
                      limits:
                        nvidia.com/gpu: 1
      ```

   1. Deploy the PyTorchJob configuration file using `kubectl` to start training\.

      ```
      $  kubectl create -f pytorch.yaml -n ${NAMESPACE}
      ```

      The job creates a pod running the container from Deep Learning Containers that is referenced in the field `spec.containers.image`\. This is located in the YAML file above under the container name `pytorch`\.

   1. You should see the following output\.

      ```
      pod/pytorch-training created
      ```

   1. Check the status\.

      The name of the job `pytorch-training` appears in the status\. It might take some time for the job to reach a `Running` state\. Run the following watch command to monitor the state of the job\.

      ```
      $ kubectl get pods n ${NAMESPACE} -w
      ```

      You should see the following output\.

      ```
      NAME READY STATUS RESTARTS AGE
      pytorch-training 0/1 Running 8 19m
      ```

1. Monitor your PyTorchJob\.

   1. Check the logs to watch the training progress\.

      ```
      $ kubectl logs pytorch-training-worker-0 -n ${NAMESPACE}
      ```

      You should see something similar to the following output\.

      ```
      Cloning into 'examples'...
      Downloading http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz to ../data/MNIST/raw/train-images-idx3-ubyte.gz
      9920512it [00:00, 40133996.38it/s]                           
      Extracting ../data/MNIST/raw/train-images-idx3-ubyte.gz to ../data/MNIST/raw
      Downloading http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz to ../data/MNIST/raw/train-labels-idx1-ubyte.gz
      Extracting ../data/MNIST/raw/train-labels-idx1-ubyte.gz to ../data/MNIST/raw
      32768it [00:00, 831315.84it/s]
      Downloading http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz to ../data/MNIST/raw/t10k-images-idx3-ubyte.gz
      1654784it [00:00, 13019129.43it/s]                           
      Extracting ../data/MNIST/raw/t10k-images-idx3-ubyte.gz to ../data/MNIST/raw
      Downloading http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz to ../data/MNIST/raw/t10k-labels-idx1-ubyte.gz
      8192it [00:00, 337197.38it/s]
      Extracting ../data/MNIST/raw/t10k-labels-idx1-ubyte.gz to ../data/MNIST/raw
      Processing...
      Done!
      Train Epoch: 1 [0/60000 (0%)]    Loss: 2.300039
      Train Epoch: 1 [640/60000 (1%)]    Loss: 2.213470
      Train Epoch: 1 [1280/60000 (2%)]    Loss: 2.170460
      Train Epoch: 1 [1920/60000 (3%)]    Loss: 2.076699
      Train Epoch: 1 [2560/60000 (4%)]    Loss: 1.868078
      Train Epoch: 1 [3200/60000 (5%)]    Loss: 1.414199
      Train Epoch: 1 [3840/60000 (6%)]    Loss: 1.000870
      ```

   1. Monitor the job state\.

       Run the following command to refresh the job state\. When the status changes to `Succeeded`, the training job is done\. 

      ```
      $ watch -n 5 kubectl get pytorchjobs pytorch-training -n ${NAMESPACE}
      ```

See [Cleanup](deep-learning-containers-eks-kubeflow-setup.md#deep-learning-containers-eks-kubeflow-cleanup) for information on cleaning up a cluster after you are done using it\. 

## TensorFlow GPU training<a name="deep-learning-containers-eks-kubeflow-tutorials-gpu-training-tf"></a>

Your deployment of Kubeflow on AWS comes with [TFJob](https://www.kubeflow.org/docs/components/training/tftraining/)\. This is the Kubeflow implementation of [Kubernetes custom resource](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) that is used to run distributed TensorFlow training jobs on Kubernetes\. 

 This tutorial guides you through training a classification model on [MNIST with Keras](https://github.com/keras-team/keras-io/blob/master/examples/vision/mnist_convnet.py) in a single node GPU instance running a container from Deep Learning Containers managed by Kubeflow on AWS\.

1.  Create a TFJob\.

   1. Create the job configuration file\.

      Open `vi` or `vim`, then copy and paste the following content\. Save this file as `tf.yaml`\. 

      ```
      apiVersion: kubeflow.org/v1
      kind: TFJob
      metadata:
        name: tensorflow-training
      spec:
        tfReplicaSpecs:
          Worker:
            restartPolicy: OnFailure
            template:
              metadata:
                annotations:
                  sidecar.istio.io/inject: "false"
              spec:
                containers:
                  - name: tensorflow
                    image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-training:2.9.1-gpu-py39-cu112-ubuntu20.04-ec2
                    command: ["/bin/sh","-c"]
                    args: ["git clone https://github.com/keras-team/keras-io.git && python keras-io/examples/vision/mnist_convnet.py"]
                    resources:
                      limits:
                        nvidia.com/gpu: 1
      ```

   1. Deploy the TFJob configuration file using `kubectl` to start training\.

      ```
      $  kubectl create -f tf.yaml ${NAMESPACE}
      ```

      The job creates a pod running the container from Deep Learning Containers that is referenced in the field `spec.containers.image`\. This is located in the YAML file above under the container name `tensorflow`\.

   1. You should see the following output\.

      ```
      pod/tensorflow-training created
      ```

   1. Check the status\.

      The name of the job `tensorflow-training` appears in the status\. It might take some time for the job to reach a `Running `state\. Run the following watch command to monitor the state of the job\.

      ```
      $ watch -n 5 kubectl get pods ${NAMESPACE}
      ```

      You should see the following output\.

      ```
      NAME READY STATUS RESTARTS AGE
      tensorflow-training 0/1 Running 8 19m
      ```

1. Monitor your TFJob\.

   1. Check the logs to watch the training progress\.

      ```
      $ kubectl logs tensorflow-training-worker-0 ${NAMESPACE}
      ```

      You should see something similar to the following output\.

      ```
      Cloning into 'keras'...
      Using TensorFlow backend.
      Downloading data from https://s3.amazonaws.com/img-datasets/mnist.npz
      
          8192/11490434 [..............................] - ETA: 0s
       6479872/11490434 [===============>..............] - ETA: 0s
       8740864/11490434 [=====================>........] - ETA: 0s
      11493376/11490434 [==============================] - 0s 0us/step
      x_train shape: (60000, 28, 28, 1)
      60000 train samples
      10000 test samples
      Train on 60000 samples, validate on 10000 samples
      Epoch 1/12
      2019-03-19 01:52:33.863598: I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX512F
      2019-03-19 01:52:33.867616: I tensorflow/core/common_runtime/process_util.cc:69] Creating new thread pool with default inter op setting: 2. Tune using inter_op_parallelism_threads for best performance.
      
        128/60000 [..............................] - ETA: 10:43 - loss: 2.3076 - acc: 0.0625
        256/60000 [..............................] - ETA: 5:59 - loss: 2.2528 - acc: 0.1445
        384/60000 [..............................] - ETA: 4:24 - loss: 2.2183 - acc: 0.1875
        512/60000 [..............................] - ETA: 3:35 - loss: 2.1652 - acc: 0.1953
        640/60000 [..............................] - ETA: 3:05 - loss: 2.1078 - acc: 0.2422
        ...
      ```

   1. Monitor the job state\.

       Run the following command to refresh the job state\. When the status changes to `Succeeded`, the training job is done\. 

      ```
      $ watch -n 5 kubectl get tfjobs tensorflow-training ${NAMESPACE}
      ```

See [Cleanup](deep-learning-containers-eks-kubeflow-setup.md#deep-learning-containers-eks-kubeflow-cleanup) for information on cleaning up a cluster after you are done using it\. 