# AK8S-04 Creating a GKE Cluster via Cloud Shell
## Deploy GKE clusters
In Cloud Shell, type the following command to set the environment variable for the zone and cluster name.
```shell
export my_zone=us-central1-a
export my_cluster=standard-cluster-1
```
In Cloud Shell, type the following command to create a Kubernetes cluster.
```shell
gcloud container clusters create $my_cluster --num-nodes 3 --zone $my_zone --enable-ip-alias
```
## Modify GKE clusters
In Cloud Shell, execute the following command to modify standard-cluster-1 to have four nodes:
```shell
gcloud container clusters resize $my_cluster --zone $my_zone --size=4
```
## Connect to a GKE cluster
To create a kubeconfig file with the credentials of the current user (to allow authentication) and provide the endpoint details for a specific cluster (to allow communicating with that cluster through the kubectl command-line tool), execute the following command:
```shell
gcloud container clusters get-credentials $my_cluster --zone $my_zone
```
Open the kubeconfig file with the nano text editor:
```shell
nano ~/.kube/config
```
## Use kubectl to inspect a GKE cluster
In Cloud Shell, execute the following command to print out the content of the kubeconfig file:
```shell
kubectl config view
```
In Cloud Shell, execute the following command to print out the active context:
```shell
kubectl config current-context
```
In Cloud Shell, execute the following command to print out some details for all the cluster contexts in the kubeconfig file:
```shell
kubectl config get-contexts
```
In Cloud Shell, execute the following command to change the active context:
```shell
kubectl config use-context gke_${GOOGLE_CLOUD_PROJECT}_us-central1-a_standard-cluster-1
```
In Cloud Shell, execute the following command to view the resource usage across the nodes of the cluster:
```shell
kubectl top nodes
```
In Cloud Shell, execute the following command to enable bash autocompletion for kubectl:
```shell
source <(kubectl completion bash)
```
## Deploy Pods to GKE clusters
### Use kubectl to deploy Pods to GKE
In Cloud Shell, execute the following command to deploy the latest version of nginx as a Pod named nginx-1:
```shell
kubectl run nginx-1 --image nginx:latest
```
In Cloud Shell, execute the following command to view all the deployed Pods in the active context cluster:
```shell
kubectl get pods
```
You will now enter your Pod name into a variable that we will use throughout this lab.
```shell
export my_nginx_pod=nginx-1
```
In Cloud Shell, execute the following command to view the complete details of the Pod you just created.
```shell
kubectl describe pod $my_nginx_pod
```
### Push a file into a container
In Cloud Shell, type the following commands to open a file named test.html in the nano text editor.
```shell
nano ~/test.html
```
Add the following text (shell script) to the empty test.html file:
```HTML
<html> <header><title>This is title</title></header>
<body> Hello world </body>
</html>
```
Press CTRL+X, then press Y and enter to save the file and exit the nano editor.
In Cloud Shell, execute the following command to place the file into the appropriate location within the nginx container in the nginx Pod to be served statically:
```shell
kubectl cp ~/test.html $my_nginx_pod:/usr/share/nginx/html/test.html
```
#### Expose the Pod for testing
In Cloud Shell, execute the following command to create a service to expose our nginx Pod externally:
```shell
kubectl expose pod $my_nginx_pod --port 80 --type LoadBalancer
```
In Cloud Shell, execute the following command to view details about services in the cluster:
```shell
kubectl get services
```
In Cloud Shell, execute the following command to verify that the nginx container is serving the static HTML file that you copied.
```shell
curl http://[EXTERNAL_IP]/test.html
```
In Cloud Shell, execute the following command to view the resources being used by the nginx Pod:
```shell
kubectl top pods
```
## Introspect GKE Pods
### Prepare the environment
In Cloud Shell enter the following command to clone the repository to the lab Cloud Shell.
```shell
git clone https://github.com/GoogleCloudPlatformTraining/training-data-analyst
```
Change to the directory that contains the sample files for this lab.
```shell
cd ~/training-data-analyst/courses/ak8s/04_GKE_Shell/
```
A sample manifest YAML file for a Pod called new-nginx-pod.yaml has been provided for you:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: new-nginx
  labels:
    name: new-nginx
spec:
  containers:
  - name: new-nginx
    image: nginx
    ports:
    - containerPort: 80
```
To deploy your manifest, execute the following command:
```shell
kubectl apply -f ./new-nginx-pod.yaml
```
To see a list of Pods, execute the following command:
```shell
kubectl get pods
```
### Use shell redirection to connect to a Pod
In Cloud Shell, execute the following command to start an interactive bash shell in the nginx container:
```shell
kubectl exec -it new-nginx /bin/bash
```
In Cloud Shell, in the nginx bash shell, execute the following commands to install the nano text editor:
```shell
apt-get update
apt-get install nano
```
In Cloud Shell, in the nginx bash shell, execute the following commands to switch to the static files directory and create a test.html file:
```shell
cd /usr/share/nginx/html
nano test.html
```
In Cloud Shell, in the nginx bash shell nano session, type the following text:
```HTML
<html> <header><title>This is title</title></header>
<body> Hello world </body>
</html>
```
Press CTRL+X, then press Y and enter to save the file and exit the nano editor.
In Cloud Shell, in the nginx bash shell, execute the following command to exit the nginx bash shell:
```shell
exit
```
In Cloud Shell, execute the following command to set up port forwarding from Cloud Shell to the nginx Pod (from port 10081 of the Cloud Shell VM to port 80 of the nginx container):
```shell
kubectl port-forward new-nginx 10081:80
```
In the Cloud Shell menu bar, click the plus sign (+) icon to start a new Cloud Shell session.
In the second Cloud Shell session, execute the following command to test the modified nginx container through the port forwarding:
```shell
curl http://127.0.0.1:10081/test.html
```
### View the logs of a Pod
In the Cloud Shell menu bar, click the plus sign (+) icon to start another new Cloud Shell session.
In third Cloud Shell window, execute the following command to display the logs and to stream new logs as they arrive (and also include timestamps) for the new-nginx Pod:
```shell
kubectl logs new-nginx -f --timestamps
```
You will see the logs display in this new window
Close the third Cloud Shell window to stop displaying the log messages.
Close the original Cloud Shell window to stop the port forwarding process.
