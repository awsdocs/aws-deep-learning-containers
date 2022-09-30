# Troubleshooting AWS Deep Learning Containers on EKS<a name="deep-learning-containers-eks-troubleshooting"></a>

 The following are common errors that might be returned in the command line when using AWS Deep Learning Containers on an Amazon EKS cluster\. Each error is followed by a solution to the error\.

## Troubleshooting<a name="deep-learning-containers-eks-troubleshooting-examples"></a>

**Topics**
+ [Setup Errors](#deep-learning-containers-eks-troubleshooting-setup)
+ [Usage Errors](#deep-learning-containers-eks-troubleshooting-usage)
+ [Cleanup Errors](#deep-learning-containers-eks-troubleshooting-cleanup)

### Setup Errors<a name="deep-learning-containers-eks-troubleshooting-setup"></a>

The following errors might be returned when setting up Deep Learning Containers on your Amazon EKS cluster\.
+ ** Error: registry `kubeflow` does not exist**

  ```
  $ ks pkg install kubeflow/tf-serving
  	 ERROR registry 'kubeflow' does not exist
  ```

   To solve this error, run the following command\. 

  ```
  ks registry add kubeflow github.com/google/kubefl ow/tree/master/kubeflow
  ```
+ ** Error: context deadline exceeded**

  ```
  $ eksctl create cluster <args>
  	 [✖] waiting for CloudFormation stack "eksctl-training-cluster-1-nodegroup-ng-8c4c94bc" to reach "CREATE_COMPLETE" status: RequestCanceled: waiter context canceled
  	 caused by: context deadline exceeded
  ```

   To solve this error, verify that you have not exceeded capacity for your account\. You can also try to create your cluster in a different region\. 
+ ** Error: The connection to the server localhost:8080 was refused **

  ```
  $ kubectl get nodes
  	 The connection to the server localhost:8080 was refused - did you specify the right host or port?
  ```

   To solve this error, copy the cluster to the Kubernetes configuration by running the following\. 

  ```
  cp ~/.kube/eksctl/clusters/<cluster-name> ~/.kube/config
  ```
+ ** Error: handle object: patching object from cluster: merging object with existing state: Unauthorized**

  ```
  $ ks apply default
  	 ERROR handle object: patching object from cluster: merging object with existing state: Unauthorized
  ```

  This error is due to a concurrency issue that can occur when multiple users with different authorization or credentials credentials try to start jobs on the same cluster\. Verify that you are starting a job on the correct cluster\.
+ ** Error: Could not create app; directory '/home/ubuntu/kubeflow\-tf\-hvd' already exists**

  ```
  $ APP_NAME=kubeflow-tf-hvd; ks init ${APP_NAME}; cd ${APP_NAME}
  	 INFO Using context "arn:aws:eks:eu-west-1:999999999999:cluster/training-gpu-1" from kubeconfig file "/home/ubuntu/.kube/config"
  	 ERROR Could not create app; directory '/home/ubuntu/kubeflow-tf-hvd' already exists
  ```

   You can safely ignore this warning\. However, you may have additional cleanup to do inside that folder\. To simplify cleanup, delete the folder\. 

### Usage Errors<a name="deep-learning-containers-eks-troubleshooting-usage"></a>

```
ssh: Could not resolve hostname openmpi-worker-1.openmpi.kubeflow-dist-train-tf: Name or service not known
```

 If you see this error message while using the Amazon EKS cluster, run the NVIDIA device plugin installation step again\. Verify that you have targeted the right cluster by either passing in the specific config file or switching your active cluster to the targeted cluster\. 

### Cleanup Errors<a name="deep-learning-containers-eks-troubleshooting-cleanup"></a>

The following errors might be returned when cleaning up the resources of your Amazon EKS cluster\.
+ ** Error: the server doesn't have a resource type "*namspace*"**

  ```
  $ kubectl delete namespace ${NAMESPACE}
  	 error: the server doesn't have a resource type "namspace"
  ```

   Verify the spelling of your namespace is correct\. 
+ ** Error: the server has asked for the client to provide credentials**

  ```
  $ ks delete default
  	 ERROR the server has asked for the client to provide credentials
  ```

  To solve this error, verify that `~/.kube/config` points to the correct cluster and that AWS credentials have been correctly configured using `aws configure` or by exporting AWS environment variables\.
+ ** Error: finding app root from starting path: : unable to find ksonnet project**

  ```
  $ ks delete default
  	 ERROR finding app root from starting path: : unable to find ksonnet project
  ```

  To solve this error, verify that you are in the directory created by the ksonnet app\. This is the folder where `ks init` was run\.
+ ** Error: Error from server \(NotFound\): pods "openmpi\-master" not found**

  ```
  $ kubectl logs -n ${NAMESPACE} -f ${COMPONENT}-master > results/benchmark_1.out
  	 Error from server (NotFound): pods "openmpi-master" not found
  ```

  This error might be caused by trying to access resources after the context is deleted\. Deleting the default context causes the corresponding resources to be deleted as well\. 