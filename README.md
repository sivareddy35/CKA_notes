<img width="1177" height="206" alt="image" src="https://github.com/user-attachments/assets/5dc93bc2-25e4-4760-8e2b-547882048315" />

  <img width="487" height="263" alt="image" src="https://github.com/user-attachments/assets/2b072250-658e-4662-927c-1c52bc0eda82" />


* KodeKloud docs ref: https://notes.kodekloud.com/
* Kubernetess purpose is to host the applications in the form of containers in autiomated fashion so that we can easily as many instance of the applications as required to enable communication between different service within our application.
* There are many thing involved to work with:
* Master Node:
  * It is reponsible for managing kubernetes. It involves storing information regarding nodes and planning which container goes where, moinitoring the nodes and containers on them.
  * The master node does all of these using a set of componets together known as Control plane components.  
  * `etcd`: It is a database that stores in key value format.
  * `kube-scheduler`: identifies the right nodes to place a container on based on the container resource requirements, the worker node capacity and any other policies or constraints. Such as taint and tolerations and node-affinity and node-anti affinity rules.
  * `Node-Controllers`: These are responsible for on-boarding new nodes to the clusters and handling situations where nodes become unavailable or get destroyed.
  * `Replication-controller`: It ensureds that desired number of continers are always up and running.
  * `kube-api`: It is reposible for orchestrating all operations witin the cluster, it expose kubernets api which is used by external users to perform managemnet operations of the cluster and as well as the various controllers monitor the state of the cluster and making necessary changes as required by the worker nodes to communicate with the server.
* The applications are in the form of containers, different components that form the entire management system on the master node can be hosted in the form of master node.
* DNS service, networking solutions can be deployed in the form of containers. So we need softare that can run containers that is container runtime Interface(CRI). We need to install container runtime on all the nodes ie docker or supported equivalent installed on all the nodes the cluster, including maste node. It does not need docker always can be container d or rkt, etc.. 
* `kubelet`: It is an agent that runs on each node in the cluster and listen instructions from kube-api server, it deploys or destroys the containers on the nodes as required.
* `kube-api` server periodically fetches the reports from kubelet to monitor the status of the nodes and containers on them.
* `kube-proxy`: It ensure the necessary rules are inplaced on the worker nodes to allow the containers running on them to reach each other.
* Master(Manage, plan , schedule, Monitor Node) has etcd cluster, kube-api, control-manager, kube-scheduler.
* * Woeker Node( Host the applications as Containers) has kubelet (kube-proxy), Container Runtime Interface(CRI) (Docker, container d, rkt).

### etcd:
* It is distributed reliable key-value store which is simple, secure and fast
* `Traditional databases (relational)` are in the tabular format, which stores the data in the form of rows and colums. Ex: SQL, Relational DBs. Every time when new information is added entire table is get affected and leads to lot of empty cells. These have strict schema and performs complex queries using SQL delivers good performance however it is very regid in the form of flexibility and best for structured data.
* `Document store stores` in the form of document, each individual is a document. These files can be in any format and the changes in the one file will not affect the other.
* Document store stores the data in the form of Json format, these don't need to have schema defined, can be limiting interms of performing complex queries by joining different tables delivers good performance and is very flexible and best suited for semi structured data.
* `Key-Value Store` stores a value against a key. It does not necessarily have a schema and don't support complex quries but performace is super fast and is flexible we can store anything witout breaking out anything else or worrying about data structures.
* We can insall etcd by downloading binary from github releases page. When we run etcd is starts a service that listens a port on `2379` by default. We will ideally run it a system service or a pod in kubernetes cluster. We can then attach to any clients to etcd service.
* To store and restrive the information a default client comes with etcd is `etcd control client` or etcd ctl. It is command line for client for etcd which is used to store and restore key-value pairs.
  
  <img width="796" height="255" alt="image" src="https://github.com/user-attachments/assets/0e11ca65-558d-43a6-9b22-88eee417be1c" />
  
* To store the value run `./etcdctl put key1 value1` which creates an entry to the database.
* To retrive data run `./etcdctl get key1` and to view more `./etcdctl` arguments.
### etcd Versions:

 <img width="1070" height="212" alt="image" src="https://github.com/user-attachments/assets/7f8ca4f7-dcbd-4d09-81e6-5bbe2477b0f1" />

### etcd in Kuberntes:
 <img width="1136" height="586" alt="image" src="https://github.com/user-attachments/assets/c0baca5c-47b8-4983-a8c1-9befd902a8c9" />

* etcd stores the information regarding the nodes, pods, configs, secretes, accounts, roles, bindings, others 
* Every information which we get by running kubectl command is from etcd server and every change we make to the cluster such as adding additional nodes, deploying pods or replicasets are updated in the etcd server. Only once it is updated in the etcd server then only the change is considered as complete.  
*  Depending on how you setup our cluster etcd is deployed differently.
*  `setup - Manually`: When we setup kuberenetes deployment we deploy etcd by downloading binaries by ourself. Install and configuring etcd as a service in the master node by ourself.
 * There are many many options passed into the service. A number of them relate to certificates. Configuring etcd as a cluster (High availability), advertise-clent-url, this is address where etcd listens, it happens to be on the IP of the server and port 2379.
 * This is url this should be configured on kube-api server when it tries to reach etcd server.
* `setup - Kubeadm`: When we setup cluster with kubeadm, kubeadm deploys etcd server as a pod in kube-system namespace.
* Kubernetes stores the data in a specific directory structure. Root directory is a registry under that there are various kuberenets constraints. Such as minios/nodes, pods, replicaSets, Deployments, Roles, Secretes. 
* In high-availability environments we will have multiple master nodes in the cluster, then we will have multiple etcd instances spread across the master node.
* In this case make sure that each etcd know about each other by setting up the right parameter in the etcd configuration. The initial cluster option is where we must specify various different instances of the etcd services.
* ETCDCTL is a CLI tool used to interact with ETCD.ETCDCTL can interact with ETCD server using Version 2 and Version 3. by default supports Version 2.
* To change ETCDCTL version 3 run `export ETCDCTL_API=3`command.    
* Commands supports ETCD Version 2 are
  
        etcdctl backup
        etcdctl cluster-health
        etcdctl mk
        etcdctl mkdir
        etcdctl set
  
* Commands supports ETCD Version 3 are
  
        etcdctl snapshot save
        etcdctl endpoint health
        etcdctl get
        etcdctl put

