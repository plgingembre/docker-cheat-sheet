# Docker

## Building a docker image for the first time

## Create a directory and move into this new directory (kind of project directory)
mkdir redis-server

## Create a Dockerfile
The Dockerfile (capital D and no extension) is a set of instructions to build an docker image
Edit this file with any text editor

Content of the Dockerfile:
```
# Use an existing docker image as a base
FROM alpine

# Download and install dependency
RUN apk add --update redis

# Tell the image what to do when it starts as a container
CMD ["redis-server"]
```

## Build the container from the image
`docker build .`

```
plgingembre@SV-LT-1179:~/docker/redis-image$ docker build .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM alpine
latest: Pulling from library/alpine
e7c96db7181b: Pull complete
Digest: sha256:769fddc7cc2f0a1c35abb2f91432e8beecf83916c421420e6a6da9f8975464b6
Status: Downloaded newer image for alpine:latest
 ---> 055936d39205
Step 2/3 : RUN apk add --update redis
 ---> Running in 077190e6a4d5 <<< Create a temporary container running the base of the primary step
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
(1/1) Installing redis (4.0.12-r0)
Executing redis-4.0.12-r0.pre-install
Executing redis-4.0.12-r0.post-install
Executing busybox-1.29.3-r10.trigger
OK: 7 MiB in 15 packages
Removing intermediate container 077190e6a4d5 <<< FS snapshot of this temporary container. Container stopped. ID of the container:
 ---> 07fa36034e67
Step 3/3 : CMD ["redis-server"]
 ---> Running in 179a53af6464 <<< Temporary Container running the base image and updated in the second step
Removing intermediate container 179a53af6464 <<< Temporary Container stopped and snapshot taken with the following ID:
 ---> 6b929386a91e
Successfully built 6b929386a91e
plgingembre@SV-LT-1179:~/docker/redis-image$
```
## Running the docker container
`docker run 6b929386a91e`

```
plgingembre@SV-LT-1179:~/docker/redis-image$ docker run 6b929386a91e
1:C 01 Jun 18:04:16.118 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 01 Jun 18:04:16.118 # Redis version=4.0.12, bits=64, commit=1be97168, modified=0, pid=1, just started
1:C 01 Jun 18:04:16.118 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1:M 01 Jun 18:04:16.120 * Running mode=standalone, port=6379.
1:M 01 Jun 18:04:16.121 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:M 01 Jun 18:04:16.121 # Server initialized
1:M 01 Jun 18:04:16.122 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1:M 01 Jun 18:04:16.122 * Ready to accept connections
^C1:signal-handler (1559412272) Received SIGINT scheduling shutdown...
1:M 01 Jun 18:04:32.838 # User requested shutdown...
1:M 01 Jun 18:04:32.838 * Saving the final RDB snapshot before exiting.
1:M 01 Jun 18:04:32.841 * DB saved on disk
1:M 01 Jun 18:04:32.842 # Redis is now ready to exit, bye bye...
plgingembre@SV-LT-1179:~/docker/redis-image$
```

## Show docker images on the local host
`docker image ls`

## Create an image with a tag name
`docker build -t your_docker_id/repo_or_project_name[:version] . ` (version is often 'latest', '.' is the directory to use for the build)

## Remove all stopped containers, networks not used, images without a containers using it, build cache
`docker system prune -a`


## Connect docker containers together

docker cli is an option, but nobody uses it.
docker compose is a docker option to automate and simplify networking orchestration for docker containers (uses docker cli underneath, but removes complexity)
docker compose uses a yaml file to give instructions to docker compose cli

a 'service' in docker language refers usually to a 'container' or 'container type'

example of a simple docker-compose.yml:
version: '3'
services: 			// services or containers we want to run
  redis-server:
    image: 'redis'	// pulls the redis container from docker hub
  node-app:
    build: .		// build the image from local directory
    ports:
      - "4001:8081"	// ports to be used to forward traffic to our containers

when using docker compose, the new "translated" commands are:
- docker run myimage ==> docker-compose up
- docker build -t login/project:version . + docker run myimage ==> docker-compose up --build

stop all the containers at the same time:
docker-compose down

different options exists to restart a crashed container ("no" (default option, quote mandatory), always, on-failure (status code different from 0) & unless-stopped)

get the status of a docker compose
docker-compose ps // local to your current directory (and read the docker-compose.yml file)



## Using docker volumes

a docker volume is a reference point to the local filesystem (mapping of a folder inside the container to a folder outside)
docker volumes is quite a pain in the CLI

Using docker volumes in the CLI:
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <docker_id>
- the first -v is to bookmark to the "node_modules" folder
- the second has a ":", which means that we point to a reference local to the host (outside of the container)

(side note: when attaching to the console of a docker container, using docker attach, we only get a handle on the primary process, not the others)



# Kubernetes

## Intro

k8s: the '8' is for the 8 letters in between the 'k' and the 's'.


