# The full pipeline
In last week's writeup, we introduced how to containerize each stage of our pipeline and why it's important. At the end, we also mentioned the necessity to incorporate Kubernetes(k8s) and Pachyderm into the pipeline. This writeup reviews the structure of our full pipeline, introduces the infrastructure after involving k8s and Pachyderm, and provides a hands-on example for deploying the pipeline.
## Reminders
In last week's writeup, spesicfc with names. Forgot object needs input trained model. Data reporscitory in perkderm. Spertate vailde, input valid iamges, model,use modle to detct object tin the vlaid iamges. Output, folers of difrent dieveces, joson file tells about wht objtect wea detced in the iamge, input into threate detection, rules as input. Simple, just one rule, not harad to implement difetn for diefernt user. Tow things. Notify sengiid notification API if thert, output josn only respons to therat output, keep trak of howmany tehts detected for each deivec. 
Docker image we crate last week fro validate stage, repo in gtihub. Infer to detect the square, put in some command line argument. Where input model is, where images input is, output directory. Threshold.
Changed bottom, script, detect ,make another image with box, not useful fro autpmate process, rather output a json file with classes scores detected. 100 things, with very little probability, set a therahold record detected wit hthis or graeter probability. Whatever you wna to set up. Extra, might also have image with box, embed the iamge in the image, good addition. 
Base image, python script, just one dependcy. Available on docker hub.mgmt-object-detect.
Python script threat, where jaon, output ,rule. Sendgrid API key. Load some rules, any of the classes in the list, move the file and send the email. Rule calss 1 person, vehincle, any secury company or users, from to email. Exract therat and compare, copy over to json and alert.a rule file per user, a single rule file with ID in it. Not toharad to implement, we alreay organizing json by device id, match certain id to rule file. Keep them same around. Line 20 to 25, match interaction. Pyothon packegae for sedngrid. 
Esting python image, adding send grisd packe and script to be run.
Same command.
Matplot how many file in each of derictoy, just count, link the ifle in derictoris, create plot, looping over file, count. Evrytime cahngne input wil be updae acoordingly.
Diferent base impat already have mtplot installed, add pyhotn, make iamge samller
<img src="https://github.com/hong142/MGMT690-Writeup-6-Yuwen-Hong/blob/master/1.png" width="350">
<img src="https://github.com/hong142/MGMT690-Writeup-6-Yuwen-Hong/blob/master/2.png" width="350">

## Docker-ized pieces


## Infrastructure
run docker build and push for each to docker hub. Each working on difernt piece, up load to registry, any one deploy pipeline, they pull down those sages. Each piece has a corresponidn g image. Each of piece runs whatecver functionality we discussed.
Final Infrastructure going touse tof]day
Python to cerate each piece, packeage up tyohn via docker into images that can be deployed anywhere. So far we ve created thise images, some pyton,  series of docekr iamges, decieded to use object store to store all of the data ascociate with procseeing. All little cylders are in there. S3, gcs, ociesn space24-25, get the right data to the right code to be procees, automate, don’t to maunly on local machine. 
A set of comput nodes, instances, cloud vm swhre we going to run docker images, processing data. Share the underline resources, ideally more fo them per node if possible. Ec2 or gce instances, or digtal ocean. In the data center, google have server ranlks, has a ceratin amount cpu andmemroy, curve as cpu as vm, specifc size. Higher amout of memory and cpu expensive. One operating system, cetain baim with that vm, splict that reduce band. If we having data input and ouptu spit across  29 more cpu and meoryt to process faster, in an optimize ay. Cluster nodes.29:30 spreading procssseing across nodes, 

Docker images runs across nodes,Going ssh to enter each of this maiches and running docketr to get each of things run.  to get Annoying, scel those up. A thoudansd nodes, someones full time job. 
kubernets, this set of nodes make it a kubernet cluster. A ceain et of things, kunert things running on each ofthis nodes, 
k runigng across nodes, one of this master process running on it. We on laptop can interact with masetr node and there is an api for that master node its decliretive, k, I want you to run this somewhere, k goes in and it talks to all little meniems,  where should I run this and it  detcide whre to run each of thos imgaes and runs them for us. For example, we can say, I want 17 of this container running, at all times, k will choose 17mirros. running acorss nodes, its kind of like magicalone of those nodes fils, automateilcaly self heal, it knows where everything is running so it will auto maticlalt shifts nodes. samrts  how things are schedule. We don’t wnot to depy each of thisngs mally across nodes, we want something do thsat for us. we can just tlk to one ppiece, k maseter and it can deploy evertyihg.
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

