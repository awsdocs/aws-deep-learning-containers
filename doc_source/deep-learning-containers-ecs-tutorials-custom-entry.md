# Custom entrypoints<a name="deep-learning-containers-ecs-tutorials-custom-entry"></a>

For some images, Deep Learning Containers use a custom entrypoint script\. If you want to use your own entrypoint, you can override the entrypoint as follows\. 

Modify the `entryPoint` parameter in the JSON file that includes your task definition\. Include the file path to your custom entry point script\. An example is shown here\.

```
"entryPoint":[
           "sh",
           "-c",
           "/usr/local/bin/mxnet-model-server --start --foreground --mms-config /home/model-server/config.properties --models densenet=https://dlc-samples.s3.amazonaws.com/pytorch/multi-model-server/densenet/densenet.mar"],
```