## Objects in k8s

Object types are things that we are can create inside a k8s cluster that have very specific purposes to make our app work the way we expect. Examples:
- StatefulSet
- ReplicaController
- Pod
- Service

This is defined in the 'kind' field of the config files


## apiVersion

It scopes or limits the types of objects that we can specify that we want to create with any given configuration file. Specifying 'apiVersion: v1' gives us access to a predefined set of different of object types. With v1, we can create object types like:
- componentStatus
- configMap
- Endpoints
- Event
- Namespace
- Pod

Another apiVersion is apps/v1, which gives us access to the following object types:
- ControllerRevision
- StatefulSet


## Node

A Node is an indvidual machine that is used by k8s to run some number of different objects (container/pod being one of them).


## Pod

A Pod is an object in k8s and is a grouping of containers with a very common purpose. k8s will NEVER run a naked single container by itself, the Pod is the smallest thing that we can deploy to run a container in k8s. So a container will always be in a Pod in k8s.
The purpose of a Pod is to allow the grouping of containers with a very similar purpose or containers that must absolutely run together in order for an application to run correctly.

Example of pod config file (client-pod.yaml)
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
  labels:
    component: web
spec:
  containers:
    - name: client
      image: stephengrider/multi-client
      ports:
        - containerPort: 3000

In the Pod config file, under 'spec:', we can specify the containers that are running in that Pod.

metadata section:
- name = essentially for logging purpose (in kubectl)
- labels = labels assigned to our Pod and is tightly coupled to the service config file to specify how we connect containers or Pods together

spec section:
- name = tag
- image = container image
- ports = port mapping to the host (not the full story of port mapping in k8s, it is way more complicated than that in k8s... :))


## Service

A Service is an object in k8s and defines the networking inside of a k8s cluster. There are 4 subtypes (or type of service):
- ClusterIP
- NodePort ==> expose a container to the outside world, only used for development, NEVER in production
- LoadBalancer
- Ingress

Example of a service config file (client-node-port.yaml):
apiVersion: v1
kind: Service
metadata:
  name: client-node-port
spec:
  type: NodePort
  ports:
    - port: 3050
      targetPort: 3000
      NodePort: 31515
  selector:
    component: web

Architecture of a NodePort service:

          +---------------------------------------------------------------------------+
          |                                                                           |
          |                                               +---------------------------+
          |                                               |                           |
          +------------+         +------------+           |        +----------------+ |
          |            |         |            |           | +------+                | |
+-------->+   kube-    +-------->+  Service   +------------>+ Port |  Multi-client  | |
          |   proxy    |         | Node Port  |           | | 3000 |   Container    | |
          |            |         |            |           | +------+                | |
          +------------+         +------------+           |        +----------------+ |
          |                                               |                           |
          |                                               |                           |
          |                                               +---------------------------+
          |               Kubernetes Node VM created by Minicube                      |
          +---------------------------------------------------------------------------+

The kube-proxy is the one single window to the outside world. Anytime a request comes to the container, it goes through the kube-proxy. The proxy inspects the request and decides how to route it to the different services or pods that we may have created inside the Node. It can be only one service or pod, or many of them.

In between the pod and service config files, the link is the [label/selector] relationship. When a selector is defined, the Node is gonna look at pods in a cluster whose label is the exact same value and will use this selector to configure the port forwarding to this pod. In our example, we use 'component: web', but that could be any other 'key: value' name for the label/selector, as long as it is the same reference value.

What are all these ports?
In our example, we have:
spec:
  type: NodePort
  ports:				==> an array, could be more than one port
    - port: 3050		==> port than another pod/container could use internally to the cluster (by another pod) to access this pod (component: web)
      targetPort: 3000	==> actual port used by the application inside the pod/container (component: web)
      NodePort: 31515	==> port exposed on the Node mapped to the container/pod's target port (if not specified, a random port between 30000-32767 is gonna be allocated)
  selector:
    component: web


## Kubectl

kubectl is the CLI to use to change the kubernetes cluster configuration

`kubectl apply -f <filename>`	==> change the current cluster configuration using the config file specified. The config is actually applied to the kube-apiserver program running on the master node of the cluster
`kubectl get <object_type>` 	==> check the status of different object types (pods, services)


## Master node

