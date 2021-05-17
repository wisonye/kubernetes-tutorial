# `Kubernetes` Tutorial

[1. What is `Kubernetes`](#1-what-is-kubernetes)</br>
[2. Installation](#2-installatio)</br>
[3. Start your local k8s minikube cluster](#3-start-your-local-k8s-minikube-cluster)</br>
[4. Basic `kubectl` commands](#4-basic-kubectl-commands)</br>
[4.1 Concept about the layer abstraction](#41-concept-about-the-layer-abstraction)</br>
[4.2 Deployment CRUD](#42-deployment-crud)</br>
[4.2.1 Create deployment](#421-create-deployment)</br>
[4.2.2 List deployment/replicaset/pod info](#422-list-deploymentreplicasetpod-info)</br>
[4.2.3 List extra info for deployment/replicaset/pod](#423-list-extra-info-for-deploymentreplicasetpod)</br>
[4.2.4 Edit deployment](#424-edit-deployment)</br>
[4.2.5 Delete deployment](#425-delete-deployment)</br>
[4.3 Logging](#43-logging)</br>
[4.4 Debugging pods](#44-debugging-pods)</br>
[5. Manage deployment and service by configuration file](https://github.com/wisonye/kubernetes-tutorial#5-manage-deployment-and-service-by-configuration-file)</br>

## 1. What is `Kubernetes`
It's a container orchestration tool developed by Google. It helps you to manage your containerized
applications in different deployment environments.

By the way, if you’re wondering where the name “Kubernetes” came from, it is a Greek word, 
meaning helmsman or pilot. 

The abbreviation `K8s` is derived by replacing the eight letters of “ubernete” with the digit 8.

</br>


## 2. Installation

`minikube` is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.
Basically, `minikube` is a kubernetes cluster run in your local machine (single node case). It
includes the 4 `master` processes and 3 `worker` processes in your single-node cluster environment.

```bash
# HyperKit is an open-source hypervisor for macOS hypervisor, optimized for lightweight
# virtual machines and container deployment.
#
# If Docker for Desktop is installed, you already have HyperKit!!!
brew install hyperkit

brew install minikube
```

As the `kubernetes CLI` is the dependency for `minikube`, so you don't need to install
`kubernetes CLI` separated.

If you see the error below, that means you've already had the prev/older version of `kubernetes CLI`:


```bash
Error: The `brew link` step did not complete successfully
The formula built, but is not symlinked into /usr/local
Could not symlink bin/kubectl
Target /usr/local/bin/kubectl
```

Then you can overwrite it manually like this:

```bash
# Remove the old binary link and relink to the newly installed version
sudo rm -rf /usr/local/bin/kubectl
brew link --overwrite kubernetes-cli
# Linking /usr/local/Cellar/kubernetes-cli/1.21.0... 225 symlinks created.

# Check the version to make sure link correctly
kubectl version
# Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.0", GitCommit:"cb303e613a121a29364f75cc67d3d580833a7479", GitTreeState:"clean", BuildDate:"2021-04-08T21:14:14Z", GoVersion:"go1.16.3", Compiler:"gc", Platform:"darwin/amd64"}
# The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

</br>

## 3. Start your local k8s minikube cluster

By default, `minikube` uses the `hyperkit` driver with a pre-installed docker, that's you don't need to install
`Docker Dekstop` to run `minikube`.

```bash
minikube start

# 😄  minikube v1.20.0 on Darwin 10.14.6
# ✨  Automatically selected the hyperkit driver. Other choices: virtualbox, ssh
# 💾  Downloading driver docker-machine-driver-hyperkit:
#     > docker-machine-driver-hyper...: 65 B / 65 B [----------] 100.00% ? p/s 0s
#     > docker-machine-driver-hyper...: 10.52 MiB / 10.52 MiB  100.00% 15.81 MiB
# 🔑  The 'hyperkit' driver requires elevated permissions. The following commands will be executed:
# 
#     $ sudo chown root:wheel /Users/wison/.minikube/bin/docker-machine-driver-hyperkit
#     $ sudo chmod u+s /Users/wison/.minikube/bin/docker-machine-driver-hyperkit
# 
# 
# Password:
# 💿  Downloading VM boot image ...
#     > minikube-v1.20.0.iso.sha256: 65 B / 65 B [-------------] 100.00% ? p/s 0s
#     > minikube-v1.20.0.iso: 245.40 MiB / 245.40 MiB  100.00% 57.09 MiB p/s 4.5s
# 👍  Starting control plane node minikube in cluster minikube
# 💾  Downloading Kubernetes v1.20.2 preload ...
#     > preloaded-images-k8s-v10-v1...: 491.71 MiB / 491.71 MiB  100.00% 40.62 Mi
# 🔥  Creating hyperkit VM (CPUs=2, Memory=6000MB, Disk=20000MB) ...
# 🐳  Preparing Kubernetes v1.20.2 on Docker 20.10.6 ...
#     ▪ Generating certificates and keys ...
#     ▪ Booting up control plane ...
#     ▪ Configuring RBAC rules ...
# 🔎  Verifying Kubernetes components...
#     ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
# 🌟  Enabled addons: storage-provisioner, default-storageclass
# 🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

</br>


## 4. Basic `kubectl` commands

#### 4.1 Concept about the layer abstraction

    `Deployment` -- manages --> `Replica set` -- manages --> `Pod` -- abstraction of --> `Container (instance)`
    
    That's why you only need to deal with the `Deployment` as an administrator, no need to interact with the `replicaset and `pod`.

</br>


#### 4.2 Deployment CRUD

##### 4.2.1 Create deployment

```bash
kubectl create deployment test-web-server-depl --image=nginx --replicas=3 --port=8088
# deployment.apps/test-web-server-depl created
```

</br>

##### 4.2.2 List deployment/replicaset/pod info

```bash
kubectl get deployment --output=wide
# NAME                   READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES   SELECTOR
# test-web-server-depl   3/3     3            3           5m49s   nginx        nginx    app=test-web-server-depl

kubectl get replicaset --output=wide
# NAME                              DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES   SELECTOR
# test-web-server-depl-79d6c8b8b4   3         3         3       5m54s   nginx        nginx    app=test-web-server-depl,pod-template-hash=79d6c8b8b4

kubectl get pod --output=wide
# NAME                                    READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
# test-web-server-depl-79d6c8b8b4-dlvpc   1/1     Running   0          6m    172.17.0.4   minikube   <none>           <none>
# test-web-server-depl-79d6c8b8b4-fj874   1/1     Running   0          6m    172.17.0.3   minikube   <none>           <none>
# test-web-server-depl-79d6c8b8b4-xg65c   1/1     Running   0          6m    172.17.0.5   minikube   <none>           <none>
```

- The `--output=[format]` option supports `json/yaml/wide`
- The `--no-headers` doesn't print the header row

</br>

As you can see above the `naming convention` would look like below:

`[deployment name]-[replicaset name]-[pod name]` 

If you don't specified the `name`, then random id will be generated like below:

`test-web-server-depl-79d6c8b8b4-xg65c`

</br>

##### 4.2.3 List extra info for deployment/replicaset/pod

Sometimes, it's very useful to know the deployment scales history, then you use `kubectl describe deployement`:

```bash
kubectl describe deployment test-web-server-depl

# Name:                   test-web-server-depl
# Namespace:              default
# CreationTimestamp:      Mon, 17 May 2021 17:08:21 +1200
# Labels:                 app=test-web-server-depl
# Annotations:            deployment.kubernetes.io/revision: 1
# Selector:               app=test-web-server-depl
# Replicas:               5 desired | 5 updated | 5 total | 3 available | 2 unavailable
# StrategyType:           RollingUpdate
# MinReadySeconds:        0
# RollingUpdateStrategy:  25% max unavailable, 25% max surge
# Pod Template:
#   Labels:  app=test-web-server-depl
#   Containers:
#    nginx:
#     Image:        nginx
#     Port:         8088/TCP
#     Host Port:    0/TCP
#     Environment:  <none>
#     Mounts:       <none>
#   Volumes:        <none>
# Conditions:
#   Type           Status  Reason
#   ----           ------  ------
#   Progressing    True    NewReplicaSetAvailable
#   Available      False   MinimumReplicasUnavailable
# OldReplicaSets:  <none>
# NewReplicaSet:   test-web-server-depl-79d6c8b8b4 (5/5 replicas created)
# Events:
#   Type    Reason             Age   From                   Message
#   ----    ------             ----  ----                   -------
#   Normal  ScalingReplicaSet  57s   deployment-controller  Scaled up replica set test-web-server-depl-79d6c8b8b4 to 3
#   Normal  ScalingReplicaSet  4s    deployment-controller  Scaled up replica set test-web-server-depl-79d6c8b8b4 to 5
```

Also, extra info for the particular pod:


```bash
kubectl describe pod test-web-server-depl-79d6c8b8b4-5tnhv

# Name:         test-web-server-depl-79d6c8b8b4-5tnhv
# Namespace:    default
# Priority:     0
# Node:         minikube/192.168.64.2
# Start Time:   Mon, 17 May 2021 17:09:14 +1200
# Labels:       app=test-web-server-depl
#               pod-template-hash=79d6c8b8b4
# Annotations:  <none>
# Status:       Running
# IP:           172.17.0.7
# IPs:
#   IP:           172.17.0.7
# Controlled By:  ReplicaSet/test-web-server-depl-79d6c8b8b4
# Containers:
#   nginx:
#     Container ID:   docker://c6d39c2319a2a4f945898124bc70b2aa2d850ca52dcb4c82a2e952e57674587f
#     Image:          nginx
#     Image ID:       docker-pullable://nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2
#     Port:           8088/TCP
#     Host Port:      0/TCP
#     State:          Running
#       Started:      Mon, 17 May 2021 17:09:18 +1200
#     Ready:          True
#     Restart Count:  0
#     Environment:    <none>
#     Mounts:
#       /var/run/secrets/kubernetes.io/serviceaccount from default-token-x5j2b (ro)
# Conditions:
#   Type              Status
#   Initialized       True
#   Ready             True
#   ContainersReady   True
#   PodScheduled      True
# Volumes:
#   default-token-x5j2b:
#     Type:        Secret (a volume populated by a Secret)
#     SecretName:  default-token-x5j2b
#     Optional:    false
# QoS Class:       BestEffort
# Node-Selectors:  <none>
# Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
#                  node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
# Events:
#   Type    Reason     Age   From               Message
#   ----    ------     ----  ----               -------
#   Normal  Scheduled  3m6s  default-scheduler  Successfully assigned default/test-web-server-depl-79d6c8b8b4-5tnhv to minikube
#   Normal  Pulling    3m5s  kubelet            Pulling image "nginx"
#   Normal  Pulled     3m2s  kubelet            Successfully pulled image "nginx" in 2.599375714s
#   Normal  Created    3m2s  kubelet            Created container nginx
#   Normal  Started    3m2s  kubelet            Started container nginx
```

Same thing, you can describe all resources:

```bash
kubectl describe nodes
kubectl describe deployments
kubectl describe pods
```

</br>

##### 4.2.4 Edit deployment

```bash
kubectl edit deployment test-web-server-depl
```

After that, it opens the live configuration file inside your default editor. Then you can change it and save it,
`kubernetes` make sure update to the new state you desired.


</br>


##### 4.2.5 Delete deployment

```bash
kubectl delete deployment test-web-server-depl
```

</br>

## 4.3 Logging

- View only one container logging with the exactly pod name:

    ```bash
    # `--follow` live update
    kubectl logs test-web-server-depl-6dfb46c84f-8ngz2 --follow
    ```


- View all container logs with live update

    ```bash
    # `--selector` use for querying and filtering
    kubectl logs --selector=app=test-web-server-depl --all-containers --follow
    ```

    </br>

## 4.4 Debugging pods

You can login into the particular pod container instance to have a look by running:

```bash
# `test-web-server-depl-6dfb46c84f-6l5hj` is the [pod name]
kubectl exec -it test-web-server-depl-6dfb46c84f-6l5hj -- bin/bash
```


</br>


## 5. Manage deployment and service by configuration file

You can use `kubectl apply --filename [configuration file name]` to create or edit deployment and service.

`kubernetes` make sure the actual state will be the same as the desired state eventually. If some pods are
terminated, then new pods will be created which work like a self-healing.

Below is the sample configuration file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: test-web-server-depl
    labels:
        app: nginx-server

# Specification for the deployment
spec:
    replicas: 5
    selector:
        matchLabels:
            app: nginx-server
    template:
        metadata:
            labels:
                app: nginx-server

        # Specification for the pod
        spec:
            containers:
            - image: nginx
              imagePullPolicy: Always
              name: nginx
              ports:
              - containerPort: 8088
                protocol: TCP
            restartPolicy: Always
            terminationGracePeriodSeconds: 5
```

</br>

After you created or updated the configuration file, you can use `kubectl delete --filename [configuration file name]`
to remove the deployment or service.

If you compare to the `Docker Swarm`, here is the command comparison list:

| Docker Swarm | Kubernetes
|--------------|---------------
|# Start/update cluster </br>docker stack deploy --compose-file [filename] | # Start/update cluster </br> kubectl apply --filename [filename]
|# Stop cluster </br>docker stack rm [stack name] | # Stop cluster </br> kubectl delete --filename [filename]

</br>


