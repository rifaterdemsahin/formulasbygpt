Here’s a step-by-step guide to implementing a basic **Thanos Receiver** deployment using **Docker** and deploying it on **OpenShift CRC**. Thanos is typically used for scaling Prometheus setups and enabling long-term storage. The **receiver** component ingests remote write requests from Prometheus.

### Step 1: Install and Set Up OpenShift CRC

Ensure that OpenShift CRC (CodeReady Containers) is installed and running. If you haven’t set it up yet, follow the official [Red Hat documentation for CRC](https://crc.dev/crc/).

1. **Download and Install CRC**:
   - Download CRC from the official website.
   - Install CRC on your local machine:
     ```bash
     crc setup
     ```

2. **Start OpenShift CRC**:
   ```bash
   crc start
   ```

3. **Login to OpenShift CRC**:
   ```bash
   crc console --credentials
   ```
   This will show the credentials you need to log into the OpenShift web console. Alternatively, use the following command:
   ```bash
   oc login -u developer -p developer
   ```

### Step 2: Prepare a Docker Image for Thanos Receiver

Before deploying Thanos Receiver to OpenShift CRC, let’s create a Docker container for it.

1. **Create a `Dockerfile` for Thanos**:
   You can use the official Thanos image or create a custom Dockerfile. Here’s an example:

   ```Dockerfile
   FROM quay.io/thanos/thanos:v0.30.2

   # Define the entrypoint for the receiver
   ENTRYPOINT [ "/bin/thanos" ]
   CMD [ "receive", "--http-address=0.0.0.0:10902", "--grpc-address=0.0.0.0:10901", "--remote-write.address=0.0.0.0:19291" ]
   ```

2. **Build the Docker Image**:
   Build and tag your Thanos Receiver image.

   ```bash
   docker build -t thanos-receiver:latest .
   ```

### Step 3: Push the Docker Image to an Accessible Registry

Push your image to a container registry that OpenShift can access (like Docker Hub, Quay.io, or your private registry).

1. **Log in to the Docker Registry**:
   If you’re using Docker Hub or Quay.io:

   ```bash
   docker login
   ```

2. **Push the Image**:
   Tag and push the image to your registry.

   ```bash
   docker tag thanos-receiver:latest quay.io/your-username/thanos-receiver:latest
   docker push quay.io/your-username/thanos-receiver:latest
   ```

### Step 4: Deploy Thanos Receiver to OpenShift CRC

Now, let’s create a deployment in OpenShift CRC for the Thanos Receiver.

1. **Log into OpenShift CLI**:
   If you’re not already logged in:

   ```bash
   oc login -u developer -p developer
   ```

2. **Create a New OpenShift Project** (Optional):
   You can create a separate project (namespace) for your Thanos Receiver.

   ```bash
   oc new-project thanos-receiver
   ```

3. **Create a Deployment for Thanos Receiver**:
   Use `oc` or the OpenShift web console to create the deployment. Here’s an example YAML for a basic deployment:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: thanos-receiver
     labels:
       app: thanos-receiver
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: thanos-receiver
     template:
       metadata:
         labels:
           app: thanos-receiver
       spec:
         containers:
         - name: thanos-receiver
           image: quay.io/your-username/thanos-receiver:latest
           ports:
             - containerPort: 10901 # gRPC
             - containerPort: 10902 # HTTP
             - containerPort: 19291 # Remote Write
           args:
             - receive
             - --http-address=0.0.0.0:10902
             - --grpc-address=0.0.0.0:10901
             - --remote-write.address=0.0.0.0:19291
   ```

   Save the above configuration as `thanos-receiver-deployment.yaml`.

4. **Apply the Deployment**:
   Deploy the Thanos Receiver using the following command:

   ```bash
   oc apply -f thanos-receiver-deployment.yaml
   ```

5. **Create a Service for Thanos Receiver**:
   Expose the Thanos Receiver service so that it can receive remote write requests. Here’s a service definition:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: thanos-receiver
   spec:
     selector:
       app: thanos-receiver
     ports:
       - protocol: TCP
         port: 10901
         targetPort: 10901
         name: grpc
       - protocol: TCP
         port: 10902
         targetPort: 10902
         name: http
       - protocol: TCP
         port: 19291
         targetPort: 19291
         name: remote-write
   ```

   Save this as `thanos-receiver-service.yaml` and apply it:

   ```bash
   oc apply -f thanos-receiver-service.yaml
   ```

6. **Expose the Service** (Optional for External Access):
   If you want to expose Thanos to external Prometheus instances, you can create a route in OpenShift:

   ```bash
   oc expose svc/thanos-receiver --port=19291
   ```

### Step 5: Connect Prometheus to Thanos Receiver

To send data from Prometheus to Thanos Receiver, modify your **Prometheus configuration** to include the `remote_write` section:

```yaml
remote_write:
  - url: "http://thanos-receiver.thanos-receiver.svc:19291/api/v1/receive"
```

You will need to replace the URL with the appropriate service URL for Thanos Receiver. If you’ve exposed it via a route, use the route URL.

### Step 6: Monitor the Deployment

1. **Check Deployment Status**:
   Make sure the Thanos Receiver is running:

   ```bash
   oc get pods
   ```

2. **View Logs**:
   You can view the logs to ensure Thanos Receiver is running correctly:

   ```bash
   oc logs deployment/thanos-receiver
   ```

3. **Verify Prometheus Remote Write**:
   Ensure that Prometheus is successfully sending data to Thanos Receiver by checking Prometheus and Thanos Receiver logs.

### Conclusion

You now have a basic deployment of Thanos Receiver running on OpenShift CRC, ready to receive remote write requests from Prometheus.