The Master node is just another node (machine or VM)
The Master node is running a program called kube apiserver (in reality composed of 4 running programs)
100% responsible for monitoring the current status of the different nodes inside of the cluster and makes sure they are doing the correct thing
Based on the INTENT (Name, # I need, # I have running), using 'My Responsabilities' in the k8s course, it instructs the nodes to launch the required containers to reach the target specified in the config file.
The master node decides how many instances of the container should be run on each node
The master node is all about INTENT based on the config files (not imperative!)

When the master nodes sends instructions to worker nodes to start a container, each node will:
- contact docker hub and find the image
- download the docker image of the container it was asked to run
- store it on some local image cache
- run as many copies of this container as requested by the master node

The master node is polling all worker nodes for status update

After the deployment (as described in our config files), the master is also continuously polling every worker node and is notified when something changes on a node. If a container dies on one node, it will get notified and will request the node to restart the container or will request the restart on another node if the given node is not able to fulfill the requirement.


## Imperative VS. Declarative

Imperative: list of commands that the admin will have to come up with to do some actions ==> do what I am telling you to do!

Declarative: define the intention of what is supposed to happen (and not how) ==> here is what it should look like, please make it happen

Kubernetes has the option to do both approaches (kubectl, etc.), but it is HIGHLY recommended to use the declarative approach to make things easier

In general, when reading blog posts, you might have people advocating for imperative mode, but the reality is that in production, we should ONLY use declarative mode, and that's what people are doing now.


## Update existing ojects

When updating an object, here is what happens

                               +----------------------------------------------------------+
                               |                                 +----------------------+ |
                               |                                 |   Virtual Machine    | |
                               |                                 |       (Node)         | |
                               |                                 |                      | |
                               |  +---------+     +--------+     | +------------------+ | |
+-------------------------+    |  |         |     |        |     | |       Pod        | | |
|   Updated Config File   +------>+ kubectl +---->+ Master +---->+ | Name: client-pod | | |
+-+---------------------+-+    |  |         |     |        |     | |                  | | |
| |   Name: client-pod  | |    |  +---------+     +--------+     | | +--------------+ | | |
| +---------------------+ |    |                                 | | | multi-client | | | |
| |      Kind: pod      | |    |                                 | | |  container   | | | |
| +---------------------+ |    |                                 | | +--------------+ | | |
| | image: multi-worker | |    |                                 | +------------------+ | |
| +---------------------+ |    |                                 +----------------------+ |
+-------------------------+    +----------------------------------------------------------+

Name and Kind properties are the ones looked at (unique identifier tokens). If there is running service inside the cluster that has the same properties (Name, Kind), it will try to update this object by using the new config file.

To update an object in a declarative way:
1/ change the config file for the pod (image name changed from 'bla/multi-client' to 'bla/multi-worker')
2/ `kubectl apply -f <filename>

To verify the update:
- `kubectl get pods` ==> missing details, we only see the RESTARTS was incremented of +1
- `kubectl describe pods <name>` ==> provide all details and events info related to the pod

But when trying to change just the port on a pod, we get an error like this:
The Pod "client-pod" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)
We can't modify the # of containers in a pod, the pod name or port used by the containers in a pod using this `kubectl apply` command ==> Need another command or object to achieve this kind of operation.


## Deployment

To be able to modify an object like a pod, we need to introduce a new object type called 'Deployment'. A Deployment maintains a set of identical pods, ensuring that they have the correct config and that the right number exists

Quick comparison between Pods and a Deployment:
+----------------------+    +----------------------------+
|         Pod          |    |         Deployment         |
+----------------------+    +----------------------------+
| Runs a single set of |    |  Runs a set of identical   |
|      containers      |    |    pods (one or more)      |
+----------------------+    +----------------------------+
| Good for one-off dev |    | Monitors the state of each |
|       purposes       |    | pod, updating as necessary |
+----------------------+    +----------------------------+
| Rarely used directly |    |        Good for dev        |
|    in production     |    +----------------------------+
+----------------------+    |    Good for production     |
                            +----------------------------+

Deployments behind the scenes uses Pods, so they are very tightly related to one another. The relationship between Deployments and Pods is like so:
+--------------------------------+
|           Deployment           |
+--------------------------------+
|          Pod Template:         |          +------------------+
| +-------------+--------------+ |          |       Pod        |
| | containers: |      1       | |          +------------------+
| +----------------------------+ |          |   Name: client   |
| |    name:    |    client    | +--------->+ +--------------+ |
| +----------------------------+ |          | | multi-worker | |
| |    port:    |     3000     | |          | |  container   | |
| +----------------------------+ |          | ++-----------+-+ |
| |   image:    | multi|worker | |          |  | port 3000 |   |
| +----------------------------+ |          |  +-----------+   |
+--------------------------------+          +------------------+

In this example, every deployment uses a pod template and if there is a need to change a Pod, the Pod template (configuration file) is updated and the deployment will try to either change the running pod or delete it and recreate it with the update configuration (like port changed).

Example of a deployment config file (client-deployment.yaml):
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: web
  template:
    metadata:
      labels:
        component: web
    spec:
      containers:
        - name: client
          image: stephengrider/multi-client
          ports:
            - containerPort: 3000

'replicas' defines the number of Pods we want running this template
'selector' is doing something very similar to what a selector is doing in a service config file (selector/labels mapping)
'template' section is well known, same as what is created for a pod config file

To apply a new deployment:
`kubectl apply -f <config-filename>` ==> <config-filename> can also be the name of a directory, in this case all files within that directory will be applied at once! :)

To check a deployment:
`kubectl get deployments`
`kubectl get pods`

To get more info on a deployment 
`kubectl get deployments -o wide`
`kubectl get pods -o wide`			==> showing the internal IP of the pods inside the cluster (not accessible)


## Delete existing ojects

To delete existing objects, use the following command
`kubectl delete -f <config-filename>` ==> <config-filename> being the file used to create the object, btw, this a VERY IMPERATIVE instruction, yes it is... :)


## Process to update a Deployment's image

To update to a new version of one of our container, do the following:
- update the container image (change code) ==> `docker build ...`
- push to docker hub (private/public repo) ==> `docker push ...`
- get the deployment to recreate our pods with the latest version of our image ==> not an easy thing to do...

Biggest issue for the last step, k8s has no way to understand that there is a new version of the container image (status shows 'unchanged').

Options:
- We could manually delete all pods running the old version of the container, and force the pods to be recreated with the latest version of our image ==> delete pods is definitely NOT a good solution (human error, application would be disrupted, etc.)
- We could also tag built images with a real version number and specify that version in the config file (like 'image: name/client-worker:v2') ==> would be detected as a real change and processed by kubectl ==> adds extra step in the production deployment process (sync'ing the config file and the CLI command ==> too tricky!)
- We could also use an imperative command to update the image version the deployment should use ==> we don't like using imperative commands, it is desync from the idea of suggesting the change to our cluster

Solution 3 is the one selected because it is the less intrusive way to do it (but none of these solutions are good anyway...)

So now, to update to a new version of one of our container, we're adding extra steps:
- update the container image (change code)
- NEW!! tag the image with a version number ==> `docker build -t name/multi-client:v5`
- push to docker hub (private/public repo) ==> `docker push -t name/multi-client:v5`
- get the deployment to use the new image version by using an imperative `kubectl` command 

Imperative command to use to update the image used in our deployment:
`kubectl set image <object_type> / <object_name> <container_name> = <new_image_to_use>

Real life example: `kubectl set image deployment/client-deployment client=stephengrider/multi-client:v5`

To check: `kubectl get pods`

This whole process is really to understand what's going on behind the scenes, but people are not really doing it this way when using AWS or GCP, they use some scripts to update their images and deployments.


## Configure docker client to query docker server running inside Minikube VM

To configure your CURRENT terminal window to use docker server inside the minikube VM, use the following command:
`eval $(minikube docker-env)

The command `minikube docker-env` does the following:
```
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.105:2376"
export DOCKER_CERT_PATH="/Users/pierre-louisgingembre/.minikube/certs"
# Run this command to configure your shell:
# eval $(minikube docker-env) ==> gives us a hint on what to use to get this from the local laptop
```

Why we want to do that? Couple of ideas:
- Use all the same debugging tools/techniques we learned with docker CLI ==> most of these commands are available with a kubectl command
- Manually kill containers to test k8s ability to self-heal
- Delete cached images in the node


## Example of a real app using microservices and k8s
```
                     +-----------------------------------------------------------------------------------------+
                     |               +-----------+------------------+                     +------------------+ |
                     |               |           |    Deployment    |                     |    Deployment    | |
                     |               |           +------------------+                     +------------------+ |
                     |               | ClusterIP | multi-client pod |               +-----+ multi-worker pod | |
                     |          +--->+  Service  +------------------+               |     +------------------+ |
                     |          |    |           | multi-client pod |               |                          |
                     |          |    |           +------------------+               v                          |
                 +---------+    |    |           | multi-client pod |         +-----+-----+------------------+ |
                 |         +----+    +-----------+------------------+         | ClusterIP |    Deployment    | |
Traffic +------->+ Ingress |                                             +--->+  Service  +------------------+ |
                 | Service |                                             |    |           |    Redis pod     | |
                 |         +----+    +-----------+------------------+    |    +-----------+------------------+ |
                 +---------+    |    |           |    Deployment    |    |                                     |
                     |          |    |           +------------------+    |                                     |
                     |          |    | ClusterIP | multi-server pod +----+    +-----------+------------------+ |
                     |          +--->+  Service  +------------------+    |    | ClusterIP |    Deployment    | |
                     |               |           | multi-server pod +----+--->+  Service  +------------------+ |
                     |               |           +------------------+    |    |           |   Postgres pod   | |
                     |               |           | multi-server pod +----+    +-----------+--------+---------+ |
                     |               +-----------+------------------+                              |           |
                     |                                                                             v           |
                     |                                                                    +--------+---------+ |
                     |                                                                    |   Postgres PVC   | |
                     |                                                                    +------------------+ |
                     +-----------------------------------------------------------------------------------------+
```
All boxes named 'xxx Service' or 'Deployment' or 'xxx PVC' will need to have a separate config file (so 11 in total)


## ClusterIP service

A ClusterIP is a way to expose a set of pods to OTHER OBJECTS IN THE CLUSTER (and not externally or from outside of the cluster)

A ClusterIP is typically used when we need to provide connectivity within the cluster, but not from outside

Example of a ClusterIP service config:
apiVersion: v1
kind: Service
metadata:
  name: client-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: web
  ports:
    - port: 3000
      targetPort: 3000


## Deployment and Service config files

If we have too many services and deployments, we can definitely combine these two config files together.

Example of a combined config file (server-config.yaml):
apiVersion: apps/v1
kind: Deployment
metadata:
  name: server-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      component: server
  template:
    metadata:
      labels:
        component: server
    spec:
      containers:
        - name: server
          image: stephengrider/multi-server
          ports:
            - containerPort: 5000
---		==> Separator between your objects configuration 
apiVersion: v1
kind: Service
metadata:
  name: server-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: server
  ports:
    - port: 5000
      targetPort: 5000

It is highly debatable whether it is better to do that or not. It feels like having a file for every object type is great and having a good naming convention is also key to make this clear.


## Volumes in k8s

The filesystem in a pod/container is not persistent, so when the deployment will recreate a new pod (update of the image or crash of the container), the new pod will come with a new empty container, with no data carried over in the database (or filesystem)

One solution for this use case, a Volume can be presented to the db container(s)/pod(s), volume on the host machine, and when a pod is being recreated, the new container that comes online has this volume available and can read and write data on this volume.

One gotcha, 2 containers writing to the same volume w/o knowing each others is a great recipe for disaster, so definitely not recommended to have both containers up w/o adding a little bit more of configuration.

A little bit of terminology for 'Volumes' in k8s:
- A 'Volume' in generic container world: some type of mechanism that allows a container to access a filesystem outside itself
- A 'Volume' in k8s: an object (like a pod or a service) that allows a container to store data at the pod level

So a 'Volume' in k8s' world is DIFFERENT from a 'Volume' in docker's world!!

A Volume in kubernetes has nothing to do with a place where we can store data for a long time

When we create a Volume in k8s, we create a data storage pocket that exists or is tight to a very specific pod ==> belongs to the pod (think about AWS volumes for EC2 disks). So when the entire pod dies, the deployent will recreate a new pod and we are screwed with our data... hence we don't use this in production for db persistent storage


## Persistent Volume

A Persistent Volume is outside of the pod, so we do not hit the same issue than described for k8s Volumes. The new pod will be able to reach the Persistent Volume and read/write data on this Volume.


## Persistent Volume Claim

A PVC, which stands for Persistent Volume Claim, needs to exist to support write requirements from a database container (like postgres).

A PVC is an advertisement of options, but it is not an actual volume available. Think of it as a "there should be a 500GB disk available to all the different pods inside my cluster".

Two ways to provision volumes for this kind of usage:
- Statically provisioned Peristent Volume is something that we have very specifically created ahead of time
- Dynamically provisioned Peristent Volume is created on the fly (when the pod asks for it)

Example of a Persistent Volume Claim config file (database-persistent-volume-claim.yaml):
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: database-persistent-volume-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi

Different 'accessModes' options available:
- ReadWriteOnce: Can be used by a single node
- ReadOnlyMany: Multiple nodes can read from this
- ReadWriteMany: Can be read and written to by many nodes

There are many other options that might be interesting but we can rely on the default values to start with.

Check Local drive options for k8s:
`kubectl get storageclass`
`kubectl describe storageclass` ==> Provisioned right away (VolumeBindingMode)
Minikube uses 'minikube-hostpath' as a way to connect the VM to the Local host.

In the case of a cloud provider, many different options:
- Google Cloud Persistent Disk
- Azure File
- Azure Disk
- AWS Block Store
Most of the time, they are defined by default by the cloud provider for you.

All other options are available at: https://kubernetes.io/docs/concepts/storage/storage-classes/

How to address a PVC in a pod template? We need to add the following section in the deployment config file, under 'spec > template > spec':
    spec:
      volumes:		==> NEW! Present a volume to the pod refering to the PVC name we created in the other config file (database-persistent-volume-claim.yaml)
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: database-persistent-volume-claim
      containers:
        - name: postgres
          image: postgres
          ports:
            - containerPort: 5432
          volumeMounts:		==> NEW! Specify where to mount this volume inside the pod/container
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
              subPath: postgres 	==> Very specific to postgres, specify where the data are gonna be nested (in a folder) under the 'mountPath' directory on the persistent volume. The reason is that postgres is not really aware of you running this inside a container (more complicated than that, but that's the idea...)

Check Persistent Volumes created:
`kubectl get pv` ==> describes briefly what persistent volume is created 
`kubectl get pvc` ==> describes briefly the advertisement of a PV that can be accessible by containers


## Environment variables

Environment variables might be very useful when we need to provide info from one container to another using k8s.

To define env varibales in a deployment config file, insert an 'env' section under 'spec > template > spec > containers' like the following example (server-deployment.yaml):
spec:
  containers:
    - name: server
      image: stephengrider/multi-server
      ports:
      	- containerPort: 5000
      env:
        - name: REDIS_HOST
          value: redis-cluster-ip-service 	==> Value from name metadata inside redis-cluster-ip-service.yaml file
        - name: REDIS_PORT
          value: '6379'
        - name: PGUSER
          value: postgres
        - name: PGHOST
          value: postgres-cluster-ip-service 	==> Value from name metadata inside postgres-cluster-ip-service.yaml file
        - name: PGPORT
          value: '5432'
        - name: PGDATABASE
          value: postgres

Important!! Pay very close attention to the way you define integer variables, it has to be converted to strings using quotes, otherwise you get an error when trying to apply these config files!


## Secrets

For passwords, we don't want to store a password in plain text in a config file. To do that, k8s has introduced another Object type called 'Secrets' which securely stores a piece of information in the cluster, such as a database password.

In this case, we don't rely on a declarative command, we need to define the secret in an imperative command, because we DO NOT want to provide the password in plain text.

To create the secret, use the following command:
`kubectl create secret generic <secret_name> --from-literal key=value

'generic' ==> other options available are 'docker-registry' (custom authentication with your docker registry) and 'tls' (for https setup with tls keys). 'generic' is commonly used for general purposed projects.
'<secret_name>' ==> reference name so that can be used in the pod config
'--from-literal' ==> we are going to add the secret info into this command, as opposed to from . file
'key=value' ==> defines whatever we want, but essentially here, we're going to use PGPASSWORD=<password> and the password will be stored as an environment variable called PGPASSWORD

Check secrets created:
`kubectl get secrets`

==> When doing that, we need to update the server deployment (of course!), but also the postgres deployment as it will have to use something different than the default postgres password!

To add the password env var, insert a new section under 'spec > template > spec > containers > env' like the following example (server-deployment.yaml AND postgres-deployment.yaml):
- name: PGPASSWORD 			==> Name of the variable that will be looked for inside the container
  valueFrom:
    secretKeyRef:
      name: pgpassword 		==> Name of the secret we defined with the imperative command (valid only locally)
      key: PGPASSWORD 		==> Name of the key defined in the kubectl command. Important to note that we could have more than one key:value pair, hence we need to specify the name of the key we want to make reference to. This key name does NOT have to be the same name as the name of the environment variable!


## Load Balancers service

A load balancer is a legacy way of getting network traffic into a cluster.

It is not the prefered service nowadays, the Ingress service is the new one used by many people!

When using public cloud providers, k8s will provision a LB service from your public cloud provider.

It seems to be depracted now, so we can focus on the Ingress service.


## Ingress service

An Ingress service exposes a set of services to the outside world.

There are many Ingress services, the typical example is Nginx.

To create confusion :), there are two concurrent projects that use the same nginx for Ingress service, but one is community-led, the other is company-led (nginx, now F5). They are totally separate!! Information about these two projects:
- https://github.com/kubernetes/ingress-nginx 		==> community-led (the one we use in our app example)
- https://github.com/nginxinc/kubernetes-ingress 	==> company-led

