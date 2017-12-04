# The full pipeline
In last week's writeup, we introduced how to containerize each stage of our pipeline and why it's important. At the end, we also mentioned the necessity to incorporate Kubernetes(k8s) and Pachyderm into the pipeline. This writeup reviews the structure of our full pipeline, introduces the infrastructure after involving k8s and Pachyderm, and provides a hands-on example for deploying the pipeline.
## [Structure](https://github.com/hong142/MGMT690-Writeup-4-Yuwen-Hong) - Docker-ized pieces
Recall that our pipeline starts from images uploaded by different devices, and the first stage is to validate images and separate valid and invalid ones into different repositories. Then we use the valid images and a trained detection model as inputs to process the object detection stage, and output JSON files under folders of each devices recording objects detected. Here a probability threshold is needed to ensure only detected object with a probability higher than the threshold is recorded. The reason we choose to output a JSON file rather than another image with boxes on objects is to simplify the automating process. You may also output images with boxes, which should be a good addition. Next is the threat detection stage, which uses both the JSON files and detection rules as inputs. Currently, we set the rules to be the same among users, but it's realizable to maintain different rules for different users. This stage only outputs when a threat is detected, and we expect to get a JSON file with threats recorded and a notification to the email service SendGrid's API. The output JSON files are still stored under folders for each device to facilitate tracking and plots generating. All the monitoring plots are stored for future reference. All four stages, validation, object detection, threat detection, and plots generating are set up with Python scripts and [containerized using docker](https://github.com/hong142/MGMT690-Writeup-5-Yuwen-Hong), each with a docker image created. Those docker images are uploaded to a registry, and ready to be pulled down anywhere to run their functionalities, respectively.
<br />
<img src="https://github.com/hong142/MGMT690-Writeup-6-Yuwen-Hong/blob/master/1.png" width="600">
## Infrastructure
With docker images created, we need to automate the pipeline and get the right data to the right code (i.e. contianer) to be proceesed. The first problem to be sloved is where to run a ceratin docker image given a set of computer nodes. Ideally, the underlying resource is shared 
A set of comput nodes, instances, cloud vm swhre we going to run docker images, processing data. Share the underline resources, ideally more fo them per node if possible. Ec2 or gce instances, or digtal ocean. In the data center, google have server ranlks, has a ceratin amount cpu andmemroy, curve as cpu as vm, specifc size. Higher amout of memory and cpu expensive. One operating system, cetain baim with that vm, splict that reduce band. If we having data input and ouptu spit across  29 more cpu and meoryt to process faster, in an optimize ay. Cluster nodes.29:30 spreading procssseing across nodes, 
Docker images runs across nodes,Going ssh to enter each of this maiches and running docketr to get each of things run.  to get Annoying, scel those up. A thoudansd nodes, someones full time job. 
Final Infrastructure going touse tof]day
K8s is the base for the pipieline. the k8s clsuter consistes of a seriese of nodes, and things runing across nodes with one mstaer prossea run onit. When you are on your laptop, you can ianteranct with the master node via API. By tellling k8s what you wnat to run, and k8s will talk to mirrors to decide where to run each of those images and run then .

its kind of like magicalone of those nodes fils, automateilcaly self heal, it knows where everything is running so it will auto maticlalt shifts nodes. samrts  how things are schedule. We don’t wnot to depy each of thisngs mally across nodes, we want something do thsat for us. we can just tlk to one ppiece, k maseter and it can deploy evertyihg.
Right data to the right code, process in sequence as we designed. Pakcyderm, another docker iamge containear running on kbnet pockd,demen, we tell kunet iwant you to run one of this, run that here, instead of talking to kn derictly, we talik to this service, not only that I want to run this things which able  to process this data…… pd35:20.
Pd will gout and pull this things down deploy them on kalso feed in the data we wnt rto process and send it to the right on eof this piece to be proceed. K cluster, what thatdooes, we had object storage where we will store data pd another contieanrruning on k pod, a sest of contieanrs running on a node. For our purspe ita a ocnatiern.
On top of k ,we have pd running, interacit with that, pd will spine up piepielie workers on top of those nodes, which are just our contanirers the ones we defined it gona pull in and out data we obkject dstore and gieve it to those conatiers when they nedds to process the data, 
In summry, we mange ato find a way to dploy all fo our set fo processing stages. In a unified way wioytpur .. into a bunjh of mahcines.we managead how to get right data to one of theose procesisgn stages. In the sequence we want. Insteraftuce .

