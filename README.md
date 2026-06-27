# Lab: Scaling and Updating Applications

## 🎯 Objectives

In this lab you will learn how to:

* Scale applications using ReplicaSets
* Perform rolling updates
* Store configuration using ConfigMaps
* Autoscale applications using Horizontal Pod Autoscaler (HPA)

---

# 1. Environment Setup

Open a terminal.

Move to the project directory:

"cd /home/project"

Clone the repository if it does not already exist:

"[ ! -d 'CC201' ] && git clone https://github.com/ibm-developer-skills-network/CC201.git"

Navigate to the lab directory:

"cd CC201/labs/3_K8sScaleAndUpdate/"

Verify the files:

"ls"

Check your namespace:

"echo $MY_NAMESPACE"

---

# 2. Deploy the Application

Edit:

"deployment.yaml"

Replace:

"<my_namespace>"

with your actual namespace.

Deploy the application:

"kubectl apply -f deployment.yaml"

Verify the pod status:

"kubectl get pods"

Wait until the pod status becomes:

"Running"

---

# 3. Expose the Application

Create a Kubernetes service:

"kubectl expose deployment/hello-world"

This creates a ClusterIP service.

---

# 4. Start Kubernetes Proxy

Open a second terminal.

Run:

"kubectl proxy"

Keep this terminal running throughout the lab.

---

# 5. Access the Application

From the first terminal:

"curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy"

Expected output:

"Hello world from hello-world-xxxxxxxx-xxxx. Your app is up and running!"

---

# 6. Scale the Application

Increase replicas:

"kubectl scale deployment hello-world --replicas=3"

Verify scaling:

"kubectl get pods"

You should now see three running pods.

Test load balancing:

"for i in `seq 10`; do curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy; done"

Different pod names should appear.

Scale down:

"kubectl scale deployment hello-world --replicas=1"

Verify:

"kubectl get pods"

Only one pod should remain.

---

# 7. Perform Rolling Updates

Edit:

"app.js"

Change:

"'Hello world from ' + hostname + '! Your app is up and running!\n'"

to:

"'Welcome to ' + hostname + '! Your app is up and running!\n'"

Build and push version 2:

"docker build -t us.icr.io/$MY_NAMESPACE/hello-world:2 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:2"

Verify images:

"ibmcloud cr images"

Update the deployment:

"kubectl set image deployment/hello-world hello-world=us.icr.io/$MY_NAMESPACE/hello-world:2"

Check rollout status:

"kubectl rollout status deployment/hello-world"

View deployment details:

"kubectl get deployments -o wide"

Verify that image tag:

"2"

appears.

Test the application:

"curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy"

---

# 8. Roll Back the Deployment

Undo the update:

"kubectl rollout undo deployment/hello-world"

Check rollout status:

"kubectl rollout status deployment/hello-world"

Verify deployment:

"kubectl get deployments -o wide"

Ensure image tag:

"1"

is active.

Test the application:

"curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy"

---

# 9. Create a ConfigMap

Create a ConfigMap:

"kubectl create configmap app-config --from-literal=MESSAGE="This message came from a ConfigMap!""

Edit:

"deployment-configmap-env-var.yaml"

Replace:

"<my_namespace>"

with your namespace.

---

# 10. Configure Environment Variables

The deployment uses:

"envFrom:

* configMapRef:
  name: app-config"

Edit:

"app.js"

Replace:

"res.send('Welcome to ' + hostname + '! Your app is up and running!\n')"

with:

"res.send(process.env.MESSAGE + '\n')"

---

# 11. Build Version 3

Build and push the image:

"docker build -t us.icr.io/$MY_NAMESPACE/hello-world:3 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:3"

Apply the updated deployment:

"kubectl apply -f deployment-configmap-env-var.yaml"

Test the application:

"curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy"

Expected output:

"This message came from a ConfigMap!"

---

# 12. Update the ConfigMap

Delete and recreate:

"kubectl delete configmap app-config && kubectl create configmap app-config --from-literal=MESSAGE="This message is different, and you didn't have to rebuild the image!""

Restart the deployment:

"kubectl rollout restart deployment hello-world"

Test again:

"curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy"

---

# 13. Configure Horizontal Pod Autoscaler

Edit:

"deployment.yaml"

Add:

"name: http
resources:
limits:
cpu: 50m
requests:
cpu: 20m"

Apply the changes:

"kubectl apply -f deployment.yaml"

Create the autoscaler:

"kubectl autoscale deployment hello-world --cpu-percent=5 --min=1 --max=10"

Check the HPA:

"kubectl get hpa hello-world"

---

# 14. Generate Load

Ensure the proxy is running:

"kubectl proxy"

Open a third terminal.

Generate traffic:

"for i in `seq 100000`; do curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy; done"

Monitor autoscaling:

"kubectl get hpa hello-world --watch"

Observe the number of replicas increasing.

Stop monitoring:

"CTRL + C"

View final HPA status:

"kubectl get hpa hello-world"

---

# 15. Cleanup Resources

Stop the proxy and load-generation terminals.

Delete the deployment:

"kubectl delete deployment hello-world"

Delete the service:

"kubectl delete service hello-world"

---

# ✅ Lab Summary

In this lab you successfully:

* Deployed an application to Kubernetes.
* Exposed the application using a Service.
* Scaled pods using ReplicaSets.
* Performed rolling updates and rollbacks.
* Used ConfigMaps to manage configuration.
* Modified application behavior without rebuilding images.
* Implemented Horizontal Pod Autoscaling.
* Monitored resource-based scaling.
* Cleaned up Kubernetes resources.

---