And to help you out :), the setup of ingress-nginx changes depending on your environment (Local, GC, AWS, Azure)!!

What are we trying to achieve with this Ingress Service? Let's look at the following diagram:
            +-----------------------------------------------+
            |                      Node                     |
            |                                +------------+ |
            | +------------------+     +---->+  Service   | |
            | |  Something that  |     |     +------------+ |
Traffic+----->+ accepts incoming +-----+                    |
            | |     traffic      |     |     +------------+ |
            | +--------+---------+     +---->+  Service   | |
            |          ^                     +------------+ |
            |          |                                    |
            |          |                                    |
            | +--------+---------+     +------------------+ |
            | |Ingress Controller+<----+  Ingress Config  | |
            | +------------------+     +------------------+ |
            +-----------------------------------------------+

The Ingress controller is something that is going to look at the current state and compare it to the desired state and make sure that it's gonna do or create something to achieve what's required in terms of ingress routing rules.

In our project, we will merge the Ingress Controller and this thing that routes traffic (very minor difference)

In GC, when using Ingress-nginx as an Ingress routing mechanism, here is what we have in reality:
          +---------------------------------------------------------------------------------------------------------------------------------------+
          |                                                             Node                                                 +------------------+ |
          |                                                             +-----------+------------------+                     |    Deployment    | |
          |                                                             |           |    Deployment    |                     +------------------+ |
          |                                                             |           +------------------+               +-----+ multi-worker pod | |
          |                                                        +--->+ ClusterIP | multi-client pod |               |     +------------------+ |
          |                                                        |    |  Service  +------------------+               |                          |
          |                                                        |    |           | multi-client pod |               v                          |
          | +----------+     +-----------+--------------------+    |    |           +------------------+         +-----+-----+------------------+ |
          | |  Google  |     | ClusterIP |     Deployment     |    |    |           | multi-client pod |         | ClusterIP |    Deployment    | |