* We must specify the path of the certificate file so that ETCD can authenticate to ETCD API server. These certificate files are available in etcd-master at the below path.
  
        --cacert /etc/kubernetes/pki/etcd/ca.crt
        --cert /etc/kubernetes/pki/etcd/server.crt
        --key /etc/kubernetes/pki/etcd/server.key
  
* Combined command
  
        kubectl exec etcd-controlplane -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / \
        --prefix --keys-only --limit=10 / \
        --cacert /etc/kubernetes/pki/etcd/ca.crt \
        --cert /etc/kubernetes/pki/etcd/server.crt \
        --key /etc/kubernetes/pki/etcd/server.key"
  
### Kube-apiserver:

 <img width="1152" height="308" alt="image" src="https://github.com/user-attachments/assets/93c77df8-5efd-4c35-acb4-6a4abcc66526" />

* It is primary management component in kubernetes.
* kubectl utility reaches to kube-api server first. Kube-api server authenticate the request, validates it and then retrives the data from etcd cluster and responds back with the requested information.
* We don't really need to use kubectl cli instead we can directly invoke kube-apis by sending a post request.
* When we create a pod the request is authenticated fist and validated in this case api-server creates a pod object without signing into a node, update the information in etcd server, update the user the pod is created.
* The scheduler continuously monitor api-server and realizes there is a new pod  with no node assigned. Scheduler identifies the right node to place the new node on and communicates that back to api server. Api-server then update the information in etcd cluster.
* Api-server then passes the information to kubelet in the appropriate worker node. Kubelet then creates pod on the node and instruct the container run time engine to deploy the application image.
* Once done kubelet update the status back to api-server and api - server then updates the data back in the cluster.
* A similar pattern is followed every time a change is requested. Kube-api server is at the center of all different tasks that needs to be performed to make a change in the cluster.
* Api-server is responsible for Authenticate User, Validate Request, Retrive data, Update ETCD, Scheduler, kubelet.
* Kube api-server is the only componet that directly interact with etcd datastrore. The other components such as scheduler, kube-control manager and kubelet uses api-server to perform their updates in the cluster in their respective areas.
* If we bootstrap our cluster using kubeadmin tool we don't need to know this but if we are setting up in the hardway then kube-api server is available as a binary in kubernetes release page.
* kube-api server runs with a lot of parameters.

### Kube-Controller-Manager:
  
 <img width="957" height="346" alt="image" src="https://github.com/user-attachments/assets/7bf5d089-d632-4563-b699-11f1c6818aea" />

* Depends on how you setup to view kube-api server varies if we setup with kubeadm tool, kubeadm deploys kube-api as pod in kube-system namespace on master node.

 <img width="1155" height="333" alt="image" src="https://github.com/user-attachments/assets/b7e13712-44ac-48ea-bb73-6b339e22290a" />

* In on kubeadm setup we can view kube-api server service located at `/etc/systemd/system/kube-apisever.service`.
* We can see the running processes and effective options by listing the process on the master node `ps -awx| grep kube-apiserver`
* Controller is a process that continuously monitor the state of the various components within the system and works towards bringing the whole system to the desired functioning state.

   <img width="939" height="237" alt="image" src="https://github.com/user-attachments/assets/c31724ea-7d32-4341-8cc1-ae6eb14c1849" />

* `Node-Controller`: It is responsible for monitoring the status of the node and taking necessary actions to keep the application running. It does with kube-api-server.
* Node-Controller check the status of the nodes every 5 seconds that way the node-controller monitor the health of the nodes, when it stops receiving the signal the node is marked as unreachable but it waits for 40 seconds before marking as unreachable.
* After a node is marked as unreachable a node is given 5 min to comeback on if it doesn't it removes the pod assigned to the node and provisions them on healthy ones if the pods are part of repliacasets.
* `Replication-Controller`: it is reponsible for monitoring the status of replicasets and ensures that the desired number of pods are available all times witin the set. If a pod dies it creates another.
* All the kubernets intelligences ie Deployment-Controller, Node-Controller, Service-Account-Controller, CronJob, Job-Controller, Stateful-Set, Namespace-Controller, Endpoint-Controller, PV-Protection Controller, RepliacaSet, PV-Binder-Controller, Replication-Controller which are build are implemented through these controllers.
* This can be considered as brain behind kubernetes.
* Controllers are packaged into a single process known as Kube-controller-manager, When we install kube-controller-manager differnt controllers get installed aswell.
* Download kube-controller manager from kubnernetes pages, extract and run it as a service it provides a list of options this is where you provide additional options to customize the controller.

 <img width="1030" height="251" alt="image" src="https://github.com/user-attachments/assets/39baafe1-1915-47dc-9dc0-ca1575680554" />

* Some default controllers are node-monitor-period and teh grace-period, eviction-timeout, etc.
* There is additional option called as controller where we can specify which controllkers to enable by default all of them are enabled but we can choose to enable a select few.
* Incase if any of the controller don't seems to work we can check here.

 <img width="851" height="191" alt="image" src="https://github.com/user-attachments/assets/cdf23d1f-3681-432f-84a3-0932fa71f019" />

* To view controller manager options depends on how you install kubernetes. In kubeadm tool kudeadmin deploys a kube-controller-manager as a pod in the kube system namespace on the master node.

 <img width="876" height="222" alt="image" src="https://github.com/user-attachments/assets/bc03e4f6-268d-4fb6-9594-ce2f51345ad4" />

* In non-kubeadm setup we can inspect the options by viewing kube-controller-manager service located at the services directory.
* We can also see the runing proccess and effective options by listing the processes on the master node. 

### Kube - Scheduler:
* It is responsible for scheduling pods on nodes ie the scheduler's only responsibility is to decides which pod goes to which node. It doesn't actually place the pod on the nodes that is the job of kubelet.
* Kubelet creates the pods and scheduler only decide which pod goes where. 
* In kubernetes scheduler decides which nodes the pods are placed on depending on certain criteria. We may have pods with different resource requirements.
* We can have nodes in the cluster dedicated to certain applications, scheduler looks for each pod and tries to find the best node for it.

  <img width="879" height="204" alt="image" src="https://github.com/user-attachments/assets/9d8c0199-f50d-4dbb-9224-981de61fc8c1" />

