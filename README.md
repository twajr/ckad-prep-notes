# ckad-prep-notes
List of resources and notes for passing the Certified Kubernetes Application Developer (CKAD) exam. Official links below. 

- [Kubernetes Certified Application Developer - Main](https://www.cncf.io/certification/ckad/)
- [CNCF Official CKAD Exam Tips](https://www2.thelinuxfoundation.org/ckad-tips)
- [CNCF Official CKAD Candidate Handbook](https://www.cncf.io/certification/candidate-handbook)


## Current Kubernetes Version
Version: 1.10.2

## Overview
The exam is 100% hands on using the innotvative exams (www.examslocal.com) product. The platform basically disables copy/paste so the exam requires an excellent understanding the quickest way to accomplish tasks on Kubernetes via the command line (kubectl). 

You will be given a list of 'tasks' to accomplish on one of four kubernetes clusters. The exam is basically 'open book' but only with the content available at kubernetes.io. Don't expect that you can just research the questions during the exam, however, as there will be very little time for 'learning' a k8s concept at exam time. 

The items in this repo / page describe and follow the official curriculum and mostly point back to the various documents at Kubernetes.io. There is a nice checklist that you can update once you think you have mastered a particular topic. 

I think the best approach is to fork this repo as a starting point for your studies, and then use the markdown checklist to ensure you cover all of the expected masterial, etc. 

Right now there are four primary sections to this document.
- Where to Practice?
- Important Tips and Tricks
- Checklist of Curriculum Progress
- List of resources (mostly K8s.io) to study

## Where to Practice?


## IMPORTANT TIPS
The exam is about speed and efficiency. If you have to spend lots of time looking at documentation, you will have zero chance of completing the many questions. With that said, the following will help with time management.

### Setting Default Namespace
You have to create various contexts to move between namespaces. It's just easier to always use the namespace flag (-n) when running commands, IMO.

### Using the RUN command
The resource created from a particular RUN command is based on its 'generator'. For example, by defauilt the run creates a deployment. However, it can also create a POD, or JOB, or CRONJOB, based on the ---restart flag. 
```
$kubectl run nginx --image=nginx   (deployment)
$kubectl run nginx --image=nginx --restart=Never   (pod)
$kubectl run busybox --image=busybox --restart=OnFailure   (job)
$kubectl run busybox --image=busybox --schedule="*/1 * * * * *"  --restart=OnFailure
```
The --schedule flag creates a Cron Job, and --restart=OnFailure creates a Job resource. 
### Extracting yaml from running resource
Use the --export and -o yaml flags to export the basic yaml from an existing resource:
```
kubectl get deploy busybox --export -o yaml > exported.yaml
```
### The --dry-run flag
The --dry-run flag can be used with the kubectl run and create commands. It provides a nice template to start your declarative yaml config files. Below is an example for creating a basic secret yaml. 
```
kubectl create secret generic my-secret --from-literal=foo=bar -o yaml --dry-run > my-secret.yaml
```
Also above, the --from-literal flag is useful for things like config maps and secrets for their basic cases. The above output:
```
apiVersion: v1
data:
  foo: YmFy
kind: Secret
metadata:
  creationTimestamp: null
  name: my-secret
```
## OTHER Key Items
### 'Exposing' Ports for PODS
Pods can all inter-communicate via their internal IP address and port. Services are needed to expose services OUTSIDE of the cluster. So, it's important to understand the basic container spec for specifying the PORT a container will use. See below:
```
spec:
    containers:
    image: nginx
    imagePullPolicy: Always
    name: busybox
    env:
    - name: PORT
        value: "80"
    ports:
    - containerPort: 80
        protocol: TCP
```
### CronJobs
Job vs CronJob -> A job runs a pod to a number of successful completions. Cron jobs manage jobs that run at specified intervals and/or repeatedly at a specific point in time. 
```
$ kubectl run x  --image=busybox --schedule="*/1 * * * *" --restart=OnFailure --dry-run -o yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  labels:
    run: x
  name: x
spec:
  startingDeadlineSeconds: 20
  concurrencyPolicy: Allow
  jobTemplate:
    metadata:
      creationTimestamp: null
    spec:
      template:
        metadata:
          creationTimestamp: null
          labels:
            run: x
        spec:
          containers:
          - image: busybox
            name: x
            resources: {}
            command: ["date"]
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'
```
### Network Policies
Resources use labels to select pods and define rules which specify what traffic is allowed to the selected pods. So, the pods themselves require certain labels / selectors to enable network policies.

By default, pods are non-isolated; they accept traffic from any source.

Pods become isolated by having a NetworkPolicy that selects them. Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections that are not allowed by any NetworkPolicy. (Other pods in the namespace that are not selected by any NetworkPolicy will continue to accept all traffic.)

Servicess are required / part of implementing network policies, it seems. 

### Deployments, Rollouts, Roll-Backs...
Well described here:
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

### Container Metrics
```
$ kubectl top pod
```
### Config Maps / Environment Variables
Command below to create a simple config map with a key/value pair.
```
$ kubectl create configmap app-config --from-literal=key123=value123
```
Mapping all keys to a pod's env is tricky:
```
spec:
  containers:
  - envFrom:
    - configMapRef:
        name: app-map
    image: nginx
    name: nginx-x
```
# Current Progress
The list below is based on the curriculum v1.0

- [x] Core Concepts - 13%
  - [x] API Primitives
  - [x] Create and Configure Basic Pods
- [ ] Configuration - 18%
  - [x] Understand ConfigMaps
  - [x] Understand SecurityContexts
  - [x] Define App Resource Requirements
  - [ ] Create and Consume Secrets
  - [x] Understand Service Accounts
- [ ] Multi-Container Pods - 10%
  - [ ] Design Patterns: Ambassador, Adapter, Sidecar
    - [ ] - Sidecar Pattern
    - [ ] - Init Containers
- [ ] Pod Design - 20%
  - [ ] Using Labels, Selectors, and Annotations
  - [ ] Understand Deployments and Rolling Updates
  - [ ] Understand Deployment Rollbacks
  - [ ] Understand Jobs and CronJobs
- [ ] - State Persistence - 8%
  - [ ] - Understand PVCs for Storage
- [ ] Observability - 18%
  - [ ] Liveness and Readiness Probes
  - [x] Understand Container Logging
  - [ ] Understand Monitoring Application in Kubernetes
  - [x] Understand Debugging in Kubernetes
- [ ] Services and Networking - 13%
  - [x] Understand Services
  - [ ] Basic Network Policies

## Core Concepts and Kubectl

- [Accessing Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
- [Accessing Cluster with API](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/)
- [Port Forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)
- [Shell to Running Container (exec)](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)

## Configuration

- [Task -> Config Maps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
- [Task -> Security Contexts](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Tasks -> Assigning Memory Resources to Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/)
- [Tasks -> Assigning CPU Resources to Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)
- [Tasks -> Pod QOS](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/)
- [Tasks -> Credentials using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)
- [Tasks -> Project Volume w/Secrets](https://kubernetes.io/docs/tasks/configure-pod-container/configure-projected-volume-storage/)
- [Tasks -> Setting Service Account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)


## Multi-Container Pods
- One or more containers running within a pod for enhancing the main container functionality (logger container, git synchronizer container); These are sidecar container

- One or more containers running within a pod for accessing external applications/servers (Redis cluster, memcache cluster); These are called ambassador container

- One or more containers running within a pod to allow access to application running within the container (Monitoring container); These are called as adapter containers- 

- [Tasks -> Init Containers](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/)

## Pod Design

- [Tasks -> ReplicaSet Rolling Updates](https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/)
- [Deployments and Rollbacks](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)

### CRON
- [Tasks -> Automated Tasks with Cron Jobs](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)
- [Tasks -> Parallel Jobs with Expansions](https://kubernetes.io/docs/tasks/job/parallel-processing-expansion/)
- [Tasks -> Course Parallel Processing with a Work Queue](https://kubernetes.io/docs/tasks/job/coarse-parallel-processing-work-queue/)
- [Tasks -> Fine Parallel Processsing with a Work Queue](https://kubernetes.io/docs/tasks/job/fine-parallel-processing-work-queue/)

## State Persistence

- [Tasks -> Configuring PVCs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

## Observability
- [App Introspection and Debugging](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application-introspection/)
- [Tasks - Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
- [Tasks -> Debugging Pods](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller/)
- [Troubleshooting Applications](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)
- [Debugging Services](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/)
- [Debugging Services Locally](https://kubernetes.io/docs/tasks/debug-application-cluster/local-debugging/)

## Services and Networking

- [Declare Network Policy](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)