Traffic---->+  Cloud   +---->+  Service  +--------------------+    |    +-----------+------------------+    +--->+  Service  +------------------+ |
          | |   Load   |     |           |       nginx-       +----+                                        |    |           |    Redis pod     | |
          | | Balancer |     |           |controller/nginx pod|    |    +-----------+------------------+    |    +-----------+------------------+ |
          | +----------+     +-----------+---------+----------+    |    |           |    Deployment    |    |                                     |
          |                                        ^               |    |           +------------------+    |                                     |
          |                                        |               |    | ClusterIP | multi-server pod +----+    +-----------+------------------+ |
          |                                        |               +--->+  Service  +------------------+    |    | ClusterIP |    Deployment    | |
          |                                +-------+--------+      |    |           | multi-server pod +----+--->+  Service  +------------------+ |
          |                                | Ingress Config |      |    |           +------------------+    |    |           |   Postgres pod   | |
          |                                +----------------+      |    |           | multi-server pod +----+    +-----------+--------+---------+ |
          |                                                        |    +-----------+------------------+                              |           |
          |                                                        |                                                                  v           |
          |                                                        |    +-----------+------------------+                     +--------+---------+ |
          |                                                        |    | ClusterIP |    Deployment    |                     |   Postgres PVC   | |
          |                                                        +--->+  Service  +------------------+                     +------------------+ |
          |                                                             |           |default-backnd pod|                                          |
          |                                                             +-----------+------------------+                                          |
          +---------------------------------------------------------------------------------------------------------------------------------------+