* Lets assume a pod requires a set up of CPU and memory requirements in this case scheduler goes throuh two phases to idenity the best node for the pod, in the first pahse scheduer tries to filter out the nodes that don't fit the profile for this pod. ex the nodes that don't have sufficient CPU and memory resources requested by the pod.
* So at first two small nodes are filtered out now we left with two nodes on which the pod can be placed.
* The scheduler ranks the nodes to identify the best fit node for the pod, it uses priority function to assign a scorer to the nodes on a scale of 1 to 10. Ex schedular calculate the amount of resources that would be free on the nodes after placing pod on them. In this case the one with higher free size will get better rank.
* We can customize the scheduler and can write our own as per our requirements. Resource Requirements and Limits, Taints and Tolerations, Node Selectors/Affinity.

 <img width="1089" height="377" alt="image" src="https://github.com/user-attachments/assets/9c789d86-bc3d-4294-b25f-ca932a2ac506" />

* To install scheduler we need to download kube-scheduler binary from kubernetes release page=, extract and run it as a servervice.
* When we run as a service we will spcify the scheduler config file. To view scheduler that depends on how we setup kubernetes. Kubeadm tool deploys scheduler as pod under kube-system namespace on the master node. We can see the options from `/etc/kubernetes/manifests/` folder.

### Kubelet:

 <img width="1086" height="211" alt="image" src="https://github.com/user-attachments/assets/0142553e-c96a-4006-8b83-61697c9642d0" />

* Kubelet in the kubernetes worker node registers the node with kubernetes cluster when it receives the instructions to load a container or pod on the node it requests the container run time engine which may be Docker to pull the required image and run an instance.
* Kubelet then continues to monitor the state of the pods and containers in it and reports to kube-api server on timely basis.
* To install kubelet if we are using kube-admn tool to deploy our cluster it does not automatically deploy kubelet this is the difference than other coponnents. 
* We must always manually install all the kubelet on our worker nodes, download the installer extract it and run it as a service.
* We can view kubelet processes and effective options by listing the process on the worker node and searching kubelet

### Kube - proxy:

 <img width="880" height="416" alt="image" src="https://github.com/user-attachments/assets/e2c044d2-7755-4317-bec0-82baaad4cdcd" />

* With in a kubernetes cluster every pod can reach every other pod this is accomplished by deploying a pod networking solution to the cluster.
* A pod network is an internal virtual network that span across all the nodes in the cluster to which all the pods are connected. Through this networking they are able to communicate each other there are many solutions available to deploy such a network.
* In this case web application deployed on the first node and a database application deployed on the second. The web app can reach the database simply by using IP of the pod.
* But there is no guarentee that IP of the database pod will always remain the same. The better way to access the web application is by using a service. A service is used to expose the database application across the cluster.
* The web application now can access the database using the name of the service DB, the service also get an ip address assiged to it. Whenever a pod tries to reach the service using its IP or name it forwards the traffic to the backend pod ie database.
* The service can't join the pod network because the service is not an actual thing, it is not a container like a pod, it does not have any interfaces or actively listening process. It is virtual component that onluy lives in the kuberenetes memory.
* Kube-proxy process that runs on each node in kubernetes cluster it looks for new services and every time a new service is created it creates appriate rules on each node to forward traffic to those services to the backend pods to connect the service accessible across the cluster from any nodes.
* One way it does this is with IP table rules in this case it creates an IP table rule on each node on the cluster to farward traffic heading towards the IP of the servicew which is `10.96.0.12` to the IP of the actual pod which is `10.32.0.15`. This is how kube-proxy configure a service.
* To install kube-proxy download binary from kubernetes release pages, extract it and run as a service.
* Kubeadm tool deploys kube-proxy as pods on each node. Infact it is deployed as daemonset so a single pod is always deployed on each node in the cluster.

### Pod:

  <img width="903" height="201" alt="image" src="https://github.com/user-attachments/assets/f76b7d5b-8093-4e30-9b10-49e12c7cbc6f" />

* Kubernetes does not deploy containers directly on the worker nodes. Containers are encapsulated into a kubernetes object known as pod.
* A pod is a single instance of an application. Pod is a smallest object that can be created in kubernetes.
* A single node kubernetes cluster with a single instance of our application run on a single docker container encapsulated in a pod.
* When the number of users accessing the application increases and need to scale the application we need to add addittion instances of the application to share the load.
* To bringup new additional instances we create all together with new instance of the same application. We will deploy additional pods on a new node on the cluster to expand the capacity.
* Pods usually have one to one relationship with container running in the application to scale up we create new pods and to scale down we delete the pods.
* We don't add additional containers to an existing pod to scale the application.
* A single pod can have multiple containers of different kind, if we need to scale the application we need to create additional pods but in some cases we may need to have a helper container that might be doing supporting task for the main application. Succh as processing user entered data, processing file uploaded by the user etc.
* If we want have helper containers along side the main container you can have both of these containers are part of the same pod.
* When a new application container is created helper is also created when it dies helper also dies. Since they are part of the same pod.
* Two contaners can directly communicate each other by refering local host since they share the same network space and can share the same storage aswell.
*  When we deploy an application with `docker run python-ap` on docker host. The app is running fine and users are able to access it. When the load increases we will deploy more instances by running the command multiple times. This works fine.
*  After futher develop of the application and architectural changes it get complex now we have a helper container that helps the main/web application by processing or fetching the data from else where. These helper containers maintain one to one relationship with our application container that need to be communicate with the application container directly to access data from those containers.
*  For this we need to maintan a map of what app and helper containers are connected to each other and we need to establish network connectivity between these container our self using links and custom networks.
*  We need to create sharable volumes and shared among the containers, we need to maintain a map of that aswell.
*  We need to monitor the state of the application container when it died manually kill the helper container. When a new container is deployed we need to deploy helper container aswell.
*  With pods kuberentes does all of this for us automatically we just neef to define what container pods consists of and containers in the pods by default access to the same storage and network namespace and samefit as in they will be created together and destroyed together.
* Even our application didn't happened to be complex and we couls live with a single container kubernetes still requires you to create pods but this is good in the long run as your application is now equipped for architectural changes and scale in the future.
* However multi container pod is a rare use case.
#### Deploy a pod:

  <img width="879" height="217" alt="image" src="https://github.com/user-attachments/assets/1a9c855b-8be7-4401-bdb7-ef6ec1e7f832" />

* `kubectl run nginx` this command deploys docker container by creating a pod. It first a pod automatically and deploys an instance of nginx docker image.
* The image is pulled from `docker hub` public repository, where latest images of the various images ares stored.
* `kubectl run nginx --image nginx` will pull nginx image and creates a pod.
* The pod will be in container creating state and changes to running state. In the current state we have not made nginx server accessible to external users. We can access internally from the node.

      kubectl run nginx --image nginx
      kubectl get pods                --> list of pods in the cluster
 