## Hands on! Deploying the pipeline
### 
kubernetes (k8s) 
kubectl is a command line interface for running commands against Kubernetes clusters
We run k8s cluster as a base of our pipeline, so before deploying anything, you can use get all to check what is running on the k8s cluster at the moment. In our case, Pachyderm is running.
kuctl command line tool leet us talk interacrtion between us and the mser node, we can do that via q ctl, ctl get pods, show us what coantiers are cruntly runign on our k clauser.  I already spine up the clusters and I told ku I want pd running. This guy. Couple of more things, there is htu is impoertant not talnle, dahsborad ruing. We have this pieces and pd ourning onptop of that pd is alrady connected to object storage. 
```
$ kubectl get all
```
How to get that up and running, go to pd . guides here.
### pd
pachctl
We don’t have to talk to k8s directly, instead we can talk to Pachyderm and deploy the things we want directly. Just like the way we talk to k8s, one way you can talik to Pachyderm is through its command line interface called "pachctl". To get the version of both Pachyderm and its command line interface, you can run "version" through the interface.
Dash is a pd dashboard, where you can create a data repository, like nice liellt egooi.
### create repository
Github, repository, take this, copy link, go back to your termial, get space lcone, paste that link, ls should see a directory. Juts pull down some of things we need to deploy things. Net run cd space direct, change apsnce to that directoyr.  You should see a buch of staff. Ls seewhats in there. Josn files, deal tiwh in just a sexond, first, 
There are three data reporstiroyties that are only input. Not the out put of other stages. Create repo, collections of version data. Don’t need to have k and pd installed, you will Hvaea the ku control and p contle, which are just like client tolls  aloow you to tlak to k and pd. Pd, little toll let us tlak to it. A clinet , just client server. We have three images mode and rules, just input of data. Caret those withpkctl caret repo, first is images, which just happened is forn p control I taold p I want you to ceuve out a piece of this obect store  call it images and where Iwill  store some input data.pctl list repo see there is one repo, imagmes nothing in it. Simiar thing for model and rule, after you do all of that, you should se thre repos with nothing in them. Pd just careting curve ou t allitle space, reapreoas areprsnetation of thie repos not actually spliiit the pace. Copy one by one. Pul donw rhte model provide as input to object detection, second coamnd, don’t rune acpmle urle one, first four. Pul down the model, gooint into that model directory. Pctl, put file. I have ythis file, you put this file in the model. File is a tensorflow model. Pctl list pasce repo, one of this has data in it. We put data int the model clyielrnar. … open that, put your email, save control o, pico space rule . Jason. Conrol-o , output this changes, enter, contlr-x
File cat space rule. 
```
$ pachctl create-repo reponame
```
### pipeline
Take next put command, utles master commad, rule into repo. Rpo should ssee spme data inrules repo. Ce .. / .. /ls json fiels in ther. Laptop, put some data in pd, tell pd I want ot processthis piece of d ata ith this ocntainre, ….etc. how we build up our piepieline. Cat valid.json, you should see something like this,just those thres pisce, specifection what to spine up. Here I’m telling packedrm I want you to caret a vialdate stage, as the conainter iawnt you to run this ocnatier, in that conatienr, iwant you to runthis comnad. Pyhotn command. Process this data, input is this repo called images. Craeate this piepieline use this doekcet image runignthis coamnd. How I careta this comnd, here I just telling what data to procees, if younotiec here, wenhn this onater comes up, pd intects data that coantiern needs to process/pfs. 
Next is to create a pipeline for each stage by passing a json file
Pd ctl create – pipielene -f, I will give you a file tiells you how to care this piepieline. And that file is …json. If you run that, then qctl get pods, I told pd to run this as an pipielien stage, pd got this contaers have k spine up on nodes andhas that worker ready fro processing. This is piepile workers just runign the ocntieanre we specifiy ealryere. Same exact thing object detct, threr dect plot. Sendgrid api reach out tosendgit and sent the eamiel. Pico, space, run thretst detc. sojsn I’m specify an enveroinmental for this things ot have acess to , hwo we create our api key here, copy api key, grab that key go back paste that between thoese coposes. Save thos eaagin.then you therect detct. Json should have the api key in it. Creraye/update that piepieline., ptcl apsce csre pirpele . f Jason. All the pipielien put togwheter, al the pods running, we can list pipieline se ethere is tuning piepie, li pore, we gor al thsose repo. We didn’t maunly create this repo, pd dose automaitally for us. Actuaolicaly carete output for us. Cd test_images ls aboucnh of iamges there. Now pactl put /file into iamges on the mster -c-f, copy the first image. Assuming everything wnet croectly, pc-job, automaitaly see runig. If any thing in imagemm iwna tyou to process this way. And we put something in there, I need to get to this ocntiendr to run it.  Goes all the way way down . pot pipieline, shoud get an email. 
```
$ pachctl create-pipeline -f name.json
```