Couple of details on this architecture:
- GC implementation is the easiest to understand
- The Ingress Config file (what we create) which describes all the routing rules that should exist inside of our application
- This config is going to be fed into a deployment that is running both the nginx-controller and an nginx pod
- The nginx pod is what takes the incoming traffic and route it off to the appropriate service inside the application (based on the rules defined in our Ingress Config file)
- The GCLB is created by default when using Ingress Nginx in GC (which is the so called LB service considered to be legacy, but still being used in GC...)
- The default-backend deployment is also created by default by GC to run a series healthchecks of our application. Ideally, that should be done by the multi-server deployment.

Why do we really need this ingress-nginx project as we could use a our custom nginx deployment?
There are many reasons behind that, one of them is that the ingress-nginx implementation knows that it is running inside a kubernetes cluster (helpful for many regards) and that is very helpful. One simple example: the ingress-nginx solution is able to provide session stickyness and route traffic to the right multi-client pod, completely bypassing the clusterIP service. The ClusterIP service still exists and monitors the different pods, but is not used to load balance traffic from the nginx pod to the multi-xxx pods.

How to get some help setting up NGINX Ingress Controller?
The official documentation can be found at: https://kubernetes.github.io/ingress-nginx/deploy/

Implementation for minikube for example:
- `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml` ==> generic for all implementations
- `minikube addons enable ingress` ==> Enable the ingress service for minikube, and that's pretty much it!

