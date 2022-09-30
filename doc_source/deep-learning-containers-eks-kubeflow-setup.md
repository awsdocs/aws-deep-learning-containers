# Kubeflow on AWS Setup<a name="deep-learning-containers-eks-kubeflow-setup"></a>

This section provides installation instructions to set up a deep learning environment using AWS Deep Learning Containers with Kubeflow on AWS, an open source distribution of Kubeflow\. After you finish Kubeflow on AWS setup, you can continue with training tutorials in this series\. 

## Deploy Kubeflow on AWS<a name="deep-learning-containers-eks-kubeflow-deploy"></a>

To deploy Kubeflow on AWS, follow the [Vanilla deployment option](https://awslabs.github.io/kubeflow-manifests/docs/deployment/vanilla) in the Kubeflow on AWS documentation\. Make sure that you follow all the [prerequisites](https://awslabs.github.io/kubeflow-manifests/docs/deployment/prerequisites)\. The installation instructions guide you through [creating an Amazon EKS cluster](https://awslabs.github.io/kubeflow-manifests/docs/deployment/create-eks-cluster/) before deploying Kubeflow on AWS\. 

 If you deployed a GPU cluster following the previous instructions, the NVIDIA device plug\-in for Kubernetes is already installed\. You do not need any additional setup\.

**Note**  
The following tutorials use the Vanilla version of Kubeflow on AWS as an example\. However, you can run all training and inference tutorials in this Kubeflow on AWS section with any other deployment option of Kubeflow on AWS\.  
For information about setting up and configuring Amazon RDS, Amazon S3, and Amazon Cognito resources as part of your Kubeflow on AWS deployment, see [Deployment options](https://awslabs.github.io/kubeflow-manifests/docs/deployment) in the Kubeflow on AWS documentation\. 

 After you have set up your Amazon EKS cluster, you can verify that your context points to your cluster in the following section\. 

## Verify cluster connection<a name="deep-learning-containers-eks-kubeflow-setup-test"></a>

These steps show how to verify your context\. This is to make sure that you interact with the correct cluster\. 

1. First, confirm that the cluster is active by running the following command\.

   ```
   $ aws eks --region <region> describe-cluster --name <cluster-name> --query cluster.status
   ```

   You should see the following output\.

   ```
   "ACTIVE"
   ```

1. To check your current context, run this command\. The `current-context` field in the output should contain your cluster name\. 

   ```
   $ kubectl config view
   ```

   If your `current-context` is not the cluster you want to interact with, run the following command to update it\. For more information about updating your `kubeconfig`, visit [Amazon EKS documentation](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig) 

   ```
   $ aws eks update-kubeconfig --region <region> --name <cluster-name>
   ```

After you have deployed Kubeflow on AWS and updated your current context, verify that your Kubeflow user profile uses the right namespace in the following section\. 

## Verify your namespace<a name="deep-learning-containers-eks-kubeflow-namespace-test"></a>

These steps show how to verify that your active Kubeflow user profile uses the namespace `kubeflow-user-example-com`\. All tutorials in this series run in this namespace\. 

1. 
**Note**  
 In Kubeflow, all namespaces should be created via [profiles](https://www.kubeflow.org/docs/components/multi-tenancy/getting-started/#manual-profile-creation )\. Kubeflow on AWS Vanilla installation creates a user profile with the namespace `kubeflow-user-example-com` by default\. 

    Ensure that a namespace named `kubeflow-user-example-com` exists by running the following command\. 

   ```
   $ kubectl get namespace
   ```

   If the namespace does not appear in the output, create a new Kubeflow profile as follows\. 

1. Open `vi` or `vim`, then copy and paste the following content\. Save this profile description file as `profile.yaml`\. Make sure to replace the email under `owner.name` with your email\. 

   ```
   apiVersion: kubeflow.org/v1beta1
   kind: Profile
   metadata:
    # replace with the name of profile you want, this is the user's namespace name
    name: kubeflow-user-example-com
   spec:
    owner:
        kind: User
        # replace with the email of the user
        name: user@example.com
   ```

1. Run the following command to create the corresponding profile resource\.

   ```
   $ kubectl apply -f profile.yaml
   ```

1. Export the `NAMESPACE` variable\.

   ```
   $ export NAMESPACE=kubeflow-user-example-com
   ```

   We refer to this namespace as the variable `${NAMESPACE}` in all Kubeflow on AWS tutorials\.

## Next steps<a name="deep-learning-containers-eks-setup-next"></a>

Now that you have finished the Kubeflow on AWS setup, you can continue with the training and inference tutorials\. 

To learn about training and inference with Deep Learning Containers on Kubeflow on AWS, see the [Training](deep-learning-containers-eks-kubeflow-tutorials-training.md) or [Inference](deep-learning-containers-eks-kubeflow-tutorials-inference.md) guides\.

## Cleanup<a name="deep-learning-containers-eks-kubeflow-cleanup"></a>

 This section provides cleanup instructions after you have finished running your tutorials\. 

### Clean Jobs<a name="deep-learning-containers-eks-kubeflow-cleaning-jobs"></a>

You can delete a specific training job when you are done running an example\. To list the jobs of a specific type \(PyTorchJob, MPIJob, TfJob\) running in a given namespace, run the following command\. 

```
$ kubectl get job_type -n ${NAMESPACE}
```

Retrieve the name of the job you want to delete, then run the following command\.

```
$ kubectl delete job_type job_name -n ${NAMESPACE}
```

Your output should look similar to the following\.

```
job_type.kubeflow.org "job_name" deleted
```

### Uninstall Kubeflow on AWS<a name="deep-learning-containers-eks-kubeflow-uninstall"></a>

Kubeflow on AWS documentation provides uninstall commands\. Make sure that you run the command that corresponds to your deployment method: [Kustomize, Helm](https://awslabs.github.io/kubeflow-manifests/docs/deployment/vanilla/guide/#uninstall-kubeflow-on-aws), or [Terraform](https://awslabs.github.io/kubeflow-manifests/docs/deployment/vanilla/guide-terraform/#cleanup)\. 

### Delete an Amazon EKS cluster<a name="deep-learning-containers-eks-kubeflow-delete-cluster"></a>

Kubeflow on AWS documentation provides a single command to [delete your entire Amazon EKS cluster](https://awslabs.github.io/kubeflow-manifests/docs/deployment/vanilla/guide/#optional-delete-amazon-eks-cluster)\. 

**Topics**
+ [Deploy Kubeflow on AWS](#deep-learning-containers-eks-kubeflow-deploy)
+ [Verify cluster connection](#deep-learning-containers-eks-kubeflow-setup-test)
+ [Verify your namespace](#deep-learning-containers-eks-kubeflow-namespace-test)
+ [Next steps](#deep-learning-containers-eks-setup-next)
+ [Cleanup](#deep-learning-containers-eks-kubeflow-cleanup)
+ [Training](deep-learning-containers-eks-kubeflow-tutorials-training.md)
+ [Inference](deep-learning-containers-eks-kubeflow-tutorials-inference.md)