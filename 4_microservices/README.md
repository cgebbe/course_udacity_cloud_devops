# Microservices at Scale using AWS & Kubernetes

## Deploy an event-driven microservice

### What are Functions as a Service (FaaS)?

- serverless platform
- does not store data
- typically event-driven

### Benefits

- simpler because...
  - server infrastructure management no longer required
  - functional programming is a design well suited to distributed computing
  - scale your functions automatically and independently of rest of application
- event based
- can be more cost-effective
- leverages cloud-native technologies

### How to create them

Apart from Cloudformation, Terraform, Pulumi, etc., you use [Chalice](https://github.com/aws/chalice) to deploy AWS lambda functions by decorating python functions:

```python
from chalice import Chalice

app = Chalice(app_name="helloworld")

# Alternatively: @app.schedule(Rate(5, unit=Rate.MINUTES))
@app.route("/")
def index():
    return {"hello": "world"}
```

### Key characteristics of cloud native

- autoscaling
- infinite compute
- infinite storage
- advanced APIs (e.g. translate German to English)
- elastic
- fault-tolerant
- designed for failure
  - property that enables a system to continue operating properly in the event of the failure
  - good example is netflix and chaos engineering
- event-driven

### Creating an AWS Lambda function using Cloud9

AWS Serverless Application Model (SAM) is an open-source framework for building serverless applications. To get started with building SAM-based applications, use the [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-reference.html#serverless-sam-cli).

**sam init** – If you're a first-time AWS SAM CLI user, you can run the sam init command without any parameters to create a Hello World application. The command generates a preconfigured AWS SAM template and example application code in the language that you choose.

**sam local invoke** and **sam local start-api** – Use these commands to test your application code locally, before deploying it to the AWS Cloud.

**sam deploy** – Use this command to deploy your serverless application to the AWS Cloud. It creates the AWS resources and sets permissions and other configurations that are defined in the AWS SAM template.

See directory "my-sam-app" in [my github exercise repository](https://github.com/cgebbe/DevOps_Microservices).

## Using Docker format containers

### Basics

**Docker Desktop** includes:

- _Docker Engine_
  - server with a long-running daemon process `dockerd`.
  - APIs which specify interfaces that programs can use to talk to and instruct the Docker daemon.
  - command line interface (CLI) client docker.
- Docker Compose (tool for defining and running multi-container Docker applications)
- Docker Content Trust
- Kubernetes
- Credential Helper

**DockerHub** is used similar to github to share container images.

### Benefits over Virtual Machines

- _Size_: Containers are much smaller than Virtual Machines (VM) and run as isolated processes versus virtualized hardware. VMs can be GBs while containers can be MBs.
- _Speed_: Virtual Machines can be slow to boot and take minutes to launch. A container can spawn much more quickly typically in seconds.
- _Composability_: Containers are designed to be programmatically built and are defined as source code in an Infrastructure as Code project (IaC). Virtual Machines are often replicas of a manually built system. Containers make IaC workflows possible because they are defined as a file and checked into source control alongside the project’s source code.

### Intermezzos

- great [Makefile tutorial](https://makefiletutorial.com/)
- `circleci config validate` to validate your config locally
- `circleci local execute` to run job(s) locally (caching not supported)
- Note: this is also possible with gitlab! `gitlab-runner --debug exec docker lint_project` ([but no `include`, stages, artifacts, ...](https://gitlab.com/gitlab-org/gitlab-runner/-/issues/2797))

### Exercise: Build docker and push to AWS ECR

See [instructions for creating the ECR registry](https://docs.aws.amazon.com/AmazonECR/latest/userguide/ECR_GetStarted.html).

Very helpful is also opening AWS ECR, your repository and then "View push commands", in particular for the login command via the AWS CLI.

## Containerization of an existing application

### Some docker tips

- Docker features [logging](https://docs.docker.com/config/containers/logging/configure/).
  - docker run -it --log-driver json-file --log-opt mode=non-blocking ubuntu
- `docker port <container_name>` shows port mappings of a container
  - docker run -p 127.0.0.1:80:9999/tcp ubuntu bash
- docker can configure [CPU, memory and GPU resources](https://docs.docker.com/config/containers/resource_constraints/)

## Container orchestration using Kubernetes

### Benefits

- Created, Used and Open Sourced by Google
- High Availability Architecture
- Auto-Scaling is Built In
- Rich Ecosystem
- Service Discovery
- Container Health Management
- Secrets and Configuration Management

### What is Kubernetes

Kubernetes is available via e.g. Amazon EKS, Google Kubernetes Engine (GKE) and Azure Kubernetes Service (AKS). The core operations involved in Kubernetes include

- creating a Kubernetes Cluster
- deploying an application into the cluster
- exposing an application ports
- scaling an application
- updating an application

![](README.assets/2022-09-18-12-13-54.png)

### Hierarchy

- each K8s **cluster** has
  - one cluster master (node) with the K8s API
  - multiple **nodes** containing
    - multiple **pods** containing
      - multiple **containers** and/or volumes

From the [k8s documentation](https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/):

#### Pods

A Pod (as in a pod of whales or pea pod) is a group of one or more containers, with _shared storage and networking (IP-address) resources_. Pods are designed to support multiple cooperating processes (as containers) that form a cohesive unit of service.

![](README.assets/2022-09-18-12-45-46.png)

Pods are used as the _unit of replication_ in Kubernetes. If your application becomes too popular and a single pod instance can’t carry the load, Kubernetes can be configured to deploy new replicas of your pod

```yaml
# Example pod definition
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
```

#### Nodes

A node may be a virtual or physical machine. Each node includes the kubelet agent, a container runtime, and the kube-network-proxy.

![](README.assets/2022-09-18-12-29-27.png)

#### Clusters

A cluster pools together many nodes. It's mainly responsible for [distributing work to individual nodes](https://medium.com/google-cloud/kubernetes-101-pods-nodes-containers-and-clusters-c1509e409e16) (when work is added/removed or when nodes are added/removed).

![](README.assets/2022-09-18-12-36-31.png)

### More Terms

#### Namespaces

Namespaces provides a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespaces are intended for use in environments with many users spread across multiple teams, or projects. For clusters with a few to tens of users, you should not need to create or think about namespaces at all.

#### Users

All Kubernetes clusters have two categories of users: service accounts managed by Kubernetes, and normal users. Any user that presents a valid certificate signed by the cluster's certificate authority (CA) is considered authenticated. From there, the role based access control (RBAC) sub-system would determine whether the user is authorized to perform a specific operation on a resource.

#### Deployments

A Kubernetes Deployment tells Kubernetes how to create or modify pod instances via ReplicaSets. In production environments, you'll likely never create pods directly but only deployments (and services), see https://stackoverflow.com/questions/41325087/what-is-the-difference-between-a-pod-and-a-deployment .

#### Services

There are multiple services: ClusterIP, ExternalName, LoadBalancer, NodePort. A typical use case is to [make an app (a port) available to outside the cluster.

- ClusterIP exposes the service on a cluster's internal IP address.
- NodePort exposes the service on each node’s IP address at a static port.
- LoadBalancer exposes the service externally using a load balancer.

Sources:

- https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/
- https://aws.amazon.com/premiumsupport/knowledge-center/eks-kubernetes-services-cluster/

### How to...

#### ...creae a cluster locally

Follow official [tutorial](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/) with [primer information](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/). I used [another tutorial](https://minikube.sigs.k8s.io/docs/start/) without interactive console. The default driver didn't work, so specified another one: `minikube start --driver=docker`. The dashboard also didn't work :/.

There are several possibilities to create a lightweight k8s cluster locally (minikube, microk8s, k3s, kind, ...), see some comparisons [1](https://www.itprotoday.com/cloud-computing-and-edge-computing/lightweight-kubernetes-showdown-minikube-vs-k3s-vs-microk8s),[2](https://blog.flant.com/small-local-kubernetes-comparison/),[3](https://microk8s.io/compare), [4](https://blog.radwell.codes/2021/05/best-kubernetes-distribution-for-local-environments/)

![](README.assets/2022-09-18-12-56-13.png)

#### ...manage multiple clusters

It seems I already used kubectl for aws; now minikube seems to require another version(?). According to [kubeflow docs](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/), the kubectl version should be within one minor version of your cluster's API server. For example, a v1.2 client should work with v1.1, v1.2, and v1.3 master. Use [asdf](https://github.com/asdf-vm/asdf) (15.6k stars) to switch between different kubectl versions based on `$PWD/.tool-versions`. Other tools are recommended in this [SO post](https://stackoverflow.com/a/64684505/2135504).

To switch between different contexts, one should setup a kubeconfig file according to [kubernetes docs](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/). The kubeconfig file is found [in this order](https://medium.com/@ahmetb/mastering-kubeconfig-4e447aa32c75): `--kubeconfig`flag, `KUBECONFIG` environment variable, `$HOME/.kube/config`. It looks like this:

```yaml
apiVersion: v1
kind: Config
preferences: {}

clusters:
  - cluster:
    name: development
  - cluster:
    name: scratch

users:
  - name: developer
  - name: experimenter

contexts:
  - context:
    name: dev-frontend
  - context:
    name: dev-storage
  - context:
    name: exp-scratch
```

Each context is a triple (cluster, user, namespace). To set context details run

```
kubectl config --kubeconfig=config-demo set-context dev-frontend --cluster=development --namespace=frontend --user=developer
kubectl config --kubeconfig=config-demo set-context dev-storage --cluster=development --namespace=storage --user=developer
kubectl config --kubeconfig=config-demo set-context exp-scratch --cluster=scratch --namespace=default --user=experimenter
```

To select a context, run `kubectl config --kubeconfig=config-demo use-context dev-frontend`.

Use [kube-ps1](https://github.com/jonmosco/kube-ps1) (2.8k stars) to show the current context and namespace. Use [kubectx](https://github.com/ahmetb/kubectx) (14k stars) to switch between contexts. Use [direnv](https://github.com/direnv/direnv) (9.6k stars) to automatically load environment variables based on current directory. My conclusion: For each cluster, setup new directory with specific kubectl version (loaded via asdf), kubeconfig file and `KUBECONFIG` environment variable (loaded via direnv).

#### create a cluster using AWS EKS

via multiple ways:

- via UI
- via CloudFormation (console)
- via [eksctl](https://github.com/weaveworks/eksctl) <-- recommended way (also configures kubectl)

```bash
# to create/edit/delete a cluster
eksctl create cluster --name eksctl-demo --region=us-east-2 --node-type t3.micro
eksctl get cluster --name=eksctl-demo --region=us-east-2

# to "interact" (?) with the cluster
# eksctl configures ~/.kube/config
kubectl get nodes
```

#### deploy app to cluster

```bash
# publish docker image with hello world flask server
docker tag python-helloworld gebbissimo/python-helloworld:v1.0.0
docker push gebbissimo/python-helloworld:v1.0.0


# Assuming the Kubernetes cluster is ready
./kubectl get nodes
# Deploy an App from the Dockerhub to the Kubernetes Cluster
./kubectl create deploy python-helloworld --image=gebbissimo/python-helloworld:v1.0.0
# See the status
./kubectl get deploy,rs,svc,pods
# Forward LOCAL port to a pod
./kubectl port-forward pod/python-helloworld-c4d4bcf9c-w7mxg --address 0.0.0.0 5000:5000

# Alternative (? not tested): deploy using docker compose
docker stack deploy --namespace my-app --compose-file /path/to/docker-compose.yml mystack
```

#### log

Get logs for a specific pod via `./kubectl logs pod/python-helloworld-c4d4bcf9c-w7mxg`.

For a production environment, Prometheus is recommended again, see:

- [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)
- [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus) including the Prometheus Operator and more helpful tools
- [Prometheus Adapter](https://github.com/kubernetes-sigs/prometheus-adapter) for k8s metrics

#### debug

Here are some debug commands:

```bash
kubectl get pods
kubectl describe pod {POD NAME}
```

#### scale

According to [k8s docs](https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/scale-intro/):

```bash
# scale horizontally _manually_
kubectl scale {deployment name} --replicas={desired number of replicas}

# scale horizontally _automatically_ (based on CPU utilization, memory or custom metrics)
# https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
```

## Operationalizing Microservices

Feels like a bouqet of topics put together: circleci, [locus load testing](https://locust.io/), .... Very shallow :/

## Operationalizing a Machine Learning Microservice API

...