### Pods with YAML:

 <img width="924" height="198" alt="image" src="https://github.com/user-attachments/assets/aeb4796e-0788-47f1-9e84-6ccf3ba51873" />

* Create a pod with YAML. Run `kubectl create -f pod-definiation.yml`
* metadata provide object type.
*  Labels are helpul to identify and group the pods in hundereds of pods. It will be difficlut to identify once they are deploy ex front-end and back-end.  We can labels these pods to filter these pods.
* Under metadata we can specify only `name` and `labels` but we can have any kind of key and value pair. 

* pod-definiation.yml
  
      apiVersion: v1.
      kind: Pod
      metadata:
        name: myapp-pod
        labels:
          app: myapp
      spec: 
        containers: 
          - name: myapp-container
            image: nginx:latest
            ports:
              - containerPort: 80

* Commands for pods:
  
        kubectl get pods                  # list of pods
        kubectl describe pod <pod-name>   # Provide detailed info of the pod
  
### Replication Controller:

 <img width="591" height="215" alt="image" src="https://github.com/user-attachments/assets/0b7d0d3d-3619-4d60-a88e-5be0b0a79909" />

* Controllers are the processes that monitors kubernetes objects and respons accordinlgly.
* Lets assume a single pod is running our application due to some reason the application crashes and pod fails.
* Replication controller helps us to run multiple instances of a single pod in a kuberentes cluster thus proving high availability.
* Even if we have a single pod replication controller can help by automatically bringing up new pod when the existing pod fails thus replicatiion controller ensure that the required number of pods are always running all times even it is one or hundred.
* Replication controllers are used to create multiple pods to share the load across them. When the numebr of user increases we deploy additional pods in the other nodes to balance the load across the pods in the cluster.
* Replication conteroller span across the multiple nodes  in the cluster it helps us to balance the load across multiple pods on different nodes aswell as to scale our application when demand increases.
* `ReplicaSet` and `ReplicationController` both have the same purpose but both are not same. ReplicationController is older technology that is being replaced by replicaset.
#### Replication Controller:

   <img width="906" height="267" alt="image" src="https://github.com/user-attachments/assets/9369cb4c-ff10-4f73-b93d-8f59aabc4274" />

* We create `template` section under the `spec` to provide a pod template used by the replicationController to create replicas.
* Pod template is the same as pod definition file except apiVersion and kind sections.
* Run `kubectl create -f rc-definition.yml`, which creates a replication controller. It first create pods using pod definition template as many as required.
* We can see the desired number of replicas/pods the current number of repliacas and how many of them are ready.
  
      kubectl get replicationcontroller   --> to view replication controller along with desired and current and ready state pod and age of pods.
      kubectl get pods  --> to view pods created by replication controller which starts with rc name.
  
#### ReplicaSet: 
*  It is similar to replication controller. Replicaset will have `selector` section helps to identify what pods falls under it.
* Replicaset need a selector definition, which helps repliset to identify what pods falls under it, we need to specify what pods falls under it even through we have provided contents of the pod in the template it self.
* It is because reoplicaset also manage the pod which are not created as part of repliacaset. Ie there are pos which are created before replicaset creation that match the labels in the selector, replicaset also takecare of those pods into consideration when creating replicas.
* Incase of replicaset a user input is required for this property and it has to be written in the form of `matchLabels`. It matches the labels specified under the labeles in the pod. The replicaset selector also probvides many other options for matching labels that don't available in replicationController.
  
      kubectl create -f rs-definiation.yml
      kubectl get replicaset

#### Lables and Selectors:

  <img width="797" height="221" alt="image" src="https://github.com/user-attachments/assets/f9079b37-1bb8-4e33-bb0d-7514d8fd3dca" />

* Lets assume we deploy a front-end application in 3 pods to create repliacaSet/replicationContoller to ensure that 3 active pods at any time. With repicasdet we can monitor existing pods if we have them created  asit is.
* Incase they were not created replicaset will create for you. The role of replicset to monitor the pods if any of them  were failesvit deploys newone.
* Replicaset is a process that monitor the pods. With the help of `labels` replicaset knows which pods to monitor out of hundreds of other pods running in different clusters.
* We can provide these labels as a filter for replicaset under the selector section we use match labels filter and provides the same label that we use while creating the pod. This ay replicaset knows which pods to monitor.
* If there are existing required number of pods mentioned in the replicaset with the same lebels. Replicaset will not create any new pods Since the requred are already exists with matching labels. Since don't want to create a new pod on deployment.
* Incase one of the pod were failed in the future the replicaset will create new pod to maintain the desired number of pods. 
* For replicaset to crteate a pod the template definition section is required.
#### Scaling replicaset:
  <img width="1045" height="325" alt="image" src="https://github.com/user-attachments/assets/029f2770-0c20-4be5-a57c-4bab6b66ebf3" />


* You can update the replicas in the definition file.
* 
      kubectl create -f replicaset-definition.yml
      kuhectl delete replicaset myapp-replicaset  --> Also deletes underlying  pods as well.
      kubectl replace -f replicaset-dfinition.yml
      kubectl explain replicaset      --> Perovide details of the repliacaset componets.
  

* You can run the below command to scale now need to change in the definition file. We can provide definition file or replicaset name.
  
      kubectl scale --replicas=6 -f repliaset-definition.yml
      kubectl scale --replicas=6 replicaset myapp-replicaset
      
* Even after updating the replicas but the number of replicas in the dedfinition file will not change.
* There are also options available to automatically scaling  the replicas based on the load.

#### Deployments:
     <img width="1112" height="334" alt="image" src="https://github.com/user-attachments/assets/49d53ef0-5d36-4252-a888-ad3a766124bf" />

