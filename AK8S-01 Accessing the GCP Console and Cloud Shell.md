# AK8S-01 Accessing the GCP Console and Cloud Shell
In this lab, you learn how to perform the following tasks
* Learn how to access the GCP Console and Cloud Shell
* Become familiar with the GCP Console
* Become familiar with Cloud Shell features, including the Cloud Shell code editor
* Use the GCP Console and Cloud Shell to create buckets and VMs and service accounts
* Perform other commands in Cloud Shell
## Create a Storage Bucket on Cloud Shell
First store The name for the storage bucket in a variable along with the location for use later
```shell
MY_BUCKET_NAME_1=unique_name_for_storage bucket.
MY_REGION=us-central1
```
Run the command to create the storage bucket
```shell
gsutil mb gs://$MY_BUCKET_NAME_1
```
## Create Virtual machine (VM) instance
Run the following command to create a VM instance
```shell
gcloud beta compute --project=qwiklabs-gcp-01-f7b491847435 instances create first-vm --zone=us-central1-c --machine-type=n1-standard-2 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=958171576124-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=first-vm --reservation-affinity=any
```
Next setup firewall rule to allow traffic into the VM
```shell
gcloud compute --project=qwiklabs-gcp-01-f7b491847435 firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```
## Create second VM instance and Storage Bucket
### Create Second Storage Bucket
```shell
MY_BUCKET_NAME_2=unique_name_for_2_storage_bucket
MY_REGION=us-central1
gsutil mb gs://$MY_BUCKET_NAME_2
```
### Create Second Virtual Machine
In Cloud Shell, execute the following command to list all the zones in a given region:
```shell
gcloud compute zones list | grep $MY_REGION
```
Select a zone from the first column of the list. Notice that GCP zones' names consist of their region name, followed by a hyphen and a letter.
Execute the following command to store your chosen zone in an environment variable.
```MY_ZONE=[ZONE]```
Set this zone to be your default zone by executing the following command.
```shell
gcloud config set compute/zone $MY_ZONE
```
Execute the following command to store a name in an environment variable you will use to create a VM. 
```shell
MY_VMNAME=second-vm
```
Create a VM in the default zone that you set earlier in this task using the new environment variable to assign the VM name.
```shell
gcloud compute instances create $MY_VMNAME \
--machine-type "n1-standard-1" \
--image-project "debian-cloud" \
--image-family "debian-9" \
--subnet "default"
```
List the virtual machine instances in your project.
```shell
gcloud compute instances list
```
## Create A Service Account
In Cloud Shell, execute the following command to create a new service account:
```shell
gcloud iam service-accounts create test-service-account2 --display-name "test-service-account2"
```
In Cloud Shell, execute the following command to grant the second service account the Project viewer role:
```shell
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT --member serviceAccount:test-service-account2@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com --role roles/viewer
```
## Download a file to Cloud Shell and copy it to Cloud Storage
Copy a picture of a cat from a Google-provided Cloud Storage bucket to your Cloud Shell.
```shell
gsutil cp gs://cloud-training/ak8s/cat.jpg cat.jpg
```
Copy the file into one of the buckets that you created earlier.
```shell
gsutil cp cat.jpg gs://$MY_BUCKET_NAME_1
```
Copy the file from the first bucket into the second bucket:
```shell
gsutil cp gs://$MY_BUCKET_NAME_1/cat.jpg gs://$MY_BUCKET_NAME_2/cat.jpg
```
## Set the access control list for a Cloud Storage object
To get the default access list that's been assigned to cat.jpg (when you uploaded it to your Cloud Storage bucket), execute the following two commands:
```shell
gsutil acl get gs://$MY_BUCKET_NAME_1/cat.jpg  > acl.txt
cat acl.txt
```
To change the object to have private access, execute the following command:
```shell
gsutil acl set private gs://$MY_BUCKET_NAME_1/cat.jpg
```
To verify the new ACL that's been assigned to cat.jpg, execute the following two commands:
```shell
gsutil acl get gs://$MY_BUCKET_NAME_1/cat.jpg  > acl-2.txt
cat acl-2.txt
```
## Authenticate as a service account in Cloud Shell
In Cloud Shell, execute the following command to view the current configuration:
```shell
gcloud config list
```
To verify the list of authorized accounts in Cloud Shell, execute the following command:
```shell
gcloud auth list
```
To verify that the current account (test-service-account) cannot access the cat.jpg file in the first bucket that you created, execute the following command:
```shell
gsutil cp gs://$MY_BUCKET_NAME_1/cat.jpg ./cat-copy.jpg
```
Verify that the current account (test-service-account) can access the cat.jpg file in the second bucket that you created:
```shell
gsutil cp gs://$MY_BUCKET_NAME_2/cat.jpg ./cat-copy.jpg
```
To switch to the lab account, execute the following command.
```shell
gcloud config set account [USERNAME]
```
To verify that you can access the cat.jpg file in the [BUCKET_NAME] bucket (the first bucket that you created), execute the following command.
```shell
gsutil cp gs://$MY_BUCKET_NAME_1/cat.jpg ./copy2-of-cat.jpg
```
Make the first Cloud Storage bucket readable by everyone, including unauthenticated users.
```shell
gsutil iam ch allUsers:objectViewer gs://$MY_BUCKET_NAME_1
```
## Open the Cloud Shell code editor
In Cloud Shell, click the Open in new window icon on the top right. Then click the pencil icon to open the Cloud Shell code editor.
In Cloud Shell, execute the following command to clone a git repository:
```shell
git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git
```
In Cloud Shell, execute the following command to create a test directory:
```shell
mkdir test
```
In the Cloud Shell code editor, click the arrow to the left of orchestrate-with-kubernetes to expand the folder.
Click the cleanup.sh file to open it in the right pane of the Cloud Shell code editor window.
Add the following text as the last line of the cleanup.sh file:
```shell
echo Finished cleanup!
```
In Cloud Shell, execute the following commands to change directory and display the contents of cleanup.sh:
```shell
cd orchestrate-with-kubernetes
cat cleanup.sh
```
Verify that the output of cat cleanup.sh includes the line of text that you added.
Return to your Cloud Shell home directory.
```shell
cd
```
In the Cloud Shell code editor, click to open the File menu and choose New File. Name the file index.html.
In the right hand pane, paste in this HTML text:
```HTML
<html><head><title>Cat</title></head>
<body>
<h1>Cat</h1>
<img src="REPLACE_WITH_CAT_URL">
</body></html>
```
Replace the string REPLACE_WITH_CAT_URL with the URL of the cat image from an earlier task.
On the Navigation menu (Navigation menu ), click Compute Engine > VM instances.
In the row for your first VM, click the SSH button.
In the SSH login window that opens on your VM, install the nginx Web server:
```shell
sudo apt-get update
sudo apt-get install nginx
```
In your Cloud Shell window, copy the HTML file you created using the Code Editor to your virtual machine:
```shell
cd orchestrate-with-kubernetes
gcloud compute scp index.html first-vm:index.nginx-debian.html --zone=us-central1-c
```
In the SSH login window for your VM, copy the HTML file from your home directory to the document root of the nginx Web server:
```shell
sudo cp index.nginx-debian.html /var/www/html
```
On the Navigation menu (Navigation menu ), click Compute Engine > VM instances. Click the link in the External IP column for your first VM. A new browser tab opens, containing a Web page that contains the cat image.