How to define the Ingress Config file?
What we want to achieve is the following:
           +-------------------------------------------------------------------+
           |                              Node                                 |
           |                               +-----------------+      +--------+ |
           |                         +---->+ The request has +----->+ Client | |
           |                         |     |  a path of '/'  |      +--------+ |
           |                         |     +-----------------+                 |
           | +------------------+    |                                         |
Traffic ---->+ Look at the path +----+                                         |
           | |  of the request  |    |                                         |
           | +------------------+    |                                         |
           |                         |     +------------------+     +--------+ |
           |                         +---->+ The request has  +---->+ Server | |
           |                               | a path of '/api' |     +--------+ |
           |                               +------------------+                |
           +-------------------------------------------------------------------+

Example of an Ingress Config file (ingress-service.yaml):
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-service
  annotations:
    kubernetes.io/ingress.class: nginx					==> Telling k8s that we want to create an ingress controller based on the nginx project
    nginx.ingress.kubernetes.io/rewrite-target: /$1 	==> Rewrite the URL while routing by removing the '/api' part for instance
spec:
  rules:
    - http:
        paths:
          - path: /?(.*)			==> Ingress Routing rule 1, to the client's clusterIP service or client pods   directly
            backend:
              serviceName: client-cluster-ip-service
              servicePort: 3000
          - path: /api/?(.*)		==> Ingress Routing rule 2, to the server's clusterIP service or server pods   directly
            backend:
              serviceName: server-cluster-ip-service
              servicePort: 5000

Check the Ingress service:
`kubectl get ingress`
`kubectl describe ingress`


## Minkube Dashboard

UI view of the cluster and everything that can be defined as an object in your cluster.