* Suppose we have a web server in production environment it need not to be one many such instances of web servering running obvious reasons. 
* Whenever newer versions of the application builds are available in docker registry you would like to upgrade your Docker instances seemlessly.
* However when you upgrade the instances we don't want upgrade all of them once as we did  this may impact users access the application.
* So we might want them upgrade them one after the other. This kind of update is knowns as rolling updates.
* Suppose one of the update we perform resulted an unexpected error and wanted to undo the recent changes. You would like to be able to rollback the changes that were recently carriedout.
* If we want to make multiple changes in you environment such as upgrading the web server version=, scaling your environment and modifying the resource allocations, etc.
* We don't want to apply each change immediately after the command is run instead you would like to apply a pause to your eniovironment, make the changes and resume so that all the changes are rolledout together.
* All of these capabilities available with kubernetes deployments.
* Pod deploys a single instance of an application such as webapp. Each container is encapsulated in pods, such pods are deployed using repliacationController/replicaSet.  
* Deployment is kubernetes object that comes higher in the hierarchy. It provides the capability to upgrade the underlying instances seemlessly using rolling updates, undo changes and pause and resume changes as required.
* Pods and replicasets are created by default once deployment is created.
  
      kubectl create -f deployment-definition.yml
      kubectl get deployments
      kubectl get rs
      kubectl get all   --> we can see all objects created in kuberntes 

#### CKA tips:
* Reference link: `https://kubernetes.io/docs/reference/kubectl/conventions/`
  
* Create an NGINX pod
  
      kubectl run ngnx --image=nginx
* Generate POD Manifest YAML file (-o ymal). Don't create it(--dry-run)
  
      kubectl run nginx --image=nginx --dry-run=client -o yaml
* Create a deployment
  
      kubectl create deployment --image=nginx nginx
* Generate Deployment YAML file (-o yaml). Don't create it (--dry-run)
  
      kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

#### Services:
 
  <img width="1114" height="204" alt="image" src="https://github.com/user-attachments/assets/921adc6b-95ff-4545-b71e-2e445f18e8e8" />

* Services enables communication between various components witin and outside of the applications or users. Ex our application has groups of pods running various sections such as a group serving for frontend load to users another group of users running for backend processes.
* Thirs group connecting to external data source. The services that enables connectivity betweemn these groups of pods.
* Services enable the frontend application made available to end users, it helps communicaton between backend and frontend pods and helps in establishing connectivity to external data source.
* THe services enable loose coupling between micro services in our application.
* External communication we deployed our pod having a web application running on it we as an external user can access the web page. In the existing kubernetes setup the node has an IP address 192.168.1.2 and my laptop is also on the same network aswell. So 192.168.1.10 and internal pod network with 10.244.0.0 and the POD has an IP 10.244.0.2. Clearly i can't ping or access the pod at 10.244.0.2 as it is in a separate network.
* The options available to view the web page is first if we were ssh to kubernetes node at 192.168.1.2 from the node we would be access able to acess the pods web page by doing curl  or if the node has GUI we fireup the browser and we see the web page in the browser following the address http://10.244.0.2 but this is from inside the ku* bernetes node. 
* To access the web server from my own laptop witout havinbg to ssh into the node by simply accessing the IP of kubernetes node.
* To achieve this we need to have kubernetes service. A service is an object just like pods, replicasets, deployments one of it's usecase is to listen to a port on the node and forward the request on that port to a port running the web application this type of service is known as NodePort service because the service listens to a particular port on a node and forward the request to pods.
#### Service types:

  <img width="441" height="211" alt="image" src="https://github.com/user-attachments/assets/55634f25-3398-454c-b8d3-e3fa17a401fb" />

* *NodePort*: This service makes internal pod accessible on a port on the node 
* *Clusterip*: In this case the service creates a virtual IP inside the cluster to enale communication between different services such as a set of front-end servers to a set of  backend servers.
* *LoadBalancer*: Where it provisions a loadbalancer for our application in supported cloud providers, which helps to distribute theload across different web servers in your frontend tier.
#### Service - NodePort:
   
  <img width="1177" height="206" alt="image" src="https://github.com/user-attachments/assets/de46c940-4414-4a50-b629-034a034e1629" />

* A service can help us by mapping a port on the node to a port on the pod.
* In this service there are 3 ports are involved the port the pod where the actual web server is runnig is 80 and it is reffered to as `target port` because that is where the request forwards to.
* The second port is the port on the service it self it is simply reffered to as `Port`, these terms are from the view points.
* The service is infact a virtaul server inside the node, inside the cluster it has it has it's own IP address and this IP address is called Cluster IP of the service.
* Finally the port on the node it self which we usedto acess the web server externally is known as NodePort as it is set to 30008 in the snippet. That is becasue node port is only be in a valida ranage which by default from 30000 to 32767.
* The only mandatory field is port it we don't provide targetport it is assumed to be same as port and if we don't provide a nodeport a free port in a valid range of 30000 to 32767 is auto allocated.
* We didn't define any thing in the definition file that connect the service to the pod, we have simply specifuied the target port but we didn't mention the target port on which pod.
* There could be hundreds of pods with web services running on port 80 for this we will use spec section.
* We can have multiple such port mappings within a single service
  
      kubectl create -f service-definition.yml
      kubectl get services
      curl http://192.168.1.2:30008   --> to access the web browser
  
  <img width="1191" height="239" alt="image" src="https://github.com/user-attachments/assets/83366310-fbca-4386-9f82-d5d1fb8e77fa" />

* In case of production enviroment there would be multiple instances of web application runnning for high availability and loadbalancing puroposes. In this case multiple similar pods running our web application. They all have the same labels with key:app and set to a value of myapp, same leble is used as a selector during the creation of the service. 
* When a service is created it looks for a matching pod and a label and finds 3 of them, the service then automatically select all the 3 pods as endpoinst to forward the exteranl request coming from the user.
* We don't need to do any additional configuration to make this happen. The algarithem used to balance the load across the 3 different pods it uses a random algorithem thus the service acts as a built in loadbalancer to distribute the load across different pods. 
* When the pods are distributed acorss multiple nodes in this case we have web application on pods on separate nodes on the cluster. Wheb we create a service without us having to do any additional configuration kuberentes automaticakky creates a service that spans acrosss all the nodes in the cluster and maps the target ports to the same nodeport on all the nodes in the cluster.
* This way we can acess the application using IP of any node in the cluster in the same port number which in this case is 30008 using ip of any of these nodes we are trying to the same port and the sane port is made avaialble on the all nodes of the cluster.
* In any case whether in a single pod on a single node, multiple pods on a single node or multiple pods on multiple nodes the service is created exactly the same without you having to do any additional steps during service creation.
* When the pods are remogved or added the service is automatically updated making it highly flexible and adaptive. Once created we won't typically ahve to make any additional configuration changes.
#### ClusterIP:

 <img width="1208" height="216" alt="image" src="https://github.com/user-attachments/assets/9be2bf8f-4481-4994-867d-a4cb60333d13" />

