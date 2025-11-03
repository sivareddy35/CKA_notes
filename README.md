# CKA_notes

  <img width="487" height="263" alt="image" src="https://github.com/user-attachments/assets/2b072250-658e-4662-927c-1c52bc0eda82" />

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

### etcd
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

### etcd in Kuberntes
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
  
### Kube - apiserver

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

### Kube - Controller - Manager
  
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