'Replica Sets' and 'Replication Controllers' are now deprecated and are replaced by Deployments, hence why we did not describe it earlier.

To launch the minikube dashboard, use the following command:
`minikube dashboard`


## Cloud Providers for Kubernetes

Why to use GC over AWS for your Kubernetes implementation?
- Google created Kubernetes!
- AWS only "recently" got Kubernetes support (subjective, true as of mid-2018)
- Far, far easier to poke around Kubernetes on Google Cloud
- Excellent documentation for beginners


## Using Travis CI with GCP

As a reminder, Travis CI is here to test our code, but also deploy the application into our cluster in GC.

Steps in the Travis config file:
- Install GC SDK CLI ==> we have to download and install the SDK every time that we run our Travis build
- Configure the SDK without GC auth info ==> we're going to authorize the CLI to make changes to our GC account, more specifically the k8s cluster we've just created
- Login to Docker CLI
- Build the 'test' version of the multi-client image
- Run tests ==> the tests don't really matter, we just want to understand the flow of docker, travis and k8s in GCP
- If tests are successful, run a script to deploy mewest images
- Build all our images, tag each one, push each to docker hub
- Apply all configs in the k8s directory
- Imperatively set latest images on each deployment

Travis will make sure that our code is always in sync with our k8s cluster.

A file must be created in the working directory: .travis.yml

An example of a (partial) .travis.yml config file is below:
sudo: required
services:
  - docker
before_install:
  - curl https://sdk.cloud.google.com | bash > /dev/null; ==> download SDK
  - source $HOME/google-cloud-home-sdk/path.bash.inc ==> modify the bash profile
  - gcloud components update kubectl ==> install kubectl
  - gcloud auth activate-service-account --key-file service-account.json ==> activate-service account is like IAM in AWS, basically an API key file

To use the last command in this file, we need to do the following operations:
- Create a Service Account
- Download service account credentials in a json file
- Download and install the Travis CLI
- Encrypt and upload the json file to our Travis account
- in .travis.yml, add code to unencrypt the json file and load it into GC SDK

To create a Service account in GCP console, go to Main Menu > IAM & admin > Service accounts and click on 'CREATE SERVICE ACCOUNT' (at the top). Follow the process, creating a travis-deployer username and make this user Kubernetes Cluster Admin as a role. Generate the keys for this user.

Download the json file created for this user on your local machine.
Important note: the json file downloaded from our GC account should NEVER be exposed to the outside world!! It is extremely important to secure this file to a location where it will never be exposed.

To download and install Travis CLI, we'll use a docker container that has ruby already installed on it and then we can use travis CLI in there. Here is the set of commands we use for that:
- `docker run -it -v $(pwd):/app ruby:2.3 sh
- `gem install travis`
- `travis login`
- Copy the json file into the 'volumed' directory so we can use it in the container
- `travis encrypt-file service-account.json -r plgingembre/multi-k8s` ==> and follow the comments from the command!!

After all these operations, we need to add the following statement to our travis config file:
`openssl aes-256-cbc -K $encrypted_0c35eebf403c_key -iv $encrypted_0c35eebf403c_iv -in service-account.json.enc -out service-account.json -d`

So now, our .travis.yml file looks like this:
sudo: required
services:
  - docker
before_install:
  - openssl aes-256-cbc -K $encrypted_0c35eebf403c_key -iv $encrypted_0c35eebf403c_iv -in service-account.json.enc -out service-account.json -d
  - curl https://sdk.cloud.google.com | bash > /dev/null;
  - source $HOME/google-cloud-home-sdk/path.bash.inc
  - gcloud components update kubectl
  - gcloud auth activate-service-account --key-file service-account.json
  - gcloud config set project multi-k8s-245514 				==> Specifies to the GC CLI what project to use in GCP, as we can have many of them
  - gcloud config set compute/zone us-west1-a 				==> Specifies to the GC CLI what zone for this project
  - gcloud container clusters get-credentials multi-cluster	==> Specifies to the GC CLI what cluster to use when issuing set of k8s related commands 


## Helm + Tiller

Helm is a client, Tiller is a server (running inside our Kubernetes cluster). You could compare this to the docker client and docker server


## Changes in Production Code

To change our code when it's running live in production, follow the steps below:
- Check out a branch
- Make changes
- Commit changes
- Push to github branch
- Create a Pull Request (PR)
- Wait for tests to show up green
- Merge the Pull Request
- See changes appear on production app!


## Namespace

A Namespace is used to kind of isolate the different resources that we are creating inside a k8s cluster

Check namespaces:
`kubectl get namespaces`
`kubectl describe namespaces`


## Deploy Kubernetes

There are many options to deploy k8s:
- Minikube ==> perfect for a developer who wants to test code on his/her local machine 
- Vagrant VM
- Kubespray ==> Production ready deployment using Ansible for automated deployment
- Cloud implementations