* A full stack web application typically has differnt kinds of pods hosting different pods of an application. You may have a number of pods runnning on a frontend web server and another set of servers running in the backend server and a set ot pods runnig a key-value store like redis and another set of pods may be running persistent database like mysql.
* The web frontend server need to communicate with the backend servers and backend servers need to communicate with data bases aswell as redis servicesetc.
* The rugut way to establish connectivity between these services are tiers of the application.
* The pods all have an IP address assigned to them as we can see in the image but as we know these IPs are not static these pods can go down any time and new pods are created all the time so we can't rely on this IP addresses for internal communication between the application and also what if the frontend pod at 10.244.0.3 need to connect with backend service which of these 3 it goes to and who makes that decision kubernetes service can help us to group the pods to gther and provide a single interface to access the pods in agroup.
* Ex a service created for backend pods will help to group all the backend pods together and provide a single interface for other pods to acess the service.
* The request is forwarded to one of the pods under the service randomly, similarly create additional services for redis and allow backend pods to allow the redis systems through the service.
* This enales us to easily and effectively deploy micro services based application on kubernetes cluster each layer can scale or move as required without impacting communication between various services.
* Each service gets an IP and a name assigned to it inside the cluster and that is the name that should be used the other pods to access the service. This type of service is known as ClusterIP service.
* ClusterIP is default service type even if we don't specify in the definition file it will conside ClusterIP service.
* Target poort is the port on which backend is exposed here it is 80 and the port where service exposed which is 80 aswell.
* To link the service to a set of pods we use selector we will refer to the pod definition and copy the labels from it under selector.
* The service can be accessed by the other pods using clusterIP for the service name.

       kubectl create -f clusterip-definition.yml
       kubectl get services. 
### Service:
  <img width="777" height="213" alt="image" src="https://github.com/user-attachments/assets/ecd0cb13-37c6-4e4d-a64a-a4e1cfeddbce" />

* Services enable communication between the groups of pods, services running on the front end pods, enable communication to the end user, frontend and backend communication and helps to establish connectity to external database source thus services enable loose coupling between micro services in our application.
* We deploy our pod havuing web application runnning it we being the external user we can access the web page, the existing setup, kubernetes node has an IP addess 192.168.1.2 and the laaptop is on the same network too with IP of 192.168.1.10.
* Internal pod ntwork is with 10.244.0.2 and pod with 10.244.0.2, we can't directly ping from the external user, we can ssh this from external user to kuberenetes node at `curl http://10.244.0.2`.
* From the node we will be access to the webpage by using curl or if the node has GUI fireup the browser and see the webpage but this is from inside the kubernetes node.
* But we need to access the webserver from own laptop without ssh to the node by simply accessinhg kubernetes node. 
* Kubernetes services is an object just like pods, replicasets, deployments one of its use case is to listen to a port on the node and forward the request on that port to  a port ruuning on the web application this type of service is `node-port` service because the service listens to a port on the node and forard reqest to the pods.
#### Service types:
 * ** NodePort**: where the service makes an internal port accessible on a port on the node.
 * **ClusterIP**: In this case service creates an virtual IP inside the cluster and enable communication between differnt services such as a set of frontend servers to a set of backend servers.
 * **LoadBalancer**: Where it provisions a load balancer for our application in supported cloud provider. Ex: To distribute the load across differnt web servers in your frontend tier.

   <img width="406" height="188" alt="image" src="https://github.com/user-attachments/assets/d5c394d6-ddb1-4316-85ec-f14ac9af7eff" />

* **Service-NodePort**:
 <img width="781" height="419" alt="image" src="https://github.com/user-attachments/assets/917334a8-c2a5-40dc-86e1-945213a6d68c" />

  * A service can help us by mapping a port on the node to a port on the pod, there are 3 ports involved the port on the pod where the actual web server is running is 80 and it is reffered to as the target port becasuse that is where the service forwards the request to.
  * The second port is the port on the service it self, referref to as the port, these terms are from the view point of the service.
  * The service is infact like a virtual server inside the node, inside the cluster it has its own IP address and that IP address is called Cluster IP of the service.
  * The port on the node itself which we use to access webserver externally and that is known as node port, set to 3008 here. 
  * The node ports can only be in a valid range which by default from `30000-32767`.
  * Out of targetPort, NodePort and port only mandatiry field is only port, if we don't provide target port it is assumes to be same as port.
  * If we don't provide NodePort it will auto assign a free port in the valid range of 30000-32767 range and ports are an array. 
  * We can have multiple such port mapping within a single service.
  * There could be hundreds of other pods with web services running on port 80 we will use labels and selectors to link these together. We will add selector section to brindhg the label to service definition file this links the pod.
    
        kubectl create -f service-definition.yml
        kubectl get services
        curl http://192.168.1.2:30008 

    <img width="1061" height="210" alt="image" src="https://github.com/user-attachments/assets/cb8a0496-0936-4e08-b288-c1ce61db51a2" />

* In the prooduction environment you have multiple instance of yor web application is runnning for hugh availability and load balancing purposes in this case When we have multiple similar pods running on our web application they all have the same labels the same label is used as a selector during the creation of the service.
* When a service is created it looks for a matching pod with the label and finds the pods of the web application.
* The service then acutomatically select all the pods matching with label as end points to forward the external request comingh from the user.
* We don't need to do any additional configuration to make this happen the algorithem used to balance the load across three different pods, it is Random algorithem thus the service acts as built in loadbalacer to distribute the load across diffirent pods. SessionAffinity :yes
* When the pods are distribute across multiple nodes in this case we have web application on pods on nodes on a separate cluster when we create a service without us having to do any additional configuration kubernetes automatically creates a service that automatically spans across all the nodes in the cluster and maps the taraget port to the same NodePort on all the nodes in the cluster this way we can access the application by using ip of any node in the cluster and using the same port number (30008 here).
* We can access the web appication using any of the IP of these nodes and trying to curl the same port, the same port is available on all available on all the nodes as part of the cluster.
* In any case whether it can be a single pod on a single node and multiple pods on a single node or multiple pods on multiple nodes the service created will be the same witout you having to do any additional steps during the service creation.
* When the pods are removed or added the service will be auto updated making it highly flexible and adaptive. Once created we won't make any additional configuration changes.
   
*** ClusterIP**:

   <img width="898" height="154" alt="image" src="https://github.com/user-attachments/assets/ac5ae374-c920-4bc0-b197-89ef39fbd990" />

