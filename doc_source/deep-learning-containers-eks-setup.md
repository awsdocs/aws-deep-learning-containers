# Amazon EKS Setup<a name="deep-learning-containers-eks-setup"></a>

This section provides installation instructions to setup a deep learning environment running AWS Deep Learning Containers on Amazon Elastic Kubernetes Service \(Amazon EKS\)\. 

## Custom Images<a name="deep-learning-containers-eks-setup-custom-images"></a>

Custom images are helpful if you want to load your own code or datasets and have them available on each node in your cluster\. Examples are provided that use custom images\. You can try them out to get started without creating your own\.
+ [Building AWS Deep Learning Containers Custom Images](deep-learning-containers-custom-images.md)

## Licensing<a name="deep-learning-containers-eks-setup-licensing"></a>

To use GPU hardware, use an Amazon Machine Image that has the necessary GPU drivers\. We recommend using the Amazon EKS\-optimized AMI with GPU support, which is used in subsequent steps of this guide\. This AMI includes software that is not AWS, so it requires an end user license agreement \(EULA\)\. You must subscribe to the EKS\-optimized AMI in the AWS Marketplace and accept the EULA before you can use the AMI in your worker node groups\.

**Important**  
To subscribe to the AMI, visit the [AWS Marketplace](https://aws.amazon.com/marketplace/pp/B07GRHFXGM)\.

## Configure Security Settings<a name="deep-learning-containers-eks-setup-security-config"></a>

To use Amazon EKS you must have a user account that has access to several security permissions\. These are set with the AWS Identity and Access Management \(IAM\) tool\.

1. Create an IAM user or update an existing IAM user by following the steps in [Creating an IAM user in your AWS account](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)\.

1. Get the credentials of this user\.

   1. Open the IAM console at [https://console\.aws\.amazon\.com/iam/](https://console.aws.amazon.com/iam/)\.

   1. Under `Users`, select your user\.

   1. Select `Security Credentials`\.

   1. Select `Create access key`\.

   1. Download the key pair or copy the information for use later\.

1. Add the following policies to your IAM user\. These policies provide the required access for Amazon EKS, IAM, and Amazon Elastic Compute Cloud \(Amazon EC2\)\.

   1. Select `Permissions`\.

   1. Select `Add permissions`\.

   1. Select `Create policy`\.

   1. From the `Create policy` window, select the `JSON` tab\. 

   1. Paste the following content\. 

      ```
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Sid": "VisualEditor0",
                  "Effect": "Allow",
                  "Action": "eks:*",
                  "Resource": "*"
              }
          ]
      }
      ```

   1. Name the policy `EKSFullAccess` and create the policy\.

   1. Navigate back to the `Grant permissions` window\.

   1. Select `Attach existing policies directly`\.

   1. Search for `EKSFullAccess`, and select the check box\.

   1. Search for `AWSCloudFormationFullAccess`, and select the check box\.

   1. Search for `AmazonEC2FullAccess`, and select the check box\.

   1. Search for `IAMFullAccess`, and select the check box\.

   1. Search `AmazonEC2ContainerRegistryReadOnly`, and select the check box\.

   1. Search `AmazonEKS_CNI_Policy`, and select the check box\.

   1. Search `AmazonS3FullAccess`, and select the check box\.

   1. Accept the changes\.

## Gateway Node<a name="deep-learning-containers-eks-setup-gateway-node"></a>

To setup an Amazon EKS cluster, use the open source tool, `eksctl`\. We recommend that you use an Amazon EC2 instance with the Deep Learning Base AMI \(Ubuntu\) to allocate and control your cluster\. You can run these tools locally on your computer or an Amazon EC2 instance that you already have running\. However, to simplify this guide we assume you're using a Deep Learning Base AMI \(DLAMI\) with Ubuntu 16\.04\. We refer to this as your gateway node\. 

Before you start, consider the location of your training data or where you want to run your cluster for responding to inference requests\. Typically your data and cluster for training or inference should be in the same Region\. Also, you spin up your gateway node in this same Region\. You can follow this quick [10 minute tutorial](https://aws.amazon.com/getting-started/tutorials/get-started-dlami/) that guides you to launch a DLAMI to use as your gateway node\.

1. Login to your gateway node\.

1. Install or upgrade AWS CLI\. To access the required new Kubernetes features, you must have the latest version\.

   ```
   $ sudo pip install --upgrade awscli
   ```

1. Install `eksctl` by running the command corresponding to your operating system in [https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)'s installation instructions\. For more information about `eksctl`, see also [eksctl documentation](https://eksctl.io/)\.

1. Install `kubectl` by following the steps in the [Installing kubectl](https://docs.aws.amazon.com//eks/latest/userguide/install-kubectl.html) guide\. 
**Note**  
 You must use a `kubectl` version that is within one minor version difference of your Amazon EKS cluster control plane version\. For example, a 1\.18 `kubectl` client works with Kubernetes 1\.17, 1\.18 and 1\.19 clusters\. 

1. Install `aws-iam-authenticator` by running the following commands\. For more information on aws\-iam\-authenticator, see [Installing `aws-iam-authenticator`](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)\.

   ```
   $ curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/aws-iam-authenticator
   $ chmod +x aws-iam-authenticator
   $ cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$HOME/bin:$PATH
   ```

1. Run `aws configure` for the IAM user from the Security Configuration section\. You are copying the IAM user's AWS Access Key, then the AWS Secret Access Key that you accessed in the IAM console and pasting these into the prompts from `aws configure`\. 

## GPU Clusters<a name="deep-learning-containers-eks-setup-gpu-clusters"></a>

1. Examine the following command to create a cluster using a p3\.8xlarge instance type\. You must make the following modifications before you run it\.
   + `name` is what you use to manage your cluster\. You can change `cluster-name` to be whatever name you like as long as there are no spaces or special characters\.
   + `eks-version` is the Amazon EKS kubernetes version\. For the supported Amazon EKS versions, see [Available Amazon EKS Kubernetes versions](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html)\.
   + `nodes` is the number of instances you want in your cluster\. In this example, we're starting with three nodes\.
   + `node-type` refers to an [instance class](https://aws.amazon.com/ec2/instance-types/)\.
   + `timeout` and `*ssh-access *` can be left alone\.
   + `ssh-public-key` is the name of the key that you want to use to login your worker nodes\. Either use a security key you already use or create a new one but be sure to swap out the ssh\-public\-key with a key that was allocated for the Region you used\. Note: You only need to provide the key name as seen in the 'key pairs' section of the Amazon EC2 Console\. 
   + `region` is the Amazon EC2 Region where the cluster is launched\. If you plan to use training data that resides in a specific Region \(other than *<us\-east\-1>*\) we recommend that you use the same Region\. The ssh\-public\-key must have access to launch instances in this Region\.
**Note**  
The rest of this guide assumes *<us\-east\-1>* as the Region\.

1. After you have made changes to the command, run it, and wait\. It can take several minutes for a single node cluster, and can take even longer if you chose to create a large cluster\.

   ```
   $ eksctl create cluster <cluster-name> \
                         --version <eks-version> \
                         --nodes 3 \
                         --node-type=<p3.8xlarge> \
                         --timeout=40m \
                         --ssh-access \
                         --ssh-public-key <key_pair_name> \
                         --region <us-east-1> \
                         --zones=us-east-1a,us-east-1b,us-east-1d \
                         --auto-kubeconfig
   ```

   You should see something similar to the following output:

   ```
   EKS cluster "training-1" in "us-east-1" region is ready
   ```

1. Ideally the auto\-kubeconfig should have configured your cluster\. However, if you run into issues you can run the command below to set your kubeconfig\. This command can also be used if you want to change your gateway node and manage your cluster from elsewhere\.

   ```
   $ aws eks --region <region> update-kubeconfig --name <cluster-name>
   ```

   You should see something similar to the following output:

   ```
   Added new context arn:aws:eks:us-east-1:999999999999:cluster/training-1 to /home/ubuntu/.kube/config
   ```

1. If you plan to use GPU instance types, make sure to run the [NVIDIA device plugin for Kubernetes](https://github.com/NVIDIA/k8s-device-plugin) on your cluster with the following command:

   ```
   $ kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.12/nvidia-device-plugin.yml
   $ kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.9.0/nvidia-device-plugin.yml
   ```

1. Verify the GPUs available on each node in your cluster

   ```
   $ kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"
   ```

## CPU Clusters<a name="deep-learning-containers-eks-setup-cpu-clusters"></a>

Refer to the previous section's discussion on using the eksctl command to launch a GPU cluster, and modify `node-type` to use a CPU instance type\.

## Habana Clusters<a name="deep-learning-containers-eks-setup-habana-clusters"></a>

Refer to the previous discussion on using the eksctl command to launch a GPU cluster, and modify `node-type` to use an instance with Habana Gaudi accelerators, such as the [DL1 instance](http://aws.amazon.com/https://aws.amazon.com/ec2/instance-types/dl1/) type\.

## Test Your Clusters<a name="deep-learning-containers-eks-setup-test"></a>

1. You can run a kubectl command on the cluster to check its status\. Try the command to make sure it is picking up the current cluster you want to manage\. 

   ```
   $ kubectl get nodes -o wide
   ```

1. Take a look in \~/\.kube\. This directory has the kubeconfig files for the various clusters configured from your gateway node\. If you browse further into the folder you can find \~/\.kube/eksctl/clusters \- This holds the kubeconfig file for clusters created using eksctl\. This file has some details which you ideally shouldn't have to modify, since the tools are generating and updating the configurations for you, but it is good to reference when troubleshooting\. 

1. Verify that the cluster is active\.

   ```
   $ aws eks --region <region> describe-cluster --name <cluster-name> --query cluster.status
   ```

   You should see the following output:

   ```
   "ACTIVE"
   ```

1. Verify the kubectl context if you have multiple clusters set up from the same host instance\. Sometimes it helps to make sure that the default context found by kubectl is set properly\. Check this using the following command: 

   ```
   $ kubectl config get-contexts
   ```

1. If the context is not set as expected, fix this using the following command: 

   ```
   $ aws eks --region <region> update-kubeconfig --name <cluster-name>
   ```

## Manage Your Clusters<a name="deep-learning-containers-eks-setup-manage"></a>

When you want to control or query a cluster you can address it by the configuration file using the kubeconfig parameter\. This is useful when you have more than one cluster\. For example, if you have a separate cluster called “training\-gpu\-1” you can call the get pods command on it by passing the configuration file as a parameter as follows:

```
$ kubectl --kubeconfig=/home/ubuntu/.kube/eksctl/clusters/training-gpu-1 get pods
```

It is useful to note that you can run this same command without the kubeconfig parameter\. In that case, the command will use the current actively controlled cluster \(`current-context`\)\.

```
$ kubectl get pods
```

If you setup multiple clusters and they have yet to have the NVIDIA plugin installed, you can install it this way:

```
$ kubectl --kubeconfig=/home/ubuntu/.kube/eksctl/clusters/training-gpu-1 create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.9.0/nvidia-device-plugin.yml
```

You also change the active cluster by updating the kubeconfig, passing the name of the cluster you want to manage\. The following command updates kubeconfig and removes the need to use the kubeconfig parameter\.

```
$ aws eks —region us-east-1 update-kubeconfig —name training-gpu-1
```

If you follow all of the examples in this guide, you might switch frequently between active clusters\. This is so you can orchestrate training or inference or use different frameworks running on different clusters\.

## Cleanup<a name="deep-learning-containers-eks-setup-cleanup"></a>

When you're done using the cluster, delete it to avoid incurring additional costs\.

```
$ eksctl delete cluster --name=<cluster-name>
```

To delete only a pod, run the following:

```
$ kubectl delete pods <name>
```

To reset the secret for access to the cluster, run the following:

```
$ kubectl delete secret ${SECRET} -n ${NAMESPACE} || true
```

To delete a `nodegroup` attached to a cluster, run the following:

```
$ eksctl delete nodegroup --name <cluster_name>
```

To attach a `nodegroup` to a cluster, run the following:

```
$ eksctl create nodegroup
                      --cluster <cluster-name> \
                      --node-ami <ami_id> \
                      --nodes <num_nodes> \
                      --node-type=<instance_type> \
                      --timeout=40m \
                      --ssh-access \
                      --ssh-public-key <key_pair_name> \
                      --region <us-east-1> \
                      --auto-kubeconfig
```

## Next steps<a name="deep-learning-containers-eks-setup-next"></a>

To learn about training and inference with Deep Learning Containers on Amazon EKS, visit [Training](deep-learning-containers-eks-tutorials-training.md) or [Inference](deep-learning-containers-eks-tutorials-inference.md)\.

**Topics**
+ [Custom Images](#deep-learning-containers-eks-setup-custom-images)
+ [Licensing](#deep-learning-containers-eks-setup-licensing)
+ [Configure Security Settings](#deep-learning-containers-eks-setup-security-config)
+ [Gateway Node](#deep-learning-containers-eks-setup-gateway-node)
+ [GPU Clusters](#deep-learning-containers-eks-setup-gpu-clusters)
+ [CPU Clusters](#deep-learning-containers-eks-setup-cpu-clusters)
+ [Habana Clusters](#deep-learning-containers-eks-setup-habana-clusters)
+ [Test Your Clusters](#deep-learning-containers-eks-setup-test)
+ [Manage Your Clusters](#deep-learning-containers-eks-setup-manage)
+ [Cleanup](#deep-learning-containers-eks-setup-cleanup)
+ [Next steps](#deep-learning-containers-eks-setup-next)
+ [Training](deep-learning-containers-eks-tutorials-training.md)
+ [Inference](deep-learning-containers-eks-tutorials-inference.md)