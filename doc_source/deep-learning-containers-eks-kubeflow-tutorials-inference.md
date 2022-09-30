# Inference<a name="deep-learning-containers-eks-kubeflow-tutorials-inference"></a>

This guide shows how to run inference services on a PyTorch or TensorFlow model\.

If you have already created a cluster and deployed Kubeflow on AWS, you can begin this tutorial\. If not, follow the steps in [Kubeflow on AWS Setup](deep-learning-containers-eks-kubeflow-setup.md)\. Select a CPU or GPU example depending on your cluster setup\. Inference examples run on single node configurations\. 

## TensorFlow CPU Inference with KServe<a name="deep-learning-containers-eks-kubeflow-tutorials-cpu-inference-tf"></a>

KServe enables serverless inferencing on Kubernetes for common machine learning \(ML\) frameworks\. Frameworks include TensorFlow, XGBoost, or PyTorch\. KServe is pre\-installed with Kubeflow on AWS\. In this tutorial, you create a KServe service to run a TensorFlow model inference on a CPU cluster\. 

**Note**  
For this example, the service is exposed on a cluster\-internal IP `ClusterIP`\. In a production environment, you might need to expose inference services externally using a load balancer\. 

1. Create a service specification file named `tf_inference.yaml` with the following contents\. This example specifies the remote location of a model and the TensorFlow inference image that our inference service uses\. The model is a public example provided by KServe, and it can be used without modification\. 

   ```
   apiVersion: "serving.kserve.io/v1beta1"
   kind: "InferenceService"
   metadata:
     name: "flower-sample"
     annotations:
       sidecar.istio.io/inject: "false"
   spec:
     predictor:
       tensorflow:
         image: "763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-inference:2.9.0-cpu-py39-ubuntu20.04-ec2"
         storageUri: "gs://kfserving-examples/models/tensorflow/flowers"
   ```

1. Apply the service description to a pod in your cluster\.

   ```
   $ kubectl apply -f tf_inference.yaml -n ${NAMESPACE}      
   ```

   Your output appears as follows\.

   ```
   inferenceservice.serving.kserve.io/flower-sample created
   ```

1. Check the status of the inference service to ensure that it is `READY` by running the following command\. It might take few minutes for the inference service to come up\. 

   ```
   $ kubectl get inferenceservices flower-sample -n ${NAMESPACE}
   ```

    In the command output, the state of the deployment should be `true` under the `READY` column\. 

   ```
   NAME            URL                                                         READY   PREV   LATEST   PREVROLLEDOUTREVISION   LATESTREADYREVISION                     AGE
   flower-sample   http://flower-sample.kubeflow-user-example-com.example.com  True           100                              flower-sample-predictor-default-00001   3m31s
   ```

1. Check the status of the pod with the following command\.

   ```
   $ kubectl get pods -n ${NAMESPACE} 
   ```

   Confirm that the pod is in a `Running` state by checking the `STATUS` in the command output\. 

   ```
   NAME                                                              READY   STATUS    RESTARTS   AGE
   flower-sample-predictor-default-00001-deployment-76c89dc6c47cvl   3/3     Running   0          24s
   ```

1. To describe the pod further, run the following command\.

   ```
   $ kubectl describe pod pod_name -n ${NAMESPACE}
   ```

1. To access the inference service, forward a port from your container to your host machine\. In a typical inference service deployment, you most likely want to set up a more permanent solution using a load balancer\. This command runs continuously in the foreground of your terminal\. 

   ```
   $ INGRESS_GATEWAY_SERVICE=$(kubectl get svc --namespace istio-system --selector="app=istio-ingressgateway" --output jsonpath='{.items[0].metadata.name}')
   $ kubectl port-forward --namespace istio-system svc/${INGRESS_GATEWAY_SERVICE} 8080:80
   ```

1. Download an input sample data by using this command\. The command creates a file `flower_input.json` containing sample data\. 

   ```
   $ curl https://raw.githubusercontent.com/kserve/kserve/release-0.8/docs/samples/v1beta1/tensorflow/input.json -o flower_input.json       
   ```

1. In a separate terminal, log in to the inference service by creating and running the following script\.

   1. Open `vi` or `vim`, then copy and paste the script below in a file named `inference_authentication.py`\. The script triggers an OpenID Connect \(OIDC\) exchange resulting in a session cookie to authenticate inference requests\. 

      ```
      import requests
      import os
      import json
      
      CLUSTER_IP = os.environ.get("CLUSTER_IP", "localhost:8080")
      DASHBOARD_URL = f"http://{CLUSTER_IP}"
      NAMESPACE = os.environ.get("NAMESPACE", "kubeflow-user-example-com")
      MODEL_NAME = os.environ.get("MODEL_NAME", "sklearn-iris")
      SERVICE_HOSTNAME = os.environ.get("SERVICE_HOSTNAME", "flower-sample.kubeflow-user-example-com.example.com")
      URL = f"http://{CLUSTER_IP}/v1/models/{MODEL_NAME}:predict"
      HEADERS = {"Host": f"{SERVICE_HOSTNAME}"}
      USERNAME = os.environ.get("USERNAME", "user@example.com")
      PASSWORD = os.environ.get("PASSWORD", "12341234")
      
      def load_json_file(filepath):
          with open(filepath) as file:
              return json.load(file)
      
      data = load_json_file("./flower_input.json")
      
      response = None
      
      def session_cookie(host, login, password):
          session = requests.Session()
          response = session.get(host)
          headers = {
              "Content-Type": "application/x-www-form-urlencoded",
          }
          data = {"login": login, "password": password}
          session.post(response.url, headers=headers, data=data)
          session_cookie = session.cookies.get_dict()["authservice_session"]
          return session_cookie
      
      cookie = {"authservice_session": session_cookie(DASHBOARD_URL, USERNAME, PASSWORD)}
      response = requests.post(URL, headers=HEADERS, json=data, cookies=cookie)
      
      print("Sending request to:", URL)
      
      status_code = response.status_code
      print("Status Code", status_code)
      if status_code == 200:
          print("JSON Response ", json.dumps(response.json(), indent=2))
      ```

   1.  To run a prediction using the sample input data, run the script above using the commands below\. 

      ```
      export INGRESS_HOST=localhost
      export INGRESS_PORT=8080
      export CLUSTER_IP=${INGRESS_HOST}:${INGRESS_PORT}
      export NAMESPACE=kubeflow-user-example-com
      export MODEL_NAME=flower-sample
      export SERVICE_HOSTNAME=$(kubectl get -n ${NAMESPACE} inferenceservice ${MODEL_NAME} -o jsonpath='{.status.url}' | cut -d "/" -f 3)
      export USERNAME=user@example.com
      export PASSWORD=12341234
      ```

      ```
      $ pip install requests
      ```

      ```
      $ python3 ./inference_authentication.py
      ```

1.  The output displays the inference results\. 

   ```
   Sending request to: http://localhost:8080/v1/models/flower-sample:predict
   Status Code 200
   JSON Response  {
     "predictions": [
       {
         "prediction": 0,
         "key": "   1",
         "scores": [
           0.999114931,
           9.20989623e-05,
           0.000136786606,
           0.000337258854,
           0.000300534302,
           1.84814289e-05
         ]
       }
     ]
   }
   ```

See [Cleanup](deep-learning-containers-eks-kubeflow-setup.md#deep-learning-containers-eks-kubeflow-cleanup) for information on cleaning up a cluster after you are done using it\. 