* A full stack web application typically has differnt kinds of pods hosting differnt parts of an application. 
* We may have many number of apps running on frontend web server, another setof servers running on backend servers and a set of servers ruuning for a key-value store like redis and another setof pods running a persistent database like mySql.
* The webfrontend server need to communicate to the backend servers and the backend servers neeed to communicate with database aswell as redis services, etc.     
* The right way to establish connectivity between the services or tiers of my application, the pods have an ip addresses assigned to them but these IPs are not static these pods can go down any time and new pods are created all the time.
* So we can't reply on these IP addresses for internal communication between the application, and if the frontend pod 10.222.0.2 want to connect to backend service a kubernetes service helps to make this decision.
* A kubernetes service can help us group the pods together and provide a single interface to access the pods in a group. For example a service created for the backend pod will help to group all the backend pods together and provide a single interface for other pods together and provide a single interface for other pods to access the service.
* The requess are forwarded to one of the pods under the service. The requests are forwarded to one of the pods under the service randomly.
* Similarly create additional services for Reids and allow the backend pods to access the Redis systems through services.
* This enables us to easily and effectively deploy a microservices based applicatoion on a kubernetes cluster. Each layer can now scale or move as required without impacting communication between the various service.
* Each service gets an IP and name assigned to it inside the cluster and that is the name that should be used by other pods to access the service. This type of service is called as ClusterIP. 
* To create such a service always use definition file and in the spec section update type to `ClusterIP` but by default sevice type is ClusterIP even we didn't specify it. Target port is the porty where backend is exposed here it is 80 and port is where the service is exposed and update selector.
  
        kubectl create -f service-definiation.yml
        kubectl get services
        kubectl get svc -A        # all namespaces
        kubectl describe service  # provide detailed info of the services, we can check endpoints as well

        alias k='kubectl'        # Set alias k for kubectl
         
* The service can be accessed by other pods using clusterIP or the service name.
* End points are the pods that serivice is idenified th traffic going to based on selector specified on the service and lables on the pods.

* **Certification tips:**
 * `--dry-run`: By default as soon as the command is run the resource will be created.
 * `--dry-run=client` this will not create resource, instead tell you whether the resource can be created and if yoour command is right.
 * `-o yaml`: This will output the resource definition in YAML format on screen.

        kubectl run nginx --image=nginx                                 # crete a pod
        kubectl run nginx --image=nginx --dry-run=client -o yaml        # It generate POD manifest file in YMAL --dry-run (Don't create)
        
        kubectl create deployment --image=nginx nginx                   # Create a deployment
        kubectl create deployment --image=nginx nginx --dry-run=client -o yaml # Generate deployment YMAL file ( -o yaml). Don't create it (--dry-run)
        
        kubectl create deployment nginx --image=nginx --replicas=4     # Create deployment with 4 replicas
        kubectl scale deployment nginx --replicas=4                    # Scale deployment replicas
        
        kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yml     # Save the deployment YAML definition
        
        kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml                # redis-service with ClusterIP to expose pod on 6379, uses pod labels as selectors
        
        kubectl create service clusterip redis redis --tcp=6379:6379 --dry-run=client -o yaml             # This don't use pod labels, assume selectors as app=redis, can't pass in selectors as an option so generate file and modify selector before service creation
        
        kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml       # It don't use the pods lables as selectors

 * The default output formaat of `kubectl` command is human-readable plain-format. Oher formats.
  * `-o jason` output a JSON formatted API object
  * `-o name` Print only the resource name and nothing else
  * `-o wide` Output in a plain-text format with any additional information.\
  * `-o yaml` Output a YAML formatted API object
    
        # output in JSON format:
    
        master $ kubectl create namespace test-123 --dry-run -o json
        {
            "kind": "Namespace",
            "apiVersion": "v1",
            "metadata": {
                "name": "test-123",
                "creationTimestamp": null
            },
            "spec": {},
            "status": {}
        }
        master $

        
        # Output with YAML format:
    
        master $ kubectl create namespace test-123 --dry-run -o yaml
        apiVersion: v1
        kind: Namespace
        metadata:
          creationTimestamp: null
          name: test-123
        spec: {}
        status: {}

        
        # Output with wide(additional details):
    
        master $ kubectl get pods -o wide
        NAME      READY   STATUS    RESTARTS   AGE     IP          NODE     NOMINATED NODE   READINESS GATES
        busybox   1/1     Running   0          3m39s   10.36.0.2   node01          <​none​>         <​none​>  
        ningx     1/1     Running   0          7m32s   10.44.0.1   node03          <​none​>         <​none​> 
        redis     1/1     Running   0          3m59s   10.36.0.1   node01          <​none​>         <​none​> 
        master $
##### Ref docs link:
* https://kubernetes.io/docs/reference/kubectl/overview
* https://kubernetes.io/docs/reference/kubectl/cheatsheet/

#### kubectl explain:
  <img width="904" height="465" alt="image" src="https://github.com/user-attachments/assets/e253fe33-3690-4fb4-8a52-0c067daaf4cf" />
  
  <img width="896" height="272" alt="image" src="https://github.com/user-attachments/assets/1be052f3-509f-4028-879c-1fd36bfbf428" />


* we can learn a lot about the resources and fields witout leaving the terminal.
* `kubectl api-resources` it list all the resources this is specially useful when we can't find on the documentation pages. If you ever forget resource name or short name or you are not sure of the underlying vesion of the API such as `v1` or is it `apps/v1` or if you are not sure exact case of resource name to be used while creating a definition file you can use kubectl api-resource command.
* Once you identify the resource name from the list and you need more information about the resource fileds and types under it and then you can run `kubectl explain <resource_name>` to be exlained, in this case pod list all top level fields in pods being API version, kind, metadata and spec and status and provide type of each field and description, only diplays top level fields.
* T9 get more deeper use `kubectl explain <resource_name>.spec` this lists the subfields within it but this again lists the sub properties of parent property that you have mentioned.
* But it is still not comprehensive so to list all the fields in the way that you would put in a YML file used a recxursive flag, it now outputs the entire comprehensive list of fields that's available on that resource.

       kubectl run redis1 --image redis:alpine --labels tier=front-end
       kubectl run --help
       kubectl create service clusterip --help
       kubectl expose --help
       kubectl describe svc <service_name>

#### Docker Images:
  <img width="772" height="609" alt="image" src="https://github.com/user-attachments/assets/93590adf-54d1-41c2-807c-5bc5b178abf0" />