etcd databse, under the hood perkderm use the databse to keep track what data iis where what jobs I s
s coming how ans when is processed, is other data need to be processd. Super quink, store the amtdata volume atched to one fo the , acess very uqicky
we don’t real care ahrere this are running, its k job. Pipeline worker one is the running instance tof this container. 2..
the mistobjeective way to do hta t is to spine up a buch aof clusters of varies sizews. And piepilient o it. Under a silullized world. And look at how each of this perfoemcs. Gaterht that ta back. infrataure optimization,
underestimate , we have three nodes, all the sudden, we did increaser traffic, don’t have to stra up over, tell k to spine a new node or one or two, spperd things across those nodes. , autoscaling, more pressure on one node, set up autosaling, bring in aother node. How google runs things.
Gke clik a  button, you have  a cluster. google cluster, not supre involved, autosace, may be more effort. 
In summary, the process is you get your k cluster, and you tell k that 
want run pd , running o n k, start talking to k and starts tliaking to pd, piepielien I want to set up, and under that hood, pd willtkank to k do all the things. 
<img src="https://github.com/hong142/MGMT690-Writeup-6-Yuwen-Hong/blob/master/2.png" width="600">
## Hands on! Deploying the pipeline
### Check Kubernetes
We run k8s cluster as a base of our pipeline, and kubectl is a command line interface for us to interact with the master node wihtout installing k8s. Before deploying anything, you can use "get all" to check what is running on the k8s cluster. Instead, if you do "get pods", you can get containers that are running. In our case, pachd, etcd as well as dash are running, and Pachyderm is already connected to the object storage.
```
$ kubectl get all
```
### Check Pachyderm
We don’t have to talk to k8s directly, instead we can talk to Pachyderm and deploy the things we want directly. Just like the way we talk to k8s, one way you can talik to Pachyderm is through its command line interface called "pachctl". To get the version of both Pachyderm and its command line interface, you can run "version" through the interface.
```
$ pachctl version
```
### Create Input Repositories
Through "pachctl", you can start to create repositories for external input data (i.e. data that are direct input instead of output of any other stages). According to our design, we will need "images", "model" and "rules". Take "images" as a example. With the following cerate command, you essentially tell the Pachyderm to carve out a space in our object storage and call it "images", so that you can store input images there. This process doesn't physically split the storage space, but rather, Pachyderm will create a representation of the repository in the storage. You can run the same command for "model" and "rules", and then you can put the model and rules in the respective repos using the "put-file" command. At this stage, we you should have three repos with something in "model" and "rules". You can use "pico" commnad to open a file and make changes to it. For example, our file for rules doesn't include the email address to be sent to when a theret is detected, so we need to manully add our own email for testing. If you made any changes, you should always use "cat" command to review the file content to ensure all the changes are stored correctly.
```
$ pachctl create-repo images
$ pachctl put-file <repo> master -c -f <file>
```
Created Repos
<br />
<img src="https://github.com/hong142/MGMT690-Writeup-6-Yuwen-Hong/blob/master/4.png" width="600">
<br />
### Create Pipeline
Next is to create a pipeline for each stage by passing a JSON file. The JSON file documents details about how to carve the specific pipeline, inculding what the stage is, which container (i.e. Docker image) to use, what data to process and what command to run. When Pachyderm gets the contianer, it will talk to k8s to spin up a node and have that worker ready for processing data. And the worker just running the contianer we sepcified early. Based on our design, "validate", "object-detect", "threat-detect" and "plot" are four pipeline stages needed. Since we are using SendGrid to send the thereat alert email for us, we need to opne the "theret-detect" to get the API key set, so that we can reach out to SendGrid API. The you should run "update-pipeline" command to ensure the change is carreid through.
```
$ pachctl create-pipeline -f <pipelineName.json>
```
### Testing
At this point, the final pipeline is put together, and you can use "put-file" to commit test images. Assume everything goes correctly, those images will automatically be processed to generate notifications and plots.
