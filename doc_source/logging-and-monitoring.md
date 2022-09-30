# Logging and Monitoring in AWS Deep Learning Containers<a name="logging-and-monitoring"></a>

Your AWS Deep Learning Containers does not come with monitoring utilities\. For information on monitoring, see [GPU Monitoring and Optimization](https://docs.aws.amazon.com//dlami/latest/devguide/tutorial-gpu.html), [Monitoring Amazon EC2](https://docs.aws.amazon.com//AWSEC2/latest/UserGuide/monitoring_ec2.html), [Monitoring Amazon ECS](https://docs.aws.amazon.com//AmazonECS/latest/developerguide/ecs-logging-monitoring.html), [Monitoring Amazon EKS](https://docs.aws.amazon.com//eks/latest/userguide/logging-monitoring.html), and [Monitoring Amazon SageMaker](https://docs.aws.amazon.com//sagemaker/latest/dg/sagemaker-incident-response.html)\. 

## Usage Tracking<a name="logging-and-monitoring-opt-out"></a>

The following Deep Learning Containers include code that allows AWS to collect the instance types, frameworks, framework versions, container types, and Python versions used for the containers\. No information on the commands used within the containers is collected or retained\. No other information about the containers is collected or retained\.
+ TensorFlow 1\.15
+ TensorFlow 2\.0
+ TensorFlow 2\.1
+ PyTorch 1\.2
+ PyTorch 1\.3\.1
+ MXNet 1\.6

To opt\-out of usage tracking, use a custom entrypoint to disable the call for the following services:
+ [Amazon EC2 Custom Entrypoints](https://docs.aws.amazon.com//dlami/latest/devguide/deep-learning-containers-ec2-tutorials-custom-entry.html)
+ [Amazon ECS Custom Entrypoints](https://docs.aws.amazon.com//dlami/latest/devguide/deep-learning-containers-ecs-tutorials-custom-entry.html)
+ [Amazon EKS Custom Entrypoints](https://docs.aws.amazon.com//dlami/latest/devguide/deep-learning-containers-eks-tutorials-custom-entry.html)

To opt out of usage tracking for TensorFlow \(version >=1\.15\), PyTorch \(version >= 1\.5\), and MXNet \(version >= 1\.6\) containers, you also need to set the `OPT_OUT_TRACKING` environment variable\.

```
OPT_OUT_TRACKING=true
```