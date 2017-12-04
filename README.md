# The full pipeline
In last week's writeup, we introduced how to containerize each stage of our pipeline and why it's important. At the end, we also mentioned the necessity to incorporate Kubernetes(k8s) and Pachyderm into the pipeline. This writeup reviews the structure of our full pipeline, introduces the infrastructure after involving k8s and Pachyderm, and provides a hands-on example for deploying the pipeline.
## [Structure](https://github.com/hong142/MGMT690-Writeup-4-Yuwen-Hong) - Docker-ized pieces
Recall that our pipeline starts from images uploaded by different devices, and the first stage is to validate images and separate valid and invalid ones into different repositories. Then we use the valid images and a trained detection model as inputs to process the object detection stage, and output JSON files under folders of each devices recording objects detected. Here a probability threshold is needed to ensure only detected object with a probability higher than the threshold is recorded. The reason we choose to output a JSON file rather than another image with boxes on objects is to simplify the automating process. You may also output images with boxes, which should be a good addition. Next is the threat detection stage, which uses both the JSON files and detection rules as inputs. Currently, we set the rules to be the same among users, but it's realizable to maintain different rules for different users. This stage only outputs when a threat is detected, and we expect to get a JSON file with threats recorded and a notification to the email service SendGrid's API. The output JSON files are still stored under folders for each device to facilitate tracking and plots generating. All the monitoring plots are stored for future reference. All four stages, validation, object detection, threat detection, and plots generation are set up with Python scripts and [containerized using docker](https://github.com/hong142/MGMT690-Writeup-5-Yuwen-Hong), each with a docker image created. Those docker images are uploaded to a registry, and ready to be pulled down anywhere to run their functionalities, respectively.
<br />
<img src="https://github.com/hong142/MGMT690-Writeup-6-Yuwen-Hong/blob/master/1.png" width="600">
## Infrastructure
With docker images created, several problems remain unsolved. First, we need to automate the pipeline and get the right data to the right code (i.e. container) to be processed and gather the output properly. The second is where to run a certain docker image, given a set of computer nodes and scale deployment. Ideally, the underlying resource is shared so that a node can run multiple containers efficiently in terms of resource optimization. 

Kubernetes (k8s), a platform working with containers, will be used to take care of scale deployment and resource optimization. It will dynamically provide the most efficient way for nodes allocation. In addition, k8s is capable of monitoring and auto-recovering on operation of application. It knows where everything is running, so it will automatically shift nodes if one is down. K8s also enables data scheduling.

The k8s cluster consisting of a series of nodes is the base for the pipeline, and things running across nodes with one master process run on it. When you are on your laptop, you can interact with the master node via API. By telling k8s what you want to run, and k8s will talk to mirrors to decide where to run each of those images and run them. We no longer have to deploy each of things manually across nodes.

On top of k8s, we have Pakcyderm (pockd), which can be considered as another docker container running on k8s. Pakcyderm will ensure the right data is processed in the right sequence as we designed. Instead of talking to k8s directly, we inform Pakcyderm what process we want to run, and Pakcyderm will pull down all the things needed and deploy them on k8s by spinning up pipeline workers on top of k8s nodes. Also, Pakcyderm talks to the object storage, s3 in our case, to get data in and out with data versioning. Under the hood, Pakcyderm use the etcd database to keep track of data.

 
In summary, the entire process is you get your k8s cluster, and you tell k8s master that you want to run Pakcyderm on it. Then you give all your commands to Pakcyderm, which will pass those commands to k8s. Ultimately, we can run everything in an unified way anywhere. 

You might encounter underestimation issue, when the traffic increases suddenly and goes beyond your total node capacity. In that case, you just need to have k8s spin up another node. Usually, you can set up k8s auto-scaling beforehand.
<br />
<img src="https://github.com/hong142/MGMT690-Writeup-6-Yuwen-Hong/blob/master/2.png" width="600">
## Hands on! Deploying the pipeline
### Check Kubernetes
We run k8s cluster as a base of our pipeline, and kubectl is a command line interface for us to interact with the master node wihtout installing k8s. Before deploying anything, you can use "get all" to check what is running on the k8s cluster. Instead, if you do "get pods", you can get containers that are running. In our case, pachd, etcd as well as dash are running, and Pachyderm is already connected to the object storage.
```
$ kubectl get all
```
### Check Pachyderm
We don’t have to talk to k8s directly, instead we can talk to Pachyderm and deploy the things we want directly. Just like the way we talk to k8s, one way you can talk to Pachyderm is through its command line interface called "pachctl". To get the version of both Pachyderm and its command line interface, you can run "version" through the interface.
```
$ pachctl version
```
### Create Input Repositories
Through "pachctl", you can start to create repositories for external input data (i.e. data that are direct input instead of output of any other stages). According to our design, we will need "images", "model" and "rules". Take "images" as a example. With the following cerate command, you essentially tell the Pachyderm to carve out a space in your object storage and call it "images", so that you can store input images there. This process doesn't physically split the storage space, but rather, Pachyderm will create a representation of the repository in the storage. You can run the same command for "model" and "rules", and then you can put the model and rules in the respective repos using the "put-file" command. At this stage, you should have three repos with something in "model" and "rules". You can use "pico" commnad to open a file and make changes to it. For example, our file for rules doesn't include the email address to be sent to when a threat is detected, so we need to manually add our own email for testing. If you made any changes, you should always use "cat" command to review the file content to ensure all the changes are stored correctly.
```
$ pachctl create-repo images
$ pachctl put-file <repo> master -c -f <file>
```
Created Repos
<br />
<img src="https://github.com/hong142/MGMT690-Writeup-6-Yuwen-Hong/blob/master/4.png" width="600">
<br />
### Create Pipeline
Next is to create a pipeline for each stage by passing a JSON file. The JSON file documents details about how to carve the specific pipeline, including what the stage is, which container (i.e. Docker image) to use, what data to process and what command to run. When Pachyderm gets the container, it will talk to k8s to spin up a node and have that worker ready for processing data. And the worker just running the container we specified early. Based on our design, "validate", "object-detect", "threat-detect" and "plot" are four pipeline stages needed. Since we are using SendGrid to send the thereat alert email for us, we need to open the "threat-detect" to get the API key set, so that we can reach out to SendGrid API. Then you should run "update-pipeline" command to ensure the change is carried through.
```
$ pachctl create-pipeline -f <pipelineName.json>
```
### Testing
At this point, the final pipeline is put together, and you can use "put-file" to commit test images. Assume everything goes correctly, those images will automatically be processed to generate notifications and plots.
