# GPU Training<a name="deep-learning-containers-eks-tutorials-gpu-training"></a>

This section is for training on GPU\-based clusters\.

For a complete list of Deep Learning Containers, refer to [Deep Learning Containers Images](deep-learning-containers-images.md)\. For tips about the best configuration settings if you're using the Intel Math Kernel Library \(MKL\), see [AWS Deep Learning Containers Intel Math Kernel Library \(MKL\) Recommendations](deep-learning-containers-mkl.md)\. 

**Topics**
+ [Apache MXNet \(Incubating\) GPU training](#deep-learning-containers-eks-tutorials-gpu-training-mxnet)
+ [TensorFlow GPU training](#deep-learning-containers-eks-tutorials-gpu-training-tf)
+ [PyTorch GPU training](#deep-learning-containers-eks-tutorials-gpu-training-pytorch)
+ [Amazon S3 Plugin for PyTorch](#deep-learning-containers-eks-training-gpu-pytorch-s3-plugin)

## Apache MXNet \(Incubating\) GPU training<a name="deep-learning-containers-eks-tutorials-gpu-training-mxnet"></a>

This tutorial guides you on training with Apache MXNet \(Incubating\) on your single node GPU cluster\.

1. Create a pod file for your cluster\. A pod file will provide the instructions about what the cluster should run\. This pod file will download the MXNet repository and run an MNIST example\. Open vi or vim and copy and past the following content\. Save this file as `mxnet.yaml`\.

   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: mxnet-training
   spec:
     restartPolicy: OnFailure
     containers:
     - name: mxnet-training
       image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/mxnet-training:1.6.0-gpu-py36-cu101-ubuntu16.04
       command: ["/bin/sh","-c"]
       args: ["git clone -b v1.4.x https://github.com/apache/incubator-mxnet.git && python ./incubator-mxnet/example/image-classification/train_mnist.py"]
   ```

1. Assign the pod file to the cluster using kubectl\.

   ```
   $ kubectl create -f mxnet.yaml
   ```

1. You should see the following output:

   ```
   pod/mxnet-training created
   ```

1. Check the status\. The name of the job “tensorflow\-training” was in the tf\.yaml file\. It will now appear in the status\. If you're running any other tests or have previously run something, it will appear in this list\. Run this several times until you see the status change to “Running”\.

   ```
   $ kubectl get pods
   ```

   You should see the following output:

   ```
   NAME READY STATUS RESTARTS AGE
   mxnet-training 0/1 Running 8 19m
   ```

1. Check the logs to see the training output\.

   ```
   $ kubectl logs mxnet-training
   ```

   You should see something similar to the following output:

   ```
   Cloning into 'incubator-mxnet'...
   INFO:root:Epoch[0] Batch [0-100]    Speed: 18437.78 samples/sec    accuracy=0.777228
   INFO:root:Epoch[0] Batch [100-200]    Speed: 16814.68 samples/sec    accuracy=0.907188
   INFO:root:Epoch[0] Batch [200-300]    Speed: 18855.48 samples/sec    accuracy=0.926719
   INFO:root:Epoch[0] Batch [300-400]    Speed: 20260.84 samples/sec    accuracy=0.938438
   INFO:root:Epoch[0] Batch [400-500]    Speed: 9062.62 samples/sec    accuracy=0.938594
   INFO:root:Epoch[0] Batch [500-600]    Speed: 10467.17 samples/sec    accuracy=0.945000
   INFO:root:Epoch[0] Batch [600-700]    Speed: 11082.03 samples/sec    accuracy=0.954219
   INFO:root:Epoch[0] Batch [700-800]    Speed: 11505.02 samples/sec    accuracy=0.956875
   INFO:root:Epoch[0] Batch [800-900]    Speed: 9072.26 samples/sec    accuracy=0.955781
   INFO:root:Epoch[0] Train-accuracy=0.923424
   ...
   ```

1. Check the logs to watch the training progress\. You can also continue to check “get pods” to refresh the status\. When the status changes to “Completed”, the training job is done\.

### Next steps<a name="deep-learning-containers-eks-training-gpu-mxnet-next"></a>

To learn GPU\-based inference on Amazon EKS using MXNet with Deep Learning Containers, see [Apache MXNet \(Incubating\) GPU inference](deep-learning-containers-eks-tutorials-gpu-inference.md#deep-learning-containers-eks-tutorials-gpu-inference-mxnet)\. 

## TensorFlow GPU training<a name="deep-learning-containers-eks-tutorials-gpu-training-tf"></a>

This tutorial guides you on training TensorFlow models on your single node GPU cluster\. 

1. Create a pod file for your cluster\. A pod file will provide the instructions about what the cluster should run\. This pod file will download Keras and run a Keras example\. This example uses the TensorFlow framework\. Open vi or vim and copy and past the following content\. Save this file as `tf.yaml`\. You can use this with either TensorFlow or TensorFlow 2\. To use it with TensorFlow 2, change the Docker image to a TensorFlow 2 image\.

   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: tensorflow-training
   spec:
     restartPolicy: OnFailure
     containers:
     - name: tensorflow-training
       image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-training:1.15.2-gpu-py37-cu100-ubuntu18.04
       command: ["/bin/sh","-c"]
       args: ["git clone https://github.com/fchollet/keras.git && python /keras/examples/mnist_cnn.py"]
       resources:
         limits:
           nvidia.com/gpu: 1
   ```

1. Assign the pod file to the cluster using kubectl\.

   ```
   $ kubectl create -f tf.yaml
   ```

1. You should see the following output:

   ```
   pod/tensorflow-training created
   ```

1. Check the status\. The name of the job “tensorflow\-training” was in the tf\.yaml file\. It will now appear in the status\. If you're running any other tests or have previously run something, it appears in this list\. Run this several times until you see the status change to “Running”\.

   ```
   $ kubectl get pods
   ```

   You should see the following output:

   ```
   NAME READY STATUS RESTARTS AGE
   tensorflow-training 0/1 Running 8 19m
   ```

1. Check the logs to see the training output\.

   ```
   $ kubectl logs tensorflow-training
   ```

   You should see something similar to the following output:

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

1. Check the logs to watch the training progress\. You can also continue to check “get pods” to refresh the status\. When the status changes to “Completed”, the training job is done\.

### Next steps<a name="deep-learning-containers-eks-training-gpu-tf-next"></a>

To learn GPU\-based inference on Amazon EKS using TensorFlow with Deep Learning Containers, see [TensorFlow GPU inference](deep-learning-containers-eks-tutorials-gpu-inference.md#deep-learning-containers-eks-tutorials-gpu-inference-tf)\. 

## PyTorch GPU training<a name="deep-learning-containers-eks-tutorials-gpu-training-pytorch"></a>

This tutorial guides you on training with PyTorch on your single node GPU cluster\.

1. Create a pod file for your cluster\. A pod file will provide the instructions about what the cluster should run\. This pod file will download the PyTorch repository and run an MNIST example\. Open vi or vim, then copy and paste the following content\. Save this file as `pytorch.yaml`\.

   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: pytorch-training
   spec:
     restartPolicy: OnFailure
     containers:
     - name: pytorch-training
       image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.5.1-gpu-py36-cu101-ubuntu16.04
       command:
         - "/bin/sh"
         - "-c"
       args:
       - "git clone https://github.com/pytorch/examples.git && python examples/mnist/main.py --no-cuda"
       env:
       - name: OMP_NUM_THREADS
         value: "36"
       - name: KMP_AFFINITY
         value: "granularity=fine,verbose,compact,1,0"
       - name: KMP_BLOCKTIME
         value: "1"
   ```

1. Assign the pod file to the cluster using kubectl\.

   ```
   $ kubectl create -f pytorch.yaml
   ```

1. You should see the following output:

   ```
   pod/pytorch-training created
   ```

1. Check the status\. The name of the job "pytorch\-training” was in the pytorch\.yaml file\. It will now appear in the status\. If you're running any other tests or have previously run something, it appears in this list\. Run this several times until you see the status change to “Running”\.

   ```
   $ kubectl get pods
   ```

   You should see the following output:

   ```
   NAME READY STATUS RESTARTS AGE
   pytorch-training 0/1 Running 8 19m
   ```

1. Check the logs to see the training output\.

   ```
   $ kubectl logs pytorch-training
   ```

   You should see something similar to the following output:

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

1. Check the logs to watch the training progress\. You can also continue to check “get pods” to refresh the status\. When the status changes to “Completed", the training job is done\.

See [Cleanup](deep-learning-containers-eks-kubeflow-setup.md#deep-learning-containers-eks-kubeflow-cleanup) for information on cleaning up a cluster after you are done using it\. 

### Next steps<a name="deep-learning-containers-eks-training-gpu-pytorch-next"></a>

To learn GPU\-based inference on Amazon EKS using PyTorch with Deep Learning Containers, see [PyTorch GPU inference](deep-learning-containers-eks-tutorials-gpu-inference.md#deep-learning-containers-eks-tutorials-gpu-inference-pytorch)\. 

## Amazon S3 Plugin for PyTorch<a name="deep-learning-containers-eks-training-gpu-pytorch-s3-plugin"></a>

Deep Learning Containers include a plugin that enables you to use data from an Amazon S3 bucket for PyTorch training\.

1. To begin using the Amazon S3 plugin on Amazon EKS, check to make sure that your cluster instances have full access to Amazon S3\. [Create an IAM role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) that grants Amazon S3 access to an Amazon EC2 instance and attach the role to your instance\. You can use the [AmazonS3FullAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AmazonS3FullAccess$serviceLevelSummary) or [AmazonS3ReadOnlyAccess](https://console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess$serviceLevelSummary) policies\.

1.  Set up your `AWS_REGION` environment variable with the region of your choice\.

   ```
   export AWS_REGION=us-east-1
   ```

1. Create a pod file for your cluster\. A pod file will provide the instructions about what the cluster should run\. This pod file will use the PyTorch Amazon S3 plugin to access an example Amazon S3 dataset\.
**Note**  
Your GPU cluster should use `p3.8xlarge` nodes or greater for this example\.

   Open vi or vim, then copy and paste the following content\. Save this file as `s3plugin.yaml`\.

   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: pytorch-s3-plugin
   spec:
     restartPolicy: OnFailure
     containers:
     - name: pytorch-s3-plugin
       image: 763104351884.dkr.ecr.us-east-1.amazonaws.com/pytorch-training:1.8.1-gpu-py36-cu111-ubuntu18.04-v1.7
       command:
         - "/bin/sh"
         - "-c"
       args:
       - "git clone https://github.com/aws/amazon-s3-plugin-for-pytorch.git && python amazon-s3-plugin-for-pytorch/examples/s3_imagenet_example.py"
       env:
       - name: OMP_NUM_THREADS
         value: "36"
       - name: KMP_AFFINITY
         value: "granularity=fine,verbose,compact,1,0"
       - name: KMP_BLOCKTIME
         value: "1"
   ```

1. Assign the pod file to the cluster using kubectl\.

   ```
   $ kubectl create -f s3plugin.yaml
   ```

1. Check the status\. The name of the job `pytorch-s3-plugin` that was specified in the `s3plugin.yaml` file will now appear alongside the status information\. You can run the following command several times until you see the status change to “Running\.”

   ```
   $ kubectl get pods
   ```

   You should see the following output:

   ```
   NAME READY STATUS RESTARTS AGE
   pytorch-s3-plugin 0/1 Running 8 19m
   ```

1. Check the logs to see more details\.

   ```
   $ kubectl logs pytorch-s3-plugin
   ```

For more information, see the [Amazon S3 Plugin for PyTorch](https://github.com/aws/amazon-s3-plugin-for-pytorch) repository\. 