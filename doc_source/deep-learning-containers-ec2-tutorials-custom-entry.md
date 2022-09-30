# Custom Entrypoints<a name="deep-learning-containers-ec2-tutorials-custom-entry"></a>

For some images, Deep Learning Containers uses a custom entrypoint script\. If you want to use your own entrypoint, you can override the entrypoint as follows\.
+ To specify a custom entrypoint script to run, use this command\.

  ```
  docker run --entrypoint=/path/to/custom_entrypoint_script -it <image> /bin/bash
  ```
+ To set the entrypoint to be empty, use this command\.

  ```
  docker run --entrypoint="" <image> /bin/bash
  ```