* We need to create our own image because we either can't a component or a service you want to use as part of your application on DokcerHub already or you and team decided that the application you are developing will be dockerized for ease of shipping and deployment.
* A simple web application built using python Flask framework first you need to understand what you are containerizing and what application we arecreating an image for and how the application is built.
* If you want to deploy an application manually write down the steps required in right order.
  * Start with an OS like ubuntu
  * update the source repositories using apt command
  * install dependencies using apt command
  * install python dependencies using pip command
  * Copy over the source code of the application to a location like `/opt`
  * Finally run the web server using flask command.
* Create a docker image
  * Create a docker file named `Dockerfile` and write down the instructions for settingup your application in it, such as installing dependencies, where to copy the source code from and to and what the entry point of the application is, etc.
  * Once done build your image with docker build command and specify the docker file as input as well as a tag name for that image.
  * This will create an image locally on your system to make it available on the public docker hub registry run docker push command and specify the name of the image.
  * The name of the image is your dokcerhub account name followed by image name which is my custom app.
* Docker file is a text file written in a specific format that docker can understand it's in instruction and argument format.
* Everything in the left Capitals is an instruction in this case `FROM`, `RUN`, `COPY` and `ENTRYPOINT` are all instructions.
* Each of these instruct docker to perform a specific action while creating the image.
* Every thing on right is an argument to those instructions. The first line from Ubuntu defines What base OS should be for this container.
* Every docker images must be based off of another image either an OS or another image that was created before based on an OS.
* All dockerfiles must start with a from instruction, run isntruction instruct docker to run a particular command on those base images.
* Docker runs apt get update command to fetch the updated packages and install dependencies on the image. 
* Copy instruction copies files from the local system onto the docker image in this case source code of our application is in the current folder and it is copying to the location /opt source code inside the docker image.
* Entrypoint allows us to specify a command that will be run when the image is run as a container.
* When docker builds the images it builds this in a leyered architecture, each line of the instruction creates a new layer in the docker image with just changes from the previous layer.
* ex: First layer is base ubuntu OS, followed by second instruction that creates a second layer which installs all apt packages. Third instruction creates a third layer with Python packages , followed by fourth layer that copies the source code over and the final layer that updates the entrypoint of the image.
* Since each layer only stores the chnages from the previous layer, it is reflected in the size .
* We can check size of each layer by running `docker history <image_name>`
* When we run docker build command you could see various steps involved and the result of each task.
* All the layers build are cast so the layered architecture helps you restart docker build from that particualr step in case fails.
* Or if you were to add new steps in the building process you won't have to sart all over again.
* All the layers build are cached by docker incase a particular step  was to fail.
* Ex Step3 failed and you were to fix the issue and rerun Docker build it reuse the previous layer from cache and continue to build the remaining layers. 
* The same is true if you add adding additional steps in the Dockerfile, this way rebuilding image is faster, and you don't have to wait for docker to rtebuild the entire image each time.
* This is helpful especially when you update the source code of your application as it may change more frequently.
#### Commands and Arguments in Docker:
* When we run `docker run ubuntu` it run an instance of ubuntu image and exits immediately.
* Unlike virtual machine containers are not meant host operating system, containes are meant to run a specific task or process such as to hosty an instance of a web server or app server or database server or simply to carrout some kind of compution or analysis.

      docker run ubuntu
      docker ps        # lists running containers
      docker ps -a     # lists all container including stopped
  
* Once the task is completes the containers exists, the container lives as long as the process inside it is alive.
* If the web server inside the container is docked or crashes the container exits.
* CMD instruction defines the program that willbe run within the docker container when is starts. For Nginx image, it is Nginx command & for MySQl image, it is MySQLD command.
* In the earlier we used to run a plain ubuntu operating system 
* Docker file has default command as bash but bash is not really a process like web server or database server. It is a shell that input from a terminal if it can't find a terminal it exits.
* When we run a ubuntu container, docker created a container from ubuntu image and launch bash program. By default docker does not attach a terminal to a container when it run.
* So bash program can't find the terminal and so it exits. Since the process that was started when the container was created finished. The container exits as well.
* To start the container we can specify diffrent command is by appending a command to docker run command and this wayover rides the default command specified within the image in this case `sleep 5` command as an option.
* `docker run ubuntu [Command]` `docker run ubuntu sleep 5`. 
* This way when a container starts it runs sleep program, wait for 5 seconds and then exits.
* If we want the image always run sleep command when it starts it will then creates its own image from the base Ubuntu image and specify new command.
* There are different ways of specifying commands either the command is as is in shell form or in a json array format. When we specify in Json format the first element in the array is executable here sleep program.
* Don't specify command and parameter together, the command and the parameter should be separate elements in the list.

 <img width="732" height="599" alt="image" src="https://github.com/user-attachments/assets/cfc1a230-2c4f-4e7b-955f-43a2e239cc1d" />


 <img width="771" height="256" alt="image" src="https://github.com/user-attachments/assets/79eb7ce4-d253-4c62-87f1-5818b631601e" />

* It always sleeps for 5 seconds and exits.
* If we want to chage the default sleep time of 5 sec to another value, one option is use docker run command with new command appended to it, here sleep 10.
* We want to pass the number of seconds the container should sleep and the sleep command should be invoked automatically that is where ENTRYPOINT instruction comes into play.
*  ENTRYPOINT defines the executable command, CMD provides default args as you can specify the program that will run as you specify when the container starts.
*  Whatever you specify in the command line in this case 10 will be appended to entrypoint.
   
  <img width="384" height="210" alt="image" src="https://github.com/user-attachments/assets/21829f97-7b46-4af5-b8e7-e2c655a032dd" />

* So that command that runs when the container starts is sleep 10. This is the difference between these two.
* Incase of CMD instruction command line parameters pass will get replace the entirely where as incase of ENTRYPOINT command line parameters will get appended.
* Now in the second case if we run ubuntu-sleeper command `docker run ubunu-sleeper` without appending the number of seconds.
* In this case the commandlins startup will just sleep and will get the error the apparand is missing.
* We can configure a default value if one was not specified this is where we use both ENTRYPOINT aswell as  CMD instruction. In this case command instruction will be appended to ENTRYPOINT instruction at the startup the command would be sleep 5 if we didn't specify any parameters in the command line.
* If we did it will over ride the command instruction to happen this we should always specify the ENTRPOINT and CMD instruction in JSON format.
* If you really want to modify the ENTRYPOINT during the runtime say from sleep to an imaginary sleep2.0 command. In this case you can over ride it by using ENTRYPOINT option in the docker run command.
* The final command at the startup will then be sleep2.0